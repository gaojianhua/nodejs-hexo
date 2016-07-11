title: windows x64 redis install  
date: 2015-12-20 22:49:55
tags:
- 开始
- 我
- 日记
categories: redis
---

    Redis是一个开源的高级key-value（键-值）缓存与存储，
以高性能著称。用于做对象缓存，可以获得极佳的性能体验，可是 Redis 的官方开发团队并没有开发针对 Windows 的版本，不过还好有 “微软开放技术团队”在维护 Windows 版的 Redis。
<!-- more -->
目前 MSOpenTech 团队在维护 2.8 和 3.0 两个版本，但是 Redis 官方团队已经不对 2.8 进行开发了，而且 windows 的 3.0 版本，不仅拥有更多特性而且安装简单方便，因此我们就介绍 3.0 的安装。

### 一、下载

进入 https://github.com/MSOpenTech/redis/releases 下载最新的 3.0 版本，不过由于 Github 下载速度感人，因此我下载后提交到了国内的 git 平台。

http://git.oschina.net/mifar/php7-redis3/raw/master/Redis-x64-3.0.501.msi

### 二、安装

1.打开安装程序

请输入图片描述

2.打个勾，添加path，安装地址默认即可

请输入图片描述

3.端口默认即可，打个勾添加到防火墙

请输入图片描述

4.打个勾，设置最大内存。一般来说小、中型网站，100M～256M即可，太大会无法启动。

请输入图片描述

5.然后就ok了，默认自动后台，默认开机启动。 不像 2.8 会很麻烦。 用连接拓展连接后就可以使用了。

### 三、管理（可选）

由于 Windows 一般都是可视 GUI，所以我们也可以通过图形化工具来辅助管理 Redis。这里推荐：Redis Desktop Manager，可以管理操作 Redis。

请输入图片描述

官网：http://redisdesktop.com/download，不过已经被 ｜ Qiang 了，所以我也下载了一份提交到了国内git：

http://git.oschina.net/mifar/php7-redis3/raw/master/redis-desktop-manager-0.8.3.3850.exe

连接的话，内网， host 填 127.0.0.1 即可。

请输入图片描述

来源: https://yq.aliyun.com/articles/6396?spm=5176.100239.yqblog1.10.ddiYYY