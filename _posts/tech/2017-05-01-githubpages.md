---
layout: post
category: 技术
title:  "GitHub Pages建立自己的博客"
tags: [GitHub Pages,Jekyll,建站]
---

可以在[GitHub Pages主页]( https://pages.github.com "GitHub Pages主页")上找到创建步骤。

### 1、可以建立个人、组织的主页，建立后的url是**http://username.github.io**.

1.1、创建一个仓库，名字为*username*.github.io或者organization.github.io，个人名称对号入座。

1.2、自定义网址，解析*username*.github.io。

[Custom dimain with GitHub Pages]( https://help.github.com/categories/github-pages-basics/ "Custom dimain with GitHub Pages")



### 2、给仓库建立一个网站

通过设置，选择主题就可以生成一个网站了。



### 3、Jekyll建站超级简单

#### 3.1、添加网页——添加Md文件

Jekyll中一篇文章就是一个文件，所有需要发布的文章都要放在`_posts`文件夹内。Jekyll对于文章的文件名也是有要求的，系统会根据文件名来生成每篇文章的链接地址。具体的格式为：`YYYY-MM-DD-文章标题.markdown` 其中`YYYY`为4位年份，`MM`是两位的月份，`DD`是两位的日期。下图是本blog的文件列表：

![filelist](http://wellsnakeblog.qiniudn.com/2014052201filelist.png)

#### 3.2、Jekyll文件头信息

在使用Markdown撰写文章之前我们需要先设置头信息。头信息需要根据[YAML](http://yaml.org/)的格式写在两行三虚线之间。在头信息可以设置预定义的全局变量的值，Jekyll会根据变量的值来生成文章页面。下图是本篇文章的头信息：

![frontmatter](http://wellsnakeblog.qiniudn.com/2014052201frontmatter.png)

`layout`使用指定的模版文件，不加扩展名。模版文件放在_layouts目录下。

`title`文章的标题。

`date`发布文章的时间。

`categories`将文章设置成不同的属性。系统在生成页面时会根据多个属性值来生成文章地址。以上设置会生`http://.../jekyll/update/...`格式的文章链接。

`tags`标签，一篇文章可以设置多个标签，使用空格分割。

基本上一篇文章只要用到以上一些信息就可以了，当然还有其它的变量可以设置，具体用法可以在[Jekyll网站](http://jekyllrb.com/docs/frontmatter/)上查看。