---
title: typora+picgo搭建github图床
date: 2020-05-31 14:04:49
tags: Hexo
categories:
- Hexo
---

windos10系统下，使用typora+picgo组合进行图床的设置，配置图床后直接上传图片到图床能够获取到图片连接。

<!--more-->



##  1. 安装PicGo

打开typora软件，依次点开`文件->偏好设置->`, 直接下载PicGo(app)，安装完成后，将`PicGo路径`设置为你刚安装的PicGo.exe的路径。

![typora+picgo搭建github图床_1](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/typora+picgo搭建github图床/typora+picgo搭建github图床_1.png)

PicGo(app)是单独下载一个软件，当然也可以选择PicGo-core(comand-line)，它是为typora安装插件，不需要下载额外的软件，只需要配置一个json配置文件即可。这里我使用PicGo(app)的方式，他可以在软件界面的相册中记录我们上传过的图片，可以方便重复获取之前使用的图片链接。





## 2. GitHub上创建图床仓库

### 2.1 创建仓库

点击头像旁边的 "+" ，选择`New repository`。

![typora+picgo搭建github图床_2](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/typora+picgo搭建github图床/typora+picgo搭建github图床_2.png)



在`Repository name`栏中键入要创建的仓库名，然后选择`create repository`。

![typora+picgo搭建github图床_3](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/typora+picgo搭建github图床/typora+picgo搭建github图床_3.png)



### 2.2 生成tokens

访问：https://github.com/settings/tokens， 然后点击`Generate new token`。

![typora+picgo搭建github图床_4](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/typora+picgo搭建github图床/typora+picgo搭建github图床_4.png)

勾选`repo`, 拉到底部，选择`Generate token`生成token。

![typora+picgo搭建github图床_5](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/typora+picgo搭建github图床/typora+picgo搭建github图床_5.png)





##  3. 配置PicGo和typora

### 3.1 PicGo图床设置

打开PicGo软件，点击`图床设置->Github图床`，将生成的token复制进去，其余的按要求进行填写即可，最后一项的域名设置为：

`https://raw.githubusercontent.com/账户名/仓库名/master`

![typora+picgo搭建github图床_6](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/typora+picgo搭建github图床/typora+picgo搭建github图床_6.png)



### 3.2 typora设置

打开`偏好设置->图像`，我的设置如下，在typora插入图片时，会将图片复制一份到本地的`source/picbed/img`，这里的picbed文件夹是clone我所创建的仓库，因为所有的图片是上传到GitHub的picbed/img路径下，本地的picbed是为了在本地方便维护，使得图床中的图片和本地picbed/img路径中是一致的。通过git命令可以在本地路径中维护github图床中的图片。

![typora+picgo搭建github图床_7](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/typora+picgo搭建github图床/typora+picgo搭建github图床_7.png)



typora插入图片时，会自动生成本地的url：

![typora+picgo搭建github图床_8](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/typora+picgo搭建github图床/typora+picgo搭建github图床_8.png)



当博客部署到网上时，并不能识别本地的url，因此图片是不能显示的，这里右键图片，选择`上传图片`，图片会上传到github图床中，并将url自动改为图床中的图片链接。这样，当博客部署到github上时，图片的连接指向图床的图片链接。

![typora+picgo搭建github图床_9](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/typora+picgo搭建github图床/typora+picgo搭建github图床_9.png)