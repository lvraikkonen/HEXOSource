title: git 常用命令总结
date: 2016-09-20 11:20:32
tags: 
- git
- github
categories: 
- 备忘

---

![gitlogo](http://7xkfga.com1.z0.glb.clouddn.com/1473343759-1415241085-Git-Logo-2Color.png)

Git，目前主流的版本控制工具，git命令是一些命令行工具的集合，它可以用来跟踪，记录文件的变动。比如你可以进行保存，比对，分析，合并等等。

## 基本的git工作流

1. 在工作目录中修改文件
2. 暂存文件，将文件的快照放入暂存区域
3. 提交更新，找到暂存区域的文件，将快照永久性存储到 Git 仓库目录

![git workflow](http://7xkfga.com1.z0.glb.clouddn.com/566304-ddcf785586305023.png)

<!-- more -->

### 分支

当你在做一个新功能的时候，最好是在一个独立的区域上开发，通常称之为分支。分支之间相互独立，并且拥有自己的历史记录。这样做的原因是：

- 稳定版本的代码不会被破坏
- 不同的功能可以由不同开发者同时开发
- 开发者可以专注于自己的分支，不用担心被其他人破坏了环境
- 在不确定之前，同一个特性可以拥有几个版本，便于比较

以上，简单介绍了一下git的基本概念，下面记录一下如下几个git命令以及使用：
- git clone
- git remote
- git commit
- git push
- git fetch
- git pull


### git clone
从远程主机克隆一个版本库，git支持SSH, Git, HTTPS等

``` shell
git clone <远程仓库> <本地文件夹名>
```

### git remote
git remote命令用于管理主机名

``` shell
git remote
```

### git commit
将索引内容添加到仓库中

``` shell
git commit  -m "提交的描述信息"
```

### git push
git push命令用于将本地分支的更新，推送到远程主机

``` shell
git push <远程主机名> <本地分支名>:<远程分支名>
```

### git fetch
git fetch命令从远程主机的更新取回本地

``` shell
git fetch <远程主机名> <分支名>
```

### git pull
git pull命令的作用是，取回远程主机某个分支的更新，再与本地的指定分支合并

``` shell
git pull <远程主机名> <远程分支名>:<本地分支名>
```


版权声明：<br>
<hr>
除非注明，本博文章均为原创，转载请以链接形式标明本文地址。<br>
