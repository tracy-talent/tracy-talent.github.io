---
title: Hello Hexo
date: 2019-04-10 10:00:00
tags:
	- hexo
categories:
	- Hexo
---
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).

## Quick Start

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/deployment.html)

### Markdown renderer

hexo默认的markdown渲染插件<font color='red'>hexo-renderer-marked</font>效果不是很好，有些复杂公式渲染不出来，需要安装一个更好的markdown rendrer: <font color='green'>hexo-renderer-kramed</font>

``` shell
$ npm un hexo-renderer-marked --save
$ npm i hexo-renderer-kramed --save
```

安装好之后将其添加进站点目录下的package.json中，并删除默认markdown渲染插件



