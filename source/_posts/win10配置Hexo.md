---
title: win10配置Hexo
date: 2020-04-07 22:04:58
tags: Hexo
categories:
- Hexo
---

介绍windos10安装配置Hexo博客并部署到github

<!--more-->

## 一、预装软件

- ### 安装Git

  搭建Hexo博客需要有个服务器来部署我们的博客，这里将博客部署到GitHub，免去额外配置服务器的麻烦，所以首先需要有个GitHub账号。Windows电脑本机要先安装[Git工具](https://git-scm.com/download/win)。下载安装程序一直点击下一步。

- ### 安装nodejs

  到[nodejs官网](https://nodejs.org/en/download/)下载Windows版本的安装包，按提示安装即可，默认包含node和npm包管理工具。环境变量会在安装时自动添加。

  ---

  安装完成后，在任意位置，比如桌面，单击鼠标右键，点击Git Brash Here，依次执行：

  ```
node -v
  npm -v
  ```
  
  如果返回版本号信息，说明安装成功。

  

## 二、配置SSH

- 因为我们需要部署到你的github仓库，每次更改都要deploy ，如果不配置ssh key 每次你都需要输入github 账号密码，如果你觉得无所谓或者麻烦也可以跳过此步骤。

  在Git bash中输入：

  ```
  git config --global user.name "yourname"
  git config --global user.email "youremail"
  ```

  yourname是你的GitHub账号用户名，youremail是注册GitHub的邮箱地址。

  然后生成秘钥文件：

  ```
  ssh-keygen -t rsa -b 4096 -C "email@example.com"
  ```

  此时，会在C盘生成秘钥文件：![win10配置Hexo_1](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/win10配置Hexo/win10配置Hexo_1.png)

  用notepad或者记事本打开id_rsa.pub文件，将里面的内容复制。然后到GitHub个人中心，点击右上角的头像，点击弹出菜单中的setting选项，新添加一个SSH keys， 将复制的内容粘贴进去：![win10配置Hexo_2](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/win10配置Hexo/win10配置Hexo_2.png)

  回到Windows桌面打开的git bash中，输入

  ```
  ssh -T git@github.com
  ```

  ```
  The authenticity of host 'github.com (52.74.223.119)' can't be established.
  RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
  Are you sure you want to continue connecting (yes/no)?
  ```

  在返回提示信息后输入`yes`, 回车。

  ```
  Hi ***! You've successfully authenticated, but GitHub does not provide shell access.
  ```

  到此配置结束。

## 三、Hexo安装及部署

- ### 安装hexo

  在git bash终端窗口中输入`npm install hexo-cli -g` ，进行hexo的安装。

  在自定义的位置新建一个文件夹用作博客的工作空间，比如`E：/MyBlog`. 命令终端切换到`Myblog`目录中，或者直接在该目录下右键， 选择`git bash here`。在终端中输入

  ```
  hexo init
  ```

- ### hexo+git

  Blog文件夹内安装git部署插件：

  ```
  npm install --save hexo-deployer-git
  ```

- ### 部署

  修改`_config.yml`文件，拉到最下面按如下修改：
  
  ```
  deploy:
    type: git
    repo: https:....
  branch: master
  ```

  repo: 后面替换成自己GitHub仓库的地址。
  
  ```
hexo clean && hexo g && hexo s
  ```

  按照提示在浏览器中输入`localhost:4000` ,查看生成的博客。`ctrl+c`退出，再输入
  
  ```
hexo d
  ```

  将博客部署到github上。此时就可以使用浏览器上网浏览了。
  
  

## 四、typora安装使用

hexo博客使用Markdown语法，其中typora是比较好用的软件，可以实时显示编写的可视化结果。

- ### 安装typora

  到[typora官网](https://typora.io/#windows) 下载win64位的安装包，一路next即可。![win10配置Hexo_3](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/win10配置Hexo/win10配置Hexo_3.png)

- ### 配置typora

  新建一个博客时，需要使用hexo命令new一个博客`hexo new "博客"`

  我们新建的博客都在Myblog/source/_post/目录下。我们希望打开typora可以更方便地直接打开或者修改某一篇博客。首先打开typora，左上角点击文件->偏好设置，如下设置![win10配置Hexo_4](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/win10配置Hexo/win10配置Hexo_4.png)

  这样设置时，我们可以在typora界面按快捷键`Ctrl+shift+L`直接调出文件目录，也可以选择显示的方式列表视图或者树视图。

  ![win10配置Hexo_5](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/win10配置Hexo/win10配置Hexo_5.png)

  

  

  

  

