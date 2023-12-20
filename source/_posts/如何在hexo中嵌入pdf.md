---
title: 如何在hexo中嵌入pdf
date: 2023-12-20 10:01:37
tags:
- hexo
- blog
categories: 教程
mathjax: true
---

本来是想把一周年纪念日的照片放在个人网站上，考虑到一张张上传太麻烦了，正好此前做了一本书，想着干脆直接上传 pdf 并嵌入到网站上。

说干就干，查了一些将 pdf 嵌入博客页面的方法，有`hexo-pdf`这个插件，但是似乎不支持多端适配。有朋友提到可以用`pdf.js`，简单方便，而且在桌面端和移动端都有不错的效果，那么开始实验！

<!-- more -->

接下来，进入正式流程：

+ 第一步，下载[pdf.js](https://mozilla.github.io/pdf.js/getting_started/#download)，将文件夹解压放到指定的位置。我的博客应用的是[Fluid](https://github.com/fluid-dev/hexo-theme-fluid)主题，将下载文件夹命名为 pdfjs，拷贝到`themes/fluid/source/js` 中

![](./如何在hexo中嵌入pdf\image-20231220101630961.png)

+ 第二步，修改`pdfjs/web/viewer.js`，将里面的特定内容注销，可以使用关键词搜索找到这里⬇️

![](./如何在hexo中嵌入pdf\image-20231220102717239.png)

+ 第三步，修改博客配置文件`_config.yml`（注意不是主题配置文件），将`pdfjs`文件夹加入到跳过渲染的选项里；

```yaml
skip_render: [myjs/**]
```

+ 第四步，在`pdfjs`文件夹里，新建一个 `pdf` 文件夹，用于存储需要嵌入的 pdf 文件。需要注意的是，存入的 pdf 文件名不能含有非法字符，比如`&`；

![](./如何在hexo中嵌入pdf\image-20231220103144520.png)

+ 第五步，在博文的markdown文档中使用 `<iframe>` 控件配合pdf.js 库完成 pdf  显示:

```javascript
<iframe src='/js/pdfjs/web/viewer.html?file=/js/pdfjs/pdf/我们的第一个春夏秋冬.pdf' style='width:100%;height:900px'></iframe>
```

注意此处填写的是相对路径⬆️，嵌入文档的高度和宽度可以调整，对于论文pdf或者ppt转pdf而言，将高度设置为`900px`似乎是个不错的选择。另外，由于markdown编辑器（比如typora）并不是完整的浏览器，所以在查看`.md`文档时，本地并不会成功显示 pdf 控件，而是会看到类似下面的内容：

![](./如何在hexo中嵌入pdf\image-20231220103903451.png)

部署完在网页端打开，浏览器是可以正常渲染的！效果如下：

![](.\如何在hexo中嵌入pdf\image-20231220104706148-1703040433644-1.png)
