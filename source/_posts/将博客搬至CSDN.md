---
title: 将博客搬至CSDN
date: 2020-07-27 11:23:02
tags: [Hexo]
categories:
- [Hexo]
---

 将Hexo博客同步到CSDN。注册CSDN时怎么自定义域名，怎么同步，分享同步博客到csdn的小技巧。

<!--more-->

# 1. Hexo与CSDN

因为电脑可能出现故障的原因，有时候不得不未雨绸缪，做好数据的备份、博客的同步迁移。

个人还是很喜欢Hexo写博客的方式，也考虑了HEXO和CSDN的一些利与弊，然后去做了博客的同步迁移。

## 1.2 Hexo的好处

- HEXO博客是一款采用 `Markdown` 语法书写的静态博客，写好后的源文件为 `md` 文件。然后通过Hexo的deploy命令将生成的HTML静态网页部署到服务器上，因此服务器上存的是博客的html文件。因此，博客的所有数据 `.md` 文件都在我们自己手中。
- Hexo的可自定义性比较强，可以自定义Hexo博客站点的一些美化，动态特效，新增自定义超链接页面等等。
- 没有广告，干净清爽。

## 1.3 Hexo的缺点

- 图片上传问题。本地插入图片时，图片的路径是本地路径，而部署到服务器上，其访问的是url路径，但这个问题很容易解决，两种解决方式可参考我前面的做法：1. [利用图床](https://zicair.github.io/2020/05/31/typora+picgo%E6%90%AD%E5%BB%BAgithub%E5%9B%BE%E5%BA%8A/#more)； 2. [利用Git工具](https://zicair.github.io/2020/07/28/Hexo%E5%8D%9A%E5%AE%A2%E5%A4%87%E4%BB%BD/#more)(同时备份博客)。
- 数据备份。因为自己搭建博客，所有数据在自己手上，就必须要考虑到移动办公和数据备份的因素。采用上面提到的方法2同样容易解决，而且可以在写完博客后仅通过git命令同时更新图床和博客。
- 公开性和评论问题。因为是自己搭建的并部署到github仓库上，而百度不能够抓取到github上的网页信息，因此发布的博客使用百度搜不到，当然网上也有许多解决办法。另一点是没有内置的评论系统，需要自己增加这个功能。



## 1.4 CSDN的利与弊

- 优点是只用在站点内书写，能用搜索引擎搜索到文章并有评论系统等等，一些自定义功能只有等级上来才能使用。
- 缺点是数据不在自己手中，页面不美观可自定义性差，而且会植入广告到展示的页面上。

# 2. CSDN的同步

## 2.1 注册自定义域名CSDN账户

不推荐使用手机号，这样在注册时会随机生成一串数字的用户名。

这里使用微信注册，会弹出设置用户名的选项。这里我创建时键入 `zicair` 所以进入博客页面时，域名显示的是`blog.csdn.net/zicair`。而一旦注册后想要修改则需要等到提升到相应的等级才能修改。如果不介意你的博客地址是`blog.csdn.net/255524222`这样，可以用手机号随意注册。

![image-20200729122459593](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/将博客搬至CSDN/image-20200729122459593.png)

## 2.2 博客同步到CSDN

在CSDN中点击右上角头像打开 `管理博客->博客搬家` 选项:

![image-20200729123256727](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/将博客搬至CSDN/image-20200729123256727.png)

根据你原有的博客类型选择博客搬迁的原地址。因为我是用Hexo部署到Github上的，所以选中GitHub图标。

下面需要填写部署到GitHub上的对应的仓库名称。然后申请。

## 2.3 同步效果

申请通过后，博客会经过审核。会直接同步到csdn中

![image-20200729123850647](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/将博客搬至CSDN/image-20200729123850647.png)



查看搬家记录，可以看到同步状态在`同步中`，后面有取消的开关。

![image-20200729123730915](https://raw.githubusercontent.com/zicair/MyBlog/master/picbed/将博客搬至CSDN/image-20200729123730915.png)