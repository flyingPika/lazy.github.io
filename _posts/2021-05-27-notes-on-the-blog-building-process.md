---
title: 记博客搭建过程
date: 2021-05-27 20:00:00 +0800
categories: [探寻神秘之旅, 博客]
tags: [博客]     # TAG names should always be lowercase
math: true
pin: false
---

## 一、前言

该博客使用 [GitHub Pages](https://docs.github.com/en/pages) 和 [Jekyll](https://jekyllrb.com/) 搭建，选用 [Chirpy](https://chirpy.cotes.info/) 作为博客主题模板。

## 二、安装环境

* **注意安装路径不要有空格！**

* 首先安装 Ruby 和 Devkit，在[下载页面](https://rubyinstaller.org/downloads/)选择 `Ruby+Devkit 2.7.3-1 (x64)`。

![图1](/assets/img/posts/2021-05-27/2021-05-27-1.png)
_图1_

![图2](/assets/img/posts/2021-05-27/2021-05-27-2.png)
_图2_

![图3](/assets/img/posts/2021-05-27/2021-05-27-3.png)
_图3_

* 安装程序运行结束时，安装 MSYS2。输入3然后回车，结束后退出。

![图4](/assets/img/posts/2021-05-27/2021-05-27-4.png)
_图4_

* 然后安装 RubyGems，在[下载页面](https://rubygems.org/pages/download)下载压缩包 `rubygems-3.2.17.zip`。在解压后的文件目录下，执行命令 `ruby setup.rb`

* 再之后依次执行 `gem install bundler` 和 `gem install jekyll`，以安装 Bundler 和 Jekyll。

## 三、使用模板

首先 fork 模板 [Chirpy](https://github.com/cotes2020/jekyll-theme-chirpy)，将分支 `4-0-stable` 设为主分支，删除多余分支。然后执行 ```bundle``` 命令安装依赖，再执行 ```tools/init.sh``` 进行文件初始化，提交保存修改。接着修改配置文件 `_config.yml`。修改的属性包括： `baseurl`、`lang`、`title`、`tagline`、`url`、`github`、`social`、`avatar`。

由于笔者没有推特账号，还要在 `_data/contact.yml` 中注释掉推特的相关设置。另外修改 `_tabs/about.md` 文件，即修改博客的 About 页面。设置图标的方法见[文档](https://chirpy.cotes.info/posts/customize-the-favicon/)。

### 本地调试

执行 `bundle exec jekyll s`，访问指示的本地服务地址。

### 部署

* 推送提交到 `origin/master` 以触发 GitHub Actions workflow。构建完毕并成功，远端将会自动出现用来存储站点文件的新分支 `gh-pages`。

* 回到远端，通过 Settings → Options → GitHub Pages 选择分支 `gh-pages` 作为发布源。

![图5](/assets/img/posts/2021-05-27/2021-05-27-5.png)
_图5_

* 按照 GitHub 指示的地址访问网站。

## 四、写作博文

笔者使用 VS Code，安装 Markdown Preview Enhanced 扩展编写 Markdown 文件。博文被存储在 `_posts/` 中，命名格式为 `YYYY-MM-DD-TITLE.md`。**注意文件名中的 `TITLE` 必须为英文，否则不能进行本地调试**。

### Front Matter

基本属性包括 `title`、`date`、`categories` 和 `tags`，`date` 属性必须早于当前时间。

以下设置可以启用 MathJax。

```yaml
---
math: true
---
```

以下设置可以置顶博文。

```yaml
---
pin: true
---
```

### Markdown笔记

$$f(x)=\sin(x)+x^2$$

1. 列表项一号

2. 列表项二号

&nbsp;不换行空格&ensp;半角空格（1/2个中文宽度）&emsp;全角空格（1个中文宽度）

> 引用

**粗体**

*斜体*

~~删除线~~

`标记`

---

分隔行

---

超链接：<814833529@qq.com>


`\`用于转义
