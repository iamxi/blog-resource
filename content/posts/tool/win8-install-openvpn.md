---
title: "Win8.1下安装OpenVPN"
date: 2014-02-11T09:26:00+08:00
draft: false
description:  我在win8.1下安装OpenVPN遇到了两个问题，一是安装时候TAP设备驱动程序安装失败，二是因为系统临时目录带中文而无法正确连接，下面是解决方法。
tags: [Tool]
slug: win8-install-openvpn
---

我在win8.1下安装OpenVPN遇到了两个问题，一是安装时候TAP设备驱动程序安装失败，二是因为系统临时目录带中文而无法正确连接，下面是解决方法。

系统及软件版本

* Win8.1 64位系统
* OpenVPN 2.3.2 64位

##一、安装TAP设备驱动程序失败

在安装OpenVPN时候，在最后报了**“An error occurred installing the TAP device driver”**这样的错误。这是说安装TAP驱动时候错误，可以不理会，这个时候OpenVPN其实已经安装完成。后面要做的事就是自己动手来安装TAP驱动了。

不过手动安装虚拟网络设备的时候需要知道失败的原因，解决后才能在安装。在Win7下安装是正常的，而Win8/8.1就不行了，主要是因为系统开启了“驱动程序强制签名”，而在Win8.1下不认OpenVPN的TAP驱动程序签名导致的。这个不太清楚是OpenVPN版本有关还是和系统版本有关系。解决办法就是禁用驱动程序强制签名，步骤如下：

* 1、按住Shift建不放，然后用鼠标点击右侧的Charm菜单中的重启按钮（注意，这里会重启系统，有文档没有保存的保存下）。
* 2、在出现的界面中依次选择：疑难解答->高级选项->启动设置->重新启动
* 3、在重启后的界面上选择“禁用驱动强制签名”
* 4、用Charm菜单中的搜索功能搜索“add a new TAP virtual ethernet adapter”，以管理员身份运行，看到系统提示后点“始终...”，之后就算不在禁用驱动程序强制签名的模式下也可以正确安装。

##二、系统临时目录中文问题

我的微软账号中的名字是中文，所以在生成的用户目录也是中文的，在用户目录下的临时文件夹自然在路径上也带了中文，而OpenVPN的GUI程序默认在使用临时文件夹。看来OpenVPN的GUI程序也不支持中文啊。失败后在日志中看到的错误信息就是“--tmp-dir”这个参数指定的目录错误。如果使用命令行的话完全可以通过“--tmp--dir”参数指定其他的，不过那样不方便。

其实自己的client.ovpn文件也同样可以指定参数，在里面加入一行“tmp-dir "e:\\temp"”（注意，这个路径必须有效），然后再重新打开OpenVPN就可以正常连接。再多说一句，Win7/8/8.1下，需要以管理员身份运行OpenVPN，不然显示连接上了，但是依然无法连接远程服务器。
