# Git 快速上手指南

## 1.安装

在使用Git前我们需要先安装 Git。Git 目前支持 Linux/Unix、Solaris、Mac和 Windows 平台上运行。
Git 各平台安装包下载地址为：http://git-scm.com/downloads

### Debian/Ubuntu
```
$ apt-get install libcurl4-gnutls-dev libexpat1-dev gettext \
  libz-dev libssl-dev

$ apt-get install git-core

$ git --version

```

### Centos/RedHat
```
$ yum install curl-devel expat-devel gettext-devel \
  openssl-devel zlib-devel

$ yum -y install git-core

$ git --version

```

### windows
在 Windows 平台上安装 Git 同样轻松，有个叫做 msysGit 的项目提供了安装包，可以到 GitHub 的页面上下载 exe 安装文件并运行：
安装包下载地址：https://git-for-windows.github.io/

### mac
在 Mac 平台上安装 Git 最容易的当属使用图形化的 Git 安装工具，下载地址为：
http://sourceforge.net/projects/git-osx-installer/

## 2.基本配置

Git 提供了一个叫做 git config 的工具，专门用来配置或读取相应的工作环境变量。
这些环境变量，决定了 Git 在各个环节的具体工作方式和行为。这些变量可以存放在以下三个不同的地方：
- /etc/gitconfig 文件：系统中对所有用户都普遍适用的配置。若使用 git config 时用 --system 选项，读写的就是这个文件。
- ~/.gitconfig 文件：用户目录下的配置文件只适用于该用户。若使用 git config 时用 --global 选项，读写的就是这个文件。
- 当前项目的 Git 目录中的配置文件（也就是工作目录中的 .git/config 文件）：这里的配置仅仅针对当前项目有效。每一个级别的配置都会覆盖上层的相同配置，所以 .git/config 里的配置会覆盖 /etc/gitconfig 中的同名变量。

用户信息

配置个人的用户名称和电子邮件地址：
```
$ git config --global user.name "GuoYongfeng"
$ git config --global user.email "296512521@qq.com"
```

可以查看已有的配置信息
```
$ git config --list
```

## 3.工作流程

- 克隆 Git 资源作为工作目录。
- 在克隆的资源上添加或修改文件。
- 如果其他人修改了，你可以更新资源。
- 在提交前查看修改。
- 提交修改。
- 在修改完成后，如果发现错误，可以撤回提交并再次修改并提交。


## 4.工作区、暂存区和版本库的理解
你的本地仓库由 git 维护的三棵“树”组成。第一个是你的 工作目录，它持有实际文件；第二个是 缓存区（Index），它像个缓存区域，临时保存你的改动；最后是 HEAD，指向你最近一次提交后的结果。

<img src="/img/git/flow.png" />

## 5.基本操作

### git init
新建一个仓库
```
$ mkdir demo && cd demo
$ git init
```
### git clone
如果已经有了仓库，就可以直接clone到本地

```
$ git clone git@github.com:GuoYongfeng/webpack-dev-boilerplate.git
```

### git add
git add 命令可将该文件添加到缓存，如我们添加以下两个文件：
```
$ touch README.md
$ git add README.md
```
还可以
```
$ git add -A
$ git add *
```

### git status

git status 可以查看当前版本库各个文件的状态
```
$ git status
```

### git commit
使用 git add 命令将想要快照的内容写入缓存区， 而执行 git commit 将缓存区内容添加到仓库中。

```
$ git commit -m '第一次版本提交'
```

### git reset
git reset HEAD 命令用于取消已缓存的内容。
```
$ git reset HEAD -- hello.php
```

如果粗暴一点
```
$ git reset --hard 版本号
```

### git rm
```
$ git rm README.md
```

## 6.push到服务器

首先需要在本地生成key，并且把key配置在github上

```
$ ssh-keygen -t rsa -C "guoyff@yonyou.com"
```


### 开通github账号，方便技术交流和信息获取

- 请访问：https://github.com
- 然后注册一个账号


### 设置public key

```
$ cat ~/ssh/id_rsa.pub
```

将获得的public key添加在github账户上：

> 右上角点击头像-> 点击settings-> 点击SSH KEYS-> 点击ADD SSH KEYS-> 将获取的public key粘贴于此

### 在github上新建一个仓库

<img src="/img/git/repo.png" />

### 最基础的练习

```
# 创建一个目录
$ mkdir test
# 进入所创建的目录
$ cd test
# 初始化一个git仓库
$ git init
# 为这个仓库添加一个远程地址
$ git remote add origin 你的github上的仓库地址（比如： git@github.com:GuoYongfeng/webpack-dev-boilerplate.git）
# 新建两个测试文件
$ touch .gitignore README.md
# 把文件添加到暂存区
$ git add --all
# 把文件添加到本地版本库
$ git commit -m "comments"
# 将本地版本库的资源推送到远程服务器
$ git push -u origin master
```
