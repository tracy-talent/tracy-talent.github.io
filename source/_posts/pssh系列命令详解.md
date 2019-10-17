---
title: pssh系列命令详解
date: 2019-10-17 20:43:55
tags:
	- pssh
categories:
	- Linux
---

## 安装

pssh提供OpenSSH和相关工具的并行版本。包括pssh，pscp，prsync，pnuke和pslurp。该项目包括psshlib，可以在自定义应用程序中使用。pssh是python写的可以并发在多台机器上批量执行命令的工具，它的用法可以媲美ansible的一些简单用法，执行起来速度比ansible快它支持文件并行复制，远程命令执行，杀掉远程主机上的进程等等。杀手锏是文件并行复制，，当进行再远程主机批量上传下载的时候，最好使用它。pssh用于批量ssh操作大批量机器；pssh是一个可以在多台服务器上执行命令的工具，同时支持拷贝文件，是同类工具中很出色的；比起for循环的做法，更推荐使用pssh！ (注意需要安装 python 2.4 或以上版本)

下面是直接从源码进行编译安装的步骤，安装过程很快

```shell
wget https://pypi.python.org/packages/60/9a/8035af3a7d3d1617ae2c7c174efa4f154e5bf9c24b36b623413b38be8e4a/pssh-2.3.1.tar.gz
tar xf pssh-2.3.1.tar.gz -C /usr/local
cd /usr/local/pssh-2.3.1/
python setup.py install
```

## 命令详解

`pssh --help`可以查看命令参数选项：

```
-l    远程机器的用户名
-p    一次最大允许多少连接
-o    输出内容重定向到一个文件
-e    执行错误重定向到一个文件
-t    设置命令执行的超时时间
-A   提示输入密码并且把密码传递给ssh（注意这个参数添加后只是提示作用，随便输入或者不输入直接回车都可以，可以结合sshpass -p password使用）
-O   设置ssh参数的具体配置，参照ssh_config配置文件
-x   传递多个SSH 命令，多个命令用空格分开，用引号括起来
-X   同-x 但是一次只能传递一个命令
-i   显示标准输出和标准错误在每台host执行完毕后
-I   读取每个输入命令，并传递给ssh进程 允许命令脚本传送到标准输入
```

pssh、pscp、prsync、pnuke和pslurp的具体使用：

```shell
#注:在使用工具前,确保主机间做了密钥认证,否则无法实现自动化,当然我们可以使用sshpass(yum install sshpass)配合pssh -A参数实现自动输入密码,但这要保证多台主机的密码相同,同时还要注意如果known_hosts没有信任远程主机,那么命令执行会失败,可以加上-O StrictHostKeyChecking=no参数解决,ssh能用的选项pssh也能用

# 集群刚装好系统处于原始状态，可以使用下面命令来生成其他机器的ssh秘钥并将各机器的rsa公钥添加到本机
sshpass -p password pssh -I -A -O StrictHostKeyChecking=no -h ip.txt -l brooksj -i "ssh-ketgen"  # 然后本机回车10次帮助各机器生成ssh秘钥，password为其它机器的统一密码
sshpass -p password pssh -A -O StrictHostKeyChecking=no -h ip.txt -l brooksj -i "ssh-copy-id localhost-ip" # localhost-ip改成你本机的ip
 
#pssh 远程批量执行命令 
pssh -h ip.txt -P "uptime" 
#-h  后面接主机ip文件,文件数据格式[user@]host[:port]
#-P  显示输出内容
#如果没办法密钥认证.可以采用下面方法,但不是很安全
sshpass -p 123456 pssh -A -h ip.txt -i "uptime"
 
#pscp 并行传输文件到远端
#传文件,不支持远程新建目录
pscp -h ip.txt test.py /tmp/dir1/
#传目录
pscp -r -h ip.txt test/ /tmp/dir1/
 
#prsync 并行传输文件到远端
#传文件,支持远程新建目录,即目录不存在则新建
prsync -h ip.txt test.py /tmp/dir2/
#传目录
prsync -r -h ip.txt test/ /tmp/dir3/
 
#pslurp从远程拉取文件到本地,在本地自动创建目录名为远程主机ip的目录,将拉取的文件放在对应主机IP目录下
#格式:pslurp -h ip.txt -L <本地目录>  <远程目录/文件>  <本地重命名>
#拉取文件
pslurp -h ip.txt -L /root/ /root/1.jpg picture
ll /root/172.16.1.13/picture
-rw-r--r-- 1 root root 148931 Jan  9 15:41 /root/172.16.1.13/picture
#拉取目录
pslurp -r -h ip.txt -L /root/ /root/test temp
ll -d /root/172.16.1.13/temp/
drwxr-xr-x 2 root root 23 Jan  9 15:49 /root/172.16.1.13/temp/
 
#pnuke:远程批量killall
pnuke -h ip.txt nginx
```



