+++
title = 'WebDAV远程目录共享'
date = 2025-06-27T15:22:07+08:00
draft = true
+++
本文介绍如何通过 Windows 的 WebDAV/SMB 和 Linux 的 httpd/SSH 搭建私有文件服务器，并实现跨平台远程挂载与共享。
<!--more-->
# 前言

​		在日常生活中我们难免会向别人分享一些自己收藏的珍贵学习资料，如果在同一网络情况下，我们使用windows自带的文件共享功能就可以实现。如果想要搭建自己的文件服务器向更多的朋友分享自己的学习资料，那你可能就需要我这个文章了；主要内容就是使用windows主机的webdav功能（SMB也可以）搭建私有文件服务器，使用Linux主机的httpd功能（ssh也可以）搭建私有文件服务器，windows主机挂载远程目录（webclient），linux主机挂载远程目录（sshfs、davfs）。

注：我的电脑是windows10系统、linux是麒麟3系统（其他Linux主机类似）

# 使用windows搭建webdav文件服务器

![image-20231229163801815](image-20231229163801815.png)

![image-20231229163906570](image-20231229163906570.png)

![image-20231229163951425](image-20231229163951425.png)

![image-20231229164137819](image-20231229164137819.png)



![image-20231229164651121](image-20231229164651121.png)



![image-20231229164700382](image-20231229164700382.png)

![image-20231229164727183](image-20231229164727183.png)

![image-20231229164831759](image-20231229164831759.png)

![image-20231229165001896](image-20231229165001896.png)

![image-20231229165049937](image-20231229165049937.png)

![image-20231229165207064](image-20231229165207064.png)

这样我们的文件服务器就搭建好了，鼠标右键点击网站-管理网站-启动

# 使用linux搭建httpd文件服务器

下载httpd，https://httpd.apache.org/download.cgi#apache24

![image-20231229165711614](image-20231229165711614.png)

上传到主机，解压 tar -zxvf httpd-2.4.58.tar.gz	cd  httpd-2.4.58	./configure && make && make install

安装完成后，配置共享路径和用户名密码

![image-20231229170431684](image-20231229170431684.png)

![image-20231229170501711](image-20231229170501711.png)

![image-20231229170707474](image-20231229170707474.png)

![image-20231229170800599](image-20231229170800599.png)

![image-20231229170850946](image-20231229170850946.png)

![image-20231229171226034](image-20231229171226034.png)

![image-20231229171353831](image-20231229171353831.png)

![image-20231229171702811](image-20231229171702811.png)

# windows挂载远程目录

设置 WebClient，允许 http 链接挂载
步骤1：
按下 “windows徽标键” + “R”，打开运行窗口，输入regedit，点击确定后，打开注册表编辑器窗口。

步骤2：
将路径定位到以下路径：计算机\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\WebClient\Parameters。双击右侧界面中的 BasicAuthLevel 条目，将数值数据修改为“2”，点击确定后关闭注册表编辑器。

步骤3：
按下 “windows徽标键” + “R”，打开运行窗口，输入services.msc，点击确定后，打开“服务”界面。找到 “WebClient”
服务，右键点击打开选项菜单，选择重新启动，稍等几秒，待完成后，关闭“服务”界面。

完成上述三个步骤后，WebClient 服务已经允许使用 http 协议进行挂载
![image-20231229172142554](image-20231229172142554.png)



# linux挂载远程目录

下载安装包

davfs2 ：https://savannah.nongnu.org/projects/davfs2

neon：https://notroj.github.io/neon/

Neon安装

```shell
tar -zxvf neon-0.32.5.tar.gz

cd neon-0.32.5

./configure --with-ssl

make && make install
```

 Davfs2安装

```shell
tar -zxvf davfs2-1.7.0.tar.gz
cd  davfs2-1.7.0
./configure
make && make install
```

创建用户组

```shell
groupadd davfs2
useradd davfs2 -g davfs2
```

挂载目录

```shell
mount.davfs http://192.168.100.124:8090/webdav /data/webdav
```

通过ssh协议挂载远程目录

```shell
# 挂载
sshfs username@hostname:/path/to/remote/directory /mnt/sftp -o allow_other,uid=0,gid=0
# 卸载
fusermount -u /mnt/sftp
```

