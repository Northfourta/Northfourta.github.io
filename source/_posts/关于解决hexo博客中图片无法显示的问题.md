---
title: 关于解决 hexo 博客中图片无法显示的问题
date: 2023-12-20 17:08:18
tags:
- hexo
- blog
categories: 教程
mathjax: true 
---

读研后大部分时间都待在办公室了，原来部署博客的笔记本电脑很少携带。为了方便更新博客，想着实现一下博客多端同步功能，即使在办公室也能轻松摸鱼写博客😊。虽然同步编辑实现了，却出现了博客中图片无法显示的问题；网上寻找解决办法，有效的博客并不多，甚至很多博客是一样的，存粹的copy。好在花费了一些时间，解决了这个问题，于是便想着记录一下解决过程，希望能给别的同志一些参考和启发。

<!-- more -->

## 1、初尝试

网上绝大数的解决方法都是设置`post__asset_folder属性为true` + `安装hexo-asset-image插件`，经尝试后并不好使。后来看到了[这篇博客](https://blog.csdn.net/m0_43401436/article/details/107191688)，受其启发去查看图片的路径，方定位到问题，才得以解决。

## 2、解决方案

### 2.1 创建图片资源文件夹

网上有关的解决方式几乎很大一部分会提到这一点：将博客根目录`_config.yml` 文件中的`post_asset_folder` 选项设为 `true` 来打开。事实上这正是hexo[官方文档](https://hexo.io/zh-cn/docs/asset-folders)给出的解决**方案之一**中的**一个步骤**。

该操作的作用就是在使用`hexo new "xxx"`命令新建博文时，在相同路径下同步创建一个`xxx`文件夹，而`xxx`文件夹的作用就是用来存放博客中引用的图片资源；

### 2.2 安装插件

很多博客提到`hexo-asset-image`这个插件，相信在网上找了一波解决方案的同学一定对这个名字不陌生。

插件安装方法参考官网：

```bash
npm install hexo-asset-image --save
```

该插件主要功能就是**路径转换**，根据markdown中图片的相对路径，给出html中图片的绝对路径。具体原理，金牛大王同学的[博客](https://blog.csdn.net/m0_43401436/article/details/107191688)中提过，我不再阐述。

### 2.3 配置typora图像引用设置

这里也是参考了网上的设置。这样设置，当我们向文档中添加图片时，软件会自动帮我们将图片复制到文档同名文件夹中，即前文提到的每篇博客对应的图片资源文件夹；可使我们更专注于写作。

![](image-20231220190441688.png)

### 2.4 更正markdown图片路径

图片不能正常显示，根本原因就是路径不正确，html无法识别。这里我打开`hexo g`后生成的博客`index.html`文件（文件位于博客根目录public文件夹中）

![](image-20231220191537253.png)

可以看到图片不能正常显示

![](image-20231220191709908.png)

使用浏览器检查元素工具查看图片的html源码

![](image-20231220191758373.png)

![](image-20231220191859827.png)

发现路径中出现两段重复的文字，且图片路径的最前方多了`/.io`

+ **如何去掉多余的前缀？**

  重复路径这一问题，主要是由于插件使用方法不正确导致，我们来看看[官网](https://github.com/xcodebuild/hexo-asset-image)的使用方法：

![](image-20231220200318080.png)

因此，在markdown文档中，图片的引用路径中不应该添加前缀，正确的引用如下：

![](image-20231220200518074.png)



+ **多余的`/.io`怎么处理呢？**

  经过官网查询，发现这个问题是`hexo-asset-image/index.js`中的一行代码导致的，原作者已经修复<u>（[域名是xxx.io的情况下，图片路径会从原本/xxx.jpg变成 /.io/xxx.jpg · Issue #47 · xcodebuild/hexo-asset-image (github.com)](https://github.com/xcodebuild/hexo-asset-image/issues/47)）</u>，不知为何我下载的依旧是旧版本

![](image-20231220201130720.png)

手动修改代码后，图片可正常显示。

![](image-20231220201553022.png)

至此，问题已全部解决

## 3、小结

现如今，网上的博客质量良莠不齐。各种问题也是因人而异，同样的解决方法适用于别人，可能不适用于自己。总是，遇见问题，还是要多尝试，多查资料，多思考
