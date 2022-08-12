---
title: Ubuntu 安装 pytorch-gpu
date: 2022-08-12 23:01:00
tags: 
- Pytorch
- Ubuntu
categories: 教程
mathjax: true
---
## 1. 安装驱动
终端输入以下命令查看推荐驱动版本
```
$ ubuntu-drivers devices
```
输出如下：

![在这里插入图片描述](https://img-blog.csdnimg.cn/9688ea450cf84399963656173bcd720d.png)
这里显示推荐版本为 515 版本
再在终端输入 `sudo ubuntu-drivers autoinstall` 即可自动安装，或者输入 `sudo apt install nvidia-driver-515` 安装，然后 `sudo reboot` 重启系统即可
运行 `nvidia-smi` 命令，查看驱动是否安装成功

![在这里插入图片描述](https://img-blog.csdnimg.cn/d20e252d431347cc874e0c06313a79dd.png)
返回的信息显示，安装的显卡驱动版本为 515， 最高可支持 11.7 的 cuda；到这里就说明我们的驱动已经安装成功了！！！

## 2. 安装 cuda
具体安装何版本的 cuda 取决于 pytorch 的版本。进入[pytorch官方安装页面](https://pytorch.org/get-started/locally/)，选择对应的版本
![在这里插入图片描述](https://img-blog.csdnimg.cn/06441e839f344ee4bc5d3d28873449a6.png)

这里显示 pytorch 1.12 版本对应的 cuda 是 11.6。现在我们就可以去 [cuda
下载页面](https://developer.nvidia.com/cuda-toolkit-archive)下载对应程序包

![在这里插入图片描述](https://img-blog.csdnimg.cn/b4915e69df7c4da3a9517f7bd767abcf.png)

点击进入，选择自己系统对应的版本，在终端中运行其提供的命令即可
![在这里插入图片描述](https://img-blog.csdnimg.cn/8f81a325e00f482f9cd3de75b9d02337.png)
![在这里插入图片描述](https://img-blog.csdnimg.cn/7edc450ef0eb44c8bc9782b883ae412f.png)

## 3. 配置环境变量
进入根目录，修改 bashrc 文件并添加环境变量。这一步的目的是为了让程序能够找到 cuda 的位置
```
$ cd ~
$ vi .bashrc
```
向 .bashrc 文件末尾添加如下内容
```
export LD_LIBRARY_PATH=/usr/local/cuda/lib64:/usr/local/cuda/extras/CPUTI/lib64
export CUDA_HOME=/usr/local/cuda/bin
export PATH=$PATH:$LD_LIBRARY_PATH:$CUDA_HOME
```
终端运行 `nvcc -V` 命令查看是否成功

![在这里插入图片描述](https://img-blog.csdnimg.cn/d5d760636aaf4094b68f414fcd16dec9.png)

## 4. 安装 pytorch
直接终端运行官方提供的下载命令
```
conda install pytorch torchvision torchaudio cudatoolkit=11.6 -c pytorch -c conda-forge
```
下载安装失败
尝试[清华源](https://mirrors.tuna.tsinghua.edu.cn/help/anaconda/)
先生成 `conda config --set show_channel_urls yes` 生成 `.condarc` 文件

```
channels:
  - defaults
show_channel_urls: true
default_channels:
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/r
  - https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/msys2
custom_channels:
  conda-forge: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  msys2: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  bioconda: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  menpo: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  pytorch-lts: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
  simpleitk: https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud
```

再次失败

![在这里插入图片描述](https://img-blog.csdnimg.cn/dcf61ab8a9b74acf8223429506590eb4.png)

开始面向 csdn ，寻找解决办法，参考了该篇博客：[总结：使用anaconda清华镜像源安装pytorch1.12.0stable版失败的问题综合，以及对应的解决方案](https://blog.csdn.net/wdnmdppx/article/details/125692448)，尝试未果，放弃！！！
改为安装 cuda11.3 的 pytorch

![在这里插入图片描述](https://img-blog.csdnimg.cn/7715b5adb4894ddca67099fb38785f73.png)

终端运行安装命令
```
conda install pytorch torchvision torchaudio cudatoolkit=11.3 -c pytorch
```
安装未报错

![在这里插入图片描述](https://img-blog.csdnimg.cn/28bcab7a65ca4b98a9559de588abfbe4.png)

顺利安装完成

![在这里插入图片描述](https://img-blog.csdnimg.cn/af645c7238084c46bf62d30d8e2306a6.png)

测试一下

![在这里插入图片描述](https://img-blog.csdnimg.cn/03a9695679394a52b79181cbf82ec6c5.png)
gpu 可以正常使用，nice ！！！


