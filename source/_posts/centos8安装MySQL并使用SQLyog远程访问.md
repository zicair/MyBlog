---
title: centos8安装MySQL并使用SQLyog远程访问
date: 2020-06-13 22:22:20
tags: [centos, mysql]
categories:
- [mysql]
---

在虚拟机中安装centos8并安装MySQL，在本地使用SQLyog远程访问MySQL数据库。

<!--more-->

# 1. 安装VMware pro 15（破解）

VMware安装没有什么可注意的，下载安装一路下一步即可。

前往[VMware官网](https://www.vmware.com/products/workstation-pro/workstation-pro-evaluation.html)下载VMware Workstation Pro 15，然后以管理员身份运行安装。

到这一步时，点击`许可证`，实现破解

![centos8安装MySQL并使用SQLyog远程访问_1](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/centos8安装MySQL并使用SQLyog远程访问/centos8安装MySQL并使用SQLyog远程访问_1.png)

随便选择一个注册码：

```
AZ7MK-44Y1J-H819Z-WMYNC-N7ATF
CU702-DRD1M-H89GP-JFW5E-YL8X6
YY5EA-00XDJ-480RP-35QQV-XY8F6
UY758-0RXEQ-M81WP-8ZM7Z-Y3HDA
VF750-4MX5Q-488DQ-9WZE9-ZY2D6
UU54R-FVD91-488PP-7NNGC-ZFAX6
YC74H-FGF92-081VZ-R5QNG-P6RY4
YC34H-6WWDK-085MQ-JYPNX-NZRA2
```

# 2. centos8下载安装

没什么需要注意的，一路默认，看提示安装即可。

- [官网下载](https://www.centos.org/)
- VMware新建虚拟机（自定义安装-NAT联网）
- 设置里添加centos8.iso文件，运行虚拟机安装centos

# 3. CentOS 8上安装MySQL 8.0

```java
// @mysql模块安装MySQL及其所有依赖项。
sudo dnf install @mysql
// 启动MySQL服务并使其在启动时自动启动
sudo systemctl enable --now mysqld
// 检查MySQL服务器是否正在运行
sudo systemctl status mysqld
```

运行`mysql_secure_installation`脚本执行一些安全性设置

```
sudo mysql_secure_installation
```

进入配置:

- 选择密码验证策略等级，选择0 （low），回车
- 输入密码两次
- 确认是否继续使用提供的密码？输入y ，回车
- 移除匿名用户？ 输入y ，回车
- 不允许root远程登陆？ 输入n ，回车
- 移除test数据库？ 输入y ，回车
- 重新载入权限表？ 输入y ，回车

# 4. 本机安装SQLyog并连接mysql

下载SQLyog

链接：https://pan.baidu.com/s/1fZBlIP7fwRcTVJHTqUiyCg 
提取码：hesr

解压到指定路径即可。

安装过程中需要填写注册码：

```
　　姓名(Name)：ddooo;
　　证书秘钥：8d8120df-a5c3-4989-8f47-5afc79c56e7c;

　　姓名(Name)：cr173
　　序列号(Code)：8d8120df-a5c3-4989-8f47-5afc79c56e7c

　　姓名(Name)：cr173
　　序列号(Code)：59adfdfe-bcb0-4762-8267-d7fccf16beda

　　姓名(Name)：cr173
　　序列号(Code)：ec38d297-0543-4679-b098-4baadf91f983
```

选一个填到软件的注册框内，点击“注册”按钮，sqlyog会自动检测注册信息;

**为了使SQLyog能够连接到mysql，需要做如下几点：**

- 关闭虚拟机中centos的防火墙

  `systemctl stop firewalld.service && systemctl disable firewalld.service`

- 在mysql中将root用户的host字段设为'%'：

  (否则会在测试连接时报异常：Host is not allowed to connect to this MySQL server)

  ```
  mysql -uroot -p{password}
  use mysql; 
  update user set host='%' where user='root'; 
  flush privileges;
  ```

  