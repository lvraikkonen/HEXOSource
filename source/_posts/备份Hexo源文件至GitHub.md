---
title: 备份Hexo源文件至GitHub
date: 2016-05-31 10:25:41
tags:
    - 备忘
    - hexo
---


本文转自：<http://www.leyar.me/backup-your-blog-to-github/>

Hexo是一款基于Node.js的静态博客框架，前一阵俺的笔记本泡水直接退役，但是博客的原文件还在那台死去的机器上，所以备份啊。。。
本质上，Hexo是将本地的md文件编译成静态文件上传到github上（或者其他），所以建议是将本地的整个Hexo项目（blog）原件同步提交到github或者其他代码托管的站点。

下面记录一下备份、以及在另外的电脑上恢复博客的过程，为了以后备查。

## 前提

已创建有 GitHub 仓库，并且已使用 `hexo-deployer-git` 部署到 `master` 分支。（发布博文并托管到Github上）
如果不满足请自行 google hexo 部署到 GitHub 的操作方法。

## 备份过程

在Github网站创建一个新仓库(或者使用Github托管博客的仓库，在该仓库下创建一个新的分支)，比如我新建的仓库名为 `HEXOSource`

在本地hexo根目录中， 初始化git仓库

``` shell
git init
```

创建并切换到名为 `hexo_source` 的分支

``` shell
git checkout -b hexo_source
```

创建忽略规则文件 `.gitignore`

``` shell
vi .gitignore
```

按需添加如下内容：

```
.DS_Store 
Thumbs.db
db.json  
*.log
.deploy*/
node_modules/
.npmignore
public/
```

上面最后一行 public 目录，因其已被 hexo 插件同步到 master 分支里，因此不需要再同步，deploy 是 hexo 的 git 配置存放目录，也不需要同步。其他内容可选择忽略也可以选择同步。

添加内容到仓库并提交到远程仓库

``` shell
git add .
git commit -m "first commit"
git remote add origin git@github.com:lvraikkonen/HEXOSource.git		# 后面仓库目录改成自己新建的。
git push -u origin hexo_source
```

按照以上的步骤就进行了 hexo 源文件的初次备份。
以后每次修改了内容之后，都可通过以下几条命令实现同步。

``` shell
git add .
git commit -m "..."	 # 双引号内填写更新内容
git push origin hexo_source	# 或者 git push
```

## 新机器同步

在一个新机器上写博客，用以下步骤同步至最新状态

新建博客文件夹 `hexo_blog`

在该文件夹下初始化git仓库

``` shell
git init
```

为本地仓库添加远程仓库

``` shell
git remote add origin git@github.com:lvraikkonen/HEXOSource.git
```

切换至hexo_source分支

``` shell
git checkout -b hexo_source
```

获取`hexo_source`分支源文件

``` shell
git pull origin hexo_source
```

然后就是写博客，并将.md博客文件放至_posts文件夹，然后添加修改到本地仓库

``` shell
git add .
git commit -m "写了一篇博客"
git push origin hexo_source
```

至此，已经完成了博客的撰写并修改了远端仓库的博客源文件，然后使用`hexo g`和`hexo d`更新博客就OK啦！