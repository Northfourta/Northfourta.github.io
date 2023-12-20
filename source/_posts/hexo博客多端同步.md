---
title: hexo博客多端同步
date: 2023-12-19 20:49:34
tags:
- hexo
- blog
categories: 教程
mathjax: true
---

# 安装过程

1. 在新电脑上克隆username.github.io仓库的hexo分支(就是存放源码的分支)到本地，此时本地git仓库处于hexo分支,可以执行`git branch -v`查看。

2. 在clone下来的仓库文件夹中安装和配置hexo环境，依次调用命令：npm install hexo、npm install、npm install hexo-deployer-git；（不需要hexo init）

   ```bash
   npm install hexo # 安装hexo
   npm install # 安装依赖及插件
   npm install hexo-deployer-git
   ```

3. 新的终端电脑上生成新文章：

   + **打开命令行或终端**：进入到你的 Hexo 博客根目录。

   + **运行命令创建新文章**：Hexo 提供了一个命令来创建新文章。在命令行中输入以下命令：

     ```bash
     hexo new "文章标题"
     ```

     这会在 Hexo 的 `source/_posts` 目录下创建一个新的 Markdown 文件，文件名通常会基于标题自动生成。

   + **编辑文章**：使用喜欢的文本编辑器打开新生成的 Markdown 文件，然后编写你的文章内容。可以使用 Markdown 格式书写内容，并在文件头部配置文章的元数据（如标题、日期、标签等）。

   + **保存文件**：编辑完成后保存文件。

   + **生成静态页面、预览并部署**：在完成文章的编写后，你需要运行以下命令生成 Hexo 博客的静态页面：

     ```bash
     hexo generate
     hexo server
     hexo deploy
     ```

# 安装过程中遇到的问题

**Q1：在进行 `hexo g`时，报错`pandoc exited with code null`**

**S1：[「博客搭建」Hexo Next主题配置Mathjax遇到的问题：pandoc exited with code null-CSDN博客](https://blog.csdn.net/weixin_45073562/article/details/120289648)**

1. **使用Mathjax作为渲染器**
   使用 Mathjax 进行数学公式渲染，需要使用 hexo-renderer-pandoc 或者 hexo-renderer-kramed （官方不推荐）作为 Hexo 的 Markdown 渲染器。

   ```bash
   npm un hexo-renderer-marked  # 先将原有的渲染器卸载(可选)
   npm i hexo-renderer-pandoc		# 安装pandoc
   ```

2. **修改配置文件**
   在主题配置文件_config.yml 中找到math关键词，将mathjax的false改为true。注意，mathjax和katex只能有一个置为true。

   > 注意：还有一个per_page选项，默认为false，表示只对文章开头（front-matter）含有mathjax: true语句的文章进行渲染。为true表示会所有文章进行渲染，不管有没有加上mathjax: true语句。建议置为true，否则一篇一篇文章加上这条语句太麻烦了 **(未找到)**

3. **本地安装pandoc**
   使用pandoc还需要在本地安装，在官网上下载pandoc，直接安装即可。<u>安装完后，记得电脑重启一下，我之前就是没安装pandoc以及没重启导致出错，然后hexo -g一直提示pandoc exited with code null的，重启之后，就能正确生成html文件了。</u>

