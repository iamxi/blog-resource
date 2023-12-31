---
title: "hexo3搭建博客"
date: 2017-10-01T03:44:13+08:00
draft: false
slug: blog-with-hexo3
---

前几天心血来潮，翻出了不知道那年的博客，用户hexo重新搭建了下。把这个是记录下来。

# 安装

这个不多说，[hexo 官方文档](https://hexo.io/zh-cn/docs/)有介绍。

# 配置

基本的一些配置上面，上面也有，还有一些我参考了[这篇博客](http://www.jianshu.com/p/808554f12929)，在此先谢过这位朋友的分享。

# 遇到的问题及解决

## CNAME 文件
因为 Github 需要 CNAME 文件，用于指定域名。不过放在 `public` 文件夹中的话，只要 `clean` 一次就没了。网站找了下资料，应该将其放在 `source` 目录总，这样在生成时候就会一起带到 `public` 文件夹中。一些博文中需要的图片（images）和favicon.ico等不变的内容也可以一起放在 `source` 目录下。

## README.md文件不渲染
在站点配置文件中添加

	skip_render: README.md

## stiemap 的生成
需要安装插件

	npm install hexo-generator-sitemap --save

站点配置文件中添加配置

	sitemap:
		path: sitemap.xml

## 博文永久连接
默认是时间分开。但是这样路径比较长。不方便管理。网上有种方法是urlname。使用urlname来作为链接，避免使用文件名。这样文件名就可以自己定义，方便本地管理。

具体做法是，在站点配置文件中，修改配置如下：

	permalink: post/:urlname.html
	
然后在每一篇 `post` 中都需要加一个urlname，如这篇文章的：

```
title: 20171001-hexo3搭建博客
date: 2017-10-01 03:44:13
tags: hexo
urlname: blog-with-hexo3
```

缺点是，如果那篇忘记了这个，那生成的 `html` 文件将会是未知

## 图片保存
hexo官网有推荐的方法，[文档](https://hexo.io/zh-cn/docs/asset-folders.html)。当时我的觉这个太依赖hexo，不符合markdown的理念。所以，还是选择土办法管理图片，放在 `source/images` 目录下，post中直接用相对路径应用。亲测可行。

## 博客的版本控制
hexo 的部署功能，只能建 `public` 文件夹中的内容推送到 Github 上。而配置文件和博客源文件都无法提交。如果换了电脑或电脑换了，那就什么都要从新开始了。

解决方法是，将整个博客目录都提交到 Github 上。

hexo 生成站点目录时候，已经有一个 `.gitignore` 文件，里面帮我们忽略了所有的没必要提交的内容。我估计这个文件的目录就是方便保存配置和源文件。

我的做法是，Github 上使用同一个源，用不同分支分别保存生成的页面和源文件。如 `master` 保存生成的整个站点。`post` 分支保存源文件。`master` 由 hexo 部署命令自动推送。 `post` 由我手动推送。

这里说到主题的版本控制。我使用的 NexT 主题。按教程，我是直接从 Github 上 clone 下来的。`themes/next` 文件夹中有 git 的版本信息。这样，我提交博客配置时候，这个文件夹无法被提交， git 当他是一个子项目。后来看了下网站相同问题的解决方法。大致是将主题 fork 到自己的仓库中。然后在 clone 下来。这样如果主题配置或样式有改动，可以提交到自己的远程仓库中。不然他人的源肯定是不给提交的。

# 总结
hexo 这东西还是很容易上手的。完全就看能不能持久的写博客了。
