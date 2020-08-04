---
title: 多机同步管理hexo博客
date: 2019-04-24 22:23:16
tags:
    - hexo
categories:
    - Hexo
---
## 一、关于搭建的流程

1. 创建仓库，\< your github username \>.github.io；可参考我的[tracy-talent.github.io](https://github.com/tracy-talent/tracy-talent.github.io)

2. 创建两个分支：master 与 hexo；

3. 设置hexo为默认分支（因为我们只需要手动管理这个分支上的Hexo网站文件,git clone会download默认分支）；

4. 使用git clone git@github.com:username/username.github.io.git拷贝仓库；

5. 在本地username.github.io文件夹下通过Git bash依次执行npm install hexo、hexo init、npm install 和 npm install hexo-deployer-git（此时当前分支应显示为hexo）;

6. 修改_config.yml中的deploy参数，分支应为master；

7. 下载hexo主题，推荐使用next，先fork一份[hexo-theme-next](https://github.com/iissnan/hexo-theme-next)到自己账号上(主要为了方便自己后期对主题的定制)，然后进入到themes目录下执行git submodule add git@github.com:username/hexo-theme-next.git将fork的next主题仓库download下来作为当前博客主仓库的子模块，这时username.github.io目录下会多出一个.gitmodules的文件(里面存储着所有子模块的文件路径和url)，然后进入到hexo-theme-next主题目录下，使用git branch -a查看当前子模块所出的分支，确保在主分支master上，然后git pull origin master使得子模块与远程保持一致，接着在进入到博客根目录username.github.io下，依次git add、git commit、git push提交到github hexo分支；

8. 网页端进入到username.github.io仓库下可以看到themes目录下hexo-theme-next存的是指向你fork的hexo-theme-next仓库最新commit id的指针，而不是实实在在的文件内容，点击之后会跳转到子仓库，github这样做也是为了节省存储开销

9. 后期如果对主题自定义，每次修改完之后确保将主题模块切换到master分支，然后提交修改。再回到博客主目录下提交修改，更新主线(hexo分支)指向子模块的指针，这样才能将第三方库即子模块同步到主线上，使得主线上指向子模块的指针对应的永远是子模块最新的commit id

10. 这时候如果想在另一台机器上同步部署博客，那么需要先安装npm，然后新建一个目录例如hexo(因为hexo初始化必须是一个空目录)，保证hexo已安装好(hexo install -g hexo-cli)，在hexo目录下依次执行hexo init、npm install 和 npm install hexo-deployer-git，然后退出hexo目录执行

    ```shell
    git clone git@github.com:username/username.github.io.git
    git submodule init && git submodule update
    
    #下面这一句的效果和上面三条命令的效果是一样的,多加了个参数  `--recursive`
    git clone git@github.com:username/username.github.io.git --recursive
    ```

    download完成之后将username.github.io目录下所有内容copy到hexo目录下，然后进入到hexo\themes\hexo-theme-next目录下，执行git checkout master切换到子模块的master分支上，执行git pull origin master同步子模块与远程子仓库，到此就实现了在另一台机器上与远程完全同步的工作了，可以在这台机器上写博客发表文章了，记得每次在不同机器作业时先在博客主目录下git pull origin master同步commit之后再开始写博客发表。

11. 执行hexo g -d生成网站并部署到GitHub上。

这样一来，在GitHub上的username.github.io仓库就有两个分支，一个hexo分支用来存放网站的原始文件，一个master分支用来存放生成的静态网页。完美( •̀ ω •́ )y！

## 二、关于日常的改动流程
在本地对博客进行修改（添加新博文、修改样式等等）后，通过下面的流程进行管理。

1. 先git pull origin hexo同步远程仓库
2. 依次执行git add .、git commit -m "..."、git push origin hexo指令将改动推送到GitHub（此时当前分支应为hexo）；
3. 然后才执行hexo g -d发布网站到master分支上。

## 三、Issues

1. 公式渲染失败

   hexo 默认的渲染引擎是 marked，但是 marked 不支持 mathjax，卸载marked并安装kramed

   ```bash
   npm uninstall hexo-renderer-marked --save
   npm install hexo-renderer-kramed --save
   ```

   然后，更改node_modules/hexo-renderer-kramed/lib/renderer.js

   ```bash
   // Change inline math rule
   function formatText(text) {
       // Fit kramed's rule: $$ + \1 + $$
       return text.replace(/`\$(.*?)\$`/g, '$$$$$1$$$$');
   }
   修改为
   // Change inline math rule
   function formatText(text) {
       return text;
   }
   ```

   然后停止使用hexo-math(hexo-math依赖hexo-inject一并删除)，安装mathjax

   ```bash
   npm uninstall hexo-math --save
   npm uninstall hexo-inject --save
   npm install hexo-renderer-mathjax --save
   ```

   更改默认转义规则，更改node_modules/kramed/lib/rules/inline.js

   ```bash
   // 替换11行
   escape: /^\\([\\`*{}\[\]()#$+\-.!_>])/,
   //修改为
   escape: /^\\([`*\[\]()# +\-.!_>])/,
   
   //替换20行
   em: /^\b_((?:__|[\s\S])+?)_\b|^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
   //修改为
   em: /^\*((?:\*\*|[\s\S])+?)\*(?!\*)/,
   ```

   开启mathjax，配置themes/hexo-theme-next/_config.yml

   ```bash
   mathjax:
     enable: true
     perpage: false
     cdn: //cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML
   ```

   并在博客中使用mathjax

   ```markdown
   ---
   title: Testing Mathjax with Hexo
   category: Uncategorized
   date: 2017/05/03
   mathjax: true
   ---
   ```

2. 博客顶部菜单栏点击后网址末尾多了'20%'导致网页无法找到

   修改themes/hexo-theme-next/_config.yaml

   ```bash
   menu:
     home: / || home
     about: /about/ || user
     tags: /tags/ || tags
     
     将空格链接与||之间的空格去掉
     menu:
     home: /|| home
     about: /about/|| user
     tags: /tags/|| tags
   ```

3. 站点概览中点击日志标签找不到网页

   在 themes\hexo-theme-next\layout\\_macro 找到sidebar.swig 文件找到如下代码

   ```
   <a href="{{ url_for(theme.menu.archives).split('||')[0] | trim }}">
   修改为
   <a href="{{ url_for(theme.menu.archives.split('||')[0]) | trim }}">
   ```

   