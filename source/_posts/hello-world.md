---
title: Hexo+github博客搭建
---
&emsp;&emsp;hexo是一个博客框架，可以将markdown文本编译成静态网页。github提供了github pages,这是一个静态网站托管服务。所以前者生成的静态网页可以托管在github pages上。通过在网上搜索，发现[这篇文章](https://cuiqingcai.com/7625.html)写的很详细，根据这篇文章部署了该[博客网站](https://wdxx.github.io/)。
<!--more-->

## 准备工作
&emsp;&emsp;搭建前需要先安装好Node.js，可以通过[这里](https://nodejs.org/zh-cn/download/)下载安装。然后就是安装hexo与github pages创建。需要注意的是在申请github作为github pages的仓库时，仓库名需要是{name}.github.io的命名方式。如果你还没有github账户的话赶紧去[官网](https://github.com/)申请一个吧。
### 安装hexo
安装命令如下：
```bash
npm install -g hexo-cli
```

## 初始化项目
&emsp;&emsp;到目前为止，我们的准备工作已经完成了，我们拥有了hexo框架，github仓库。接下来需要做的就是使用hexo来初始化一个博客项目，hexo提供了一个命令行工具，用于快速创建项目、页面、编译、部署hexo博客。创建博客项目的命令如下：
```bash
mkdir myblogsite
cd myblogsite
hexo init {name} ## name就是你要创建的项目的名字，一般取签名说的github的用户名就可以了。
```
&emsp;&emsp;这个步骤中hexo为我们的博客项目创建了{name}文件夹，这个文件夹下面还有node_modules、scaffolds、themes等文件夹，先不管这些文件夹有啥作用，继续后面的步骤。我{name}文件夹下面可以看到一个source/_post文件夹，这个下面可以存放markdown个是的文本文件，hexo可以将这个文件编译成静态的html,然后我们就可以用来部署了。将markdown格式的文件编译成html的命令如下：
```bash
cd {name}
hexo generate ## 也可以使用  hexo g 命令
```
&emsp;&emsp;执行上面的命令后可以发现在{name}/public下生成了许多文件，这就是转换过程重生成的文件。可以通过如下的命令在本地调试查看生成的效果：
```bash
hexo server
```
&emsp;&emsp;在上面的命令执行完成后，如果服务启动正常，那么会打印如下的log：
```bash
INFO  Start processing
INFO  Hexo is running at http://localhost:4000 . Press Ctrl+C to stop.
```
&emsp;&emsp;可以看到，访问的地址为"http://localhost:4000"，在浏览器中打开就可以看到效果了。

## 部署
&emsp;&emsp;从上面的步骤中已经生成了静态的页面，接下来只需要部署到github就可以了。在部署之前需要安装一个git的部署插件，安装命令如下:
```bash
npm install hexo-deployer-git --save
```
&emsp;&emsp;接下来需要配置部署地址。打开项目文件夹下面的"_config.yml"文件，找到"Deployment"配置项，配置成如下格式:
```bash
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: git
  repo: https://github.com/wdxx/wdxx.github.io.git
  branch: master
```
&emsp;&emsp;将type配置成git,repo配置成github项目地址，branch配置成github仓库分支。配置完地址之后执行如下部署命令:
```bash
hexo deploy
```
部署过程会上传编译好的结果，log如下：
```bash
Counting objects: 58, done.
Delta compression using up to 4 threads.
Compressing objects: 100% (52/52), done.
Writing objects: 100% (58/58), 358.87 KiB | 0 bytes/s, done.
Total 58 (delta 8), reused 0 (delta 0)
remote: Resolving deltas: 100% (8/8), done.
To https://github.com/wdxx/wdxx.github.io.git
   3cd02f7..13b140e  HEAD -> master
Branch master set up to track remote branch master from https://github.com/wdxx/wdxx.github.io.git.
INFO  Deploy done: git
```
&emsp;&emsp;现在就可以打开[https://wdxx.github.io/](https://wdxx.github.io/)看看部署好后的效果了。

## 修改主题
&emsp;&emsp;hexo提供了各种各样的主题用于定义自己喜欢的样式，一般使用最流行的[next](https://github.com/theme-next/hexo-theme-next/tree/v7.8.0)主题，主题需要自己单独下载，存放在工程的themes目录下。下载next的命令如下：
```bash
git clone https://github.com/theme-next/hexo-theme-next.git themes/next
```
&emsp;&emsp;下载完主题后，需要在工程目录的"_config.yml"中将主题切换为next,在该文件中找到"Extensions"字段，配置成如下格式:
```bash
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: next
```

## 重新部署
&emsp;&emsp;在修改完配置，或者新发布文章后，需要重新编译然后部署，命令如下：
```bash
hexo clean
hexo generate
hexo deploy
```

## 配置站点信息
&emsp;&emsp;可以在 _config.yml 中修改大部分的配置。这里我们主要修改site区域的配置。各参数说明如下:  

| 参数         | 描述     |
| :-------     | --------: |
| title        | 网站标题  |
| subtitle     | 网站副标题    |
| description  | 网站描述     |
| keywords     | 网站的关键词。使用半角逗号 , 分隔多个关键词 |
| author       | 您的名字 |
| language     | 网站使用的语言。对于简体中文用户来说，使用不同的主题可能需要设置成不同的值，请参考你的主题的文档自行设置，常见的有 zh-Hans和 zh-CN。|
| timezone     | 网站时区。Hexo 默认使用您电脑的时区。请参考 时区列表 进行设置，如 America/New_York, Japan, 和 UTC 。一般的，对于中国大陆地区可以使用 Asia/Shanghai。|

我的博客具体配置如下:
```bash
# Site
title: wdxxliu
subtitle: 个人博客
description: 一个关于嵌入式方向的个人博客
keywords: "Python C/C++ Makefile Linux 编译 工具链"
author: wdxx
language: zh-CN
timezone: Asia/Shanghai
```



