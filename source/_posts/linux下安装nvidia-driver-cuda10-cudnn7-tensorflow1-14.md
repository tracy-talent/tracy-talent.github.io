---
title: 'ubuntu下安装nvidia driver, cuda10, cudnn7, tensorflow1.14'
date: 2019-09-18 11:28:12
tags:
	- tensoflow 
categories:
	- Machine Learning
	- Tensorflow
---

## 安装nvidia driver

* 查看本机显卡

  ```shell
  lspci | grep -VGA
  ```

  终端输出显卡名称，现在电脑一般都有集显+独显2块显卡，若都是nvidia公司的，后续如果安装openGL就不会冲突，因为openGL只支持nvidia的显卡，其他公司的会被openGL安装覆盖。我这里是intel集显，所以安装cuda的时候就不能安装openGL.

* 查看本机nvidia GPU型号

  ```shell
  lspci | grep -i nvidia
  ```

  到官网[official nvidia driver](https://www.nvidia.cn/Download/index.aspx?lang=cn#)下载对应自己系统版本和GPU型号的driver，cuda和nvidia driver的对应关系可以参考[cuda vs nvidia-driver](https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html)。我这里下载的GeForce GTX 1060，linux 64bit，game ready版本的driver。在真实安装之前还需要禁用系统现有的driver。

* 禁用nouveau driver

  nouveau是ubuntu16.04默认安装的第三方开源驱动，安装cuda会跟nouveau冲突，需要事先禁掉，运行命令`lsmod | grep nouveau`后需要没有任何输出就代表禁掉了。具体禁用方法如下：

  在/etc/modprobe.d中创建文件blacklist-nouveau.conf，在文件中添加以下内容

  ```
  blacklist nouveau
  options nouveau modeset=0
  ```

  命令行下执行`sudo update-initramfs –u`，然后在执行`lsmod | grep nouveau`，若无内容输出，则禁用成功，若仍有内容输出，请检查操作，并重复上述操作。

* 卸载已有nvidia driver

  可能本机上已安装过nvidia driver，但是安装更高版本的cuda需要安装更高版本的nvidia driver，查看系统是否已安装的nvidia driver

  ```shell
  sudo dpkg --list | grep nvidia-*
  ```

  如果包含nvidia-*开头的一系列文件则说明系统已安装过nvidia driver，执行以下命令下载已有驱动

  ```shell
  sudo dpkg purge nvidia-*
  ```

* 正式安装nvidia driver

  `ctrl+alt+f1`进入文字界面(`ctrl+alt+f7`回到图形桌面)，执行以下命令关闭图形界面

  ```shell
  sudo service lightdm stop
  ```

  进入到nvidia driver runfile所在目录执行

  ```shell
  chmod a+x NVIDIA-*.run
  ./NVIDIA-*.run
  ```

  如果询问安装openGL则不要答应，安装好之后即可进入有nvidia显卡驱动的桌面



## 安装cuda10

在官网[nvidia cuda downloads](https://developer.nvidia.com/cuda-toolkit-archive)下载对应版本的cuda，然后根据自己硬件和系统选择合适的cuda进行安装，我这里下载的cuda10，下载选项如下

<div align="center">
    <img src="/images/cuda1.png">
</div>

下载好之后进入到所在目录进行安装

```shell
chmod a+x cuda-*.run
./cuda-*.run  --no-opengl --tmpdir=/usr/local/tmp
```

安装过程中要借用/tmp目录，如果/tmp目录空间不足可以用--tmpdir指定一个tmp目录如上所示。安装过程中会询问你安装各种各样的东西，除了cuda toolkit，其它的都不需要安装，安装路径自己确定也可以保持默认，确认建立软连接到/usr/local/cuda。如果有补丁patch，则在安装完主模块之后再安装patch，方法一致。



## 安装cudnn7

在官网[nvidia cudnn downloads](https://developer.nvidia.com/rdp/cudnn-download)下载对应cuda版本的cudnn，cudnn下载要求必须登录账户才可以，我这里下载最新的cuDNN v7.6.3 for cuda 10 .0，下载好之后解压，然后将其库文件copy到cuda中

```shell
cp	CUDNN_HOME/include/cudnn.h	CUDA_HOME/include
cp	CUDNN_HOME/lib64/libcudnn*	CUDA_HOME/lib64/
ln	-sf  CUDA_HOME/lib64/lincudnn.so.7.6.3	CUDA_HOME/lib64/libcudnn.so.7
ln	-sf  CUDA_HOME/lib64/libcudnn.so.7  CUDA_HOME/lib64/libcudnn.so
sudo mkdir /etc/ld.so.conf.d/nvidia.conf
sudo cat CUDA_HOME/lib64
sudo ldconfig
```

ldconfig要求.so文件是软链接，所以必须ln -sf之后再执行ldconfig上面命令中的CUDNN_HOME和CUDA_HOME分别是cudnn和cuda安装目录，安装好之后配置环境~/.bashrc

```bash
# CUDA
export CUDA_HOME=/home/liujian/cuda-9.0
export LD_LIBRARY_PATH=$CUDA_HOME/lib64:$CUDA_HOME/extras/CUPTI/lib64:$LD_LIBRARY_PATH
export PATH=$CUDA_HOME/bin:$PATH
```

source ~/.bashrc之后，执行`nvcc -V`查看cuda版本，测试cuda是否已可用

```shell
cd CUDA_HOME/samples/1_Utilities/deviceQuery
make
./deviceQuery
```

如果最后显示pass则表明cuda安装成功，否则不可用



## 源码编译安装tensorflow1.14

安装tensorflow之前查看cuda和cudnn版本号以安装相匹配版本的tensorflow

```shell
# 查看cuda版本
cat CUDA_HOME/version.txt
# 查看cudnn版本
cat cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2
```

git clone tensorflow的官方仓库

```shell
git clone https://github.com/tensorflow/tensorflow.git
```

基于tag(版本)创建分支

```shell
git tag # 查看分支
git branch r1.14.0  v1.14.0  # 基于tagv1.14.0建立r1.14.0分之
git checkout r1.14.0
```

使用python3.6安装，tensorflow的源码编译安装还需要使用google的一款编译器bazel，具体安装教程以及tensoflow版本与cuda,cudnn,bazel的版本匹配请参考官网的[tensorflow install](https://tensorflow.google.cn/install/source).

<font color='red' size=5>Caution:</font>

这里记录一下我安装过程中出现的几个问题及经验，在正式bazel编译之前执行`./configure`进行配置，配置过程中建议使用gcc而不是clang，这样编译过程中不容易出错，然后配置过程中可能会报以下错误

```shell
Traceback (most recent call last):
  File "third_party/gpus/find_cuda_config.py", line 463, in <module>
    main()
  File "third_party/gpus/find_cuda_config.py", line 455, in main
    for key, value in sorted(find_cuda_config().items()):
  File "third_party/gpus/find_cuda_config.py", line 418, in find_cuda_config
    _get_default_cuda_paths(cuda_version))
  File "third_party/gpus/find_cuda_config.py", line 159, in _get_default_cuda_paths
    ] + _get_ld_config_paths()
  File "third_party/gpus/find_cuda_config.py", line 139, in _get_ld_config_paths
    match = pattern.match(line.decode("ascii"))
UnicodeDecodeError: 'ascii' codec can't decode byte 0xc2 in position 27: ordinal not in range(128)
Asking for detailed CUDA configuration...
```

这个问题可以参考[github issue: find_cuda](https://github.com/tensorflow/tensorflow/pull/28209)进行解决，具体做法：在third_party/gpus/find_cuda_config.py文件中找到<font color='red'>match = pattern.match(line.decode("ascii"))</font>，并将其修改为<font color='green'>match = pattern.match(line.decode(sys.stdin.encoding))</font>即可，重新执行`./configure`就不会再报错了。

<font color='magenta' size=4>写在最后：</font>

google提供了tensorflow多标签的docker镜像，使用docker容器安装使用tensorflow是最便捷且安全的，用户可以在tensorflow多版本之间自由切换，在服务器上使用也不用再受权限问题困扰了，而且最新的docker 19.03已经原生支持容器使用物理机上的gpu了，不再需要安装nvidia-docker来支持gpu使用了，用户只需在物理机上安装nvidia驱动，其它的都有镜像提供，喜大普奔。docker安装可以参考我之前写的一篇博客[Linux下docker安装教程](https://www.cnblogs.com/brooksj/p/11456329.html)，至于docker下使用tensorflow镜像的教程可以参考tensorflow官方教程[Docker](https://tensorflow.google.cn/install/docker)。

