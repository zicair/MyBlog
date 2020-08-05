---
title: Hexo博客备份
date: 2020-07-28 17:26:50
tags: [Hexo]
categories:
- [Hexo]
---

部署博客时提交的是生成后的静态网页，而源文件是在我们使用的电脑上，可以使用Git工具备份这些博客文件和设置文件，并将图床搭配进来。

<!--more-->

# Hexo博客备份

## 1. 存在的问题

上周使用笔记本过程中出现了故障， wifi识别不到网络，重置网络后电脑关机准备自动重启就再也打不开机了，当天下午就去了售后维修点。服务员确认了电脑故障后就交给了维修人员，并提醒我可能会重装系统，有没有备份之类的，然而打不开机的我又怎么去备份数据，这时才突然警醒数据放在笔记本上并不安全，比如写的一些博客的源文件，一些自定义的配置文件等等，，，，结果在维修员手里直接就打开了，Fuck。。。。

第二次出现故障，直接送修，确认是网卡和主板的连接点受到了腐蚀，等了两天，换了网卡和主板。好在是好在是在保修期内，，，，

但这也让我开始了考虑博客备份迁移。

### 1.1 博客备份问题

Hexo博客是使用Markdown写成的.md文件。当部署到服务器上时生成的是静态网页html文件。而源文件都在固定的一台电脑上。而使用U盘备份又不方便。

![image-20200728180234603](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/Hexo博客备份/image-20200728180234603.png)

### 1.2 图床问题

之前使用picbed图床工具，在上传图片到github上容易出现上传失败的情况。体验很差。所以就想到同样把Github当做图床，使用git工具直接通过命令上传。

所以就想到在备份博客的一些文件时直接把图像也备份到github仓库中，再把仓库中的图像文件夹当做图床。这样在写博客时不用每次去点击上传图片，只需要在写完后，通过`ctrl + H`直接替换github上的图床路径即可。

这样就实现了通过一次git更新，备份了博客，也更新了图床。

## 2. 解决方案

### 2.1 连接仓库

1、首先在MyBlog工作目录下打开git bash，输入

```git
git init
```

2、在github上创建仓库，这里我创建为`MyBlog`

3、然后在本地连接仓库地址

```git
git remote add origin https://github.com/zicair/MyBlog.git
```



### 2.2 选择要备份的文件

选择要备份哪些文件或者文件夹，其中包含了要备份的图床文件夹`picbed`

```git
$ git add picbed/ scaffolds/ source/ themes/ .gitignore _config.yml
```

picbed设置：图床文件夹，写博客时用到的图片会自动复制一份到这个文件夹中，在偏好设置中进行设置

![image-20200728181602018](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/Hexo博客备份/image-20200728181602018.png)

### 2.3 备份上传

commit备份内容

```ruby
git commit -m "博客备份"
```

push备份内容

```git
git push -u origin master
```


第一次备份完毕，以后在`hexo d`之后，只需跟着进行如下操作：

```git
git add source/ picbed/
git commit -m ""
git push
```

其中source文件夹中包含着新写的博客，picbed是图床文件夹。

这样备份完毕后，我们在另一台电脑上，只需`git clone`一下就行了。

```
git push -u origin master 
如果当前分支与多个主机存在追踪关系，-u选项指定一个默认主机，
这样后面就可以不加任何参数使用git push。

git push
默认只推送当前分支，这叫做simple方式。
```

