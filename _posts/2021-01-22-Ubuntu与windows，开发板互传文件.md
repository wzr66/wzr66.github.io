---
title:        Ubuntu与windows，开发板互传文件
description:  Ubuntu安装并开启nfs，ssh，ftp，tftp服务，与windows，开发板互传文件
author:       WZR
categories:
    - Linux
tags:
    - Linux
    - ubuntu
    - nfs
	- ssh
	- ftp
	- tftp
---

>Ubuntu安装并开启nfs，ssh，ftp，tftp服务，与windows，开发板互传文件，系统为Ubuntu16.04LTS，虚拟机为VM15Pro，电脑系统为WIN10，开发板为正点原子阿尔法IMX6ULL。

<!-- more -->

# FTP

Ubuntu终端中执行命令安装FTP服务

```shell
sudo apt-get install vsftpd
```

等待软件安装完成后，打开ftp配置文件

```shell
sudo vim /etc/vsftpd.conf
```

找到28-31行取消"local_enable=YES"与"write_enable=YES"的注释

![image-20210130142208781](https://gitee.com/wziru/BlogPicGo/raw/master/img/20210130142208.png)

修改完成后保存退出，重启FTP服务

```shell
sudo /etc/init.d/vsftpd restart
```

# SSH

安装ssh服务

```shell
sudo apt-get install openssh-server
```

配置文件为/etc/ssh/sshd_config，这里默认即可

# NFS

Ubuntu终端中执行命令安装NFS服务

```shell
sudo apt-get install nfs-kernel-server rpcbind
```

等待软件安装完成后，在用户根目录下新建linux文件夹，linux目录下新建nfs文件夹，以后我们需要用nfs传输的文件都放到这个文件夹里。

```shell
~$ mkdir linux
~$ mkdir linux/nfs
```

打开nfs配置文件

```shell
sudo vim /etc/exports
```

在末尾加上以下内容

```sh
/home/wzr/linux/nfs *(rw,sync,no_root_squash)
```

![image-20210130144021278](https://gitee.com/wziru/BlogPicGo/raw/master/img/20210130144021.png)

重启nfs服务

```shell
sudo /etc/init.d/nfs-kernel-server restart
```

# TFTP

安装tftp服务

```shell
sudo apt-get install tftp-hpa tftpd-hpa 
sudo apt-get install xinetd
```

等待软件安装完成后，在用户根目录下linux文件夹内新建tftpboot文件夹并修改tftpboot文件夹权限为777，以后我们需要用tftp传输的文件都放到这个文件夹里，并且都要给予权限

```shell
~$ mkdir linux/tftpboot
~$ chmod 777 linux/tftboot
```

配置tftp，新建文件/etc/xinetd.d/tftp

```shell
sudo vim /etc/xinetd.d/tftp
```

输入配置内容

```shell
server tftp 
{
	socket_type 	= dgram
	protocol 		= udp
	wait 			= yes
	user			= root
	server			= /usr/sbin/in.tftpd
	server_args		= -s /home/wzr/linux/tftpboot/
	disable			= no
	per_source		= 11
	cps				= 100 2
	flags			= IPv4
}
```

![image-20210130145611656](https://gitee.com/wziru/BlogPicGo/raw/master/img/20210130145611.png)

保存退出后启动tftp服务

```shell
sudo service tftpd-hpa start
```

打开/etc/default/tftpd-hpa 文件

```shell
sudo vim /etc/default/tftpd-hpa
```

修改内容如下：

```shell
# /etc/default/tftpd-hpa

TFTP_USERNAME="tftp"
TFTP_DIRECTORY="/home/wzr/linux/tftpboot"
TFTP_ADDRESS=":69"
TFTP_OPTIONS="-l -c -s"
```

![image-20210130150417925](https://gitee.com/wziru/BlogPicGo/raw/master/img/20210130150417.png)

保存退出，重启tftp服务器

```shell
sudo service tftpd-hpa restart
```

# Ubuntu与Windows互传文件

使用FileZilla软件在Ubuntu与Windows之间互传文件。

Ubuntu作为服务器，Windows作为客户端，我们在Windows下安装客户端软件，下载地址：https://www.filezilla.cn/download/client

下载好后双击安装，完成后打开FileZilla软件，界面如下

![image-20210130151055117](https://gitee.com/wziru/BlogPicGo/raw/master/img/20210130151055.png)

ctrl+s打开站点管理器

![image-20210130151356304](https://gitee.com/wziru/BlogPicGo/raw/master/img/20210130151356.png)

## FTP

新建站点UbuntuImx6ull，选择协议为FTP，加密为明文，主机输入Ubunut主机的ip地址，用户及密码为Ubuntu用户名及密码。

点击连接，成功连接至Ubuntu主机，可以在远程站点下看到用户目录，在本地站点选择好文件后右键上传至远程站点路径，在远程站点选择好文件后右键下载至本地站点路径

![image-20210130152149928](https://gitee.com/wziru/BlogPicGo/raw/master/img/20210130152150.png)

## SSH

打开站点管理器选择协议为SFTP-SSH，也可建立连接传输文件

![image-20210130152537937](https://gitee.com/wziru/BlogPicGo/raw/master/img/20210130152538.png)

# Ubuntu与开发板uboot互传文件

## NFS

启动开发板uboot，保证能ping通Ubuntu主机。

在Ubuntu中将文件放到之前建立的/home/wzr/linux/nfs目录下，

![image-20210130153410899](https://gitee.com/wziru/BlogPicGo/raw/master/img/20210130153410.png)

在uboot中通过nfs命令下载该文件到指定位置，命令格式如下:

```shell
nfs [loadAddress] [[hostIPaddr:]bootfilename]
```

loadAddress为指定的DRAM地址，[[hostIPaddr:]bootfilename]则是Ubuntu主机服务器ip地址和对应路径文件

现在使用 nfs 命令来将 zImage 下载到开发板DRAM的 0X80800000 地址处：

```shell
=>nfs 80800000 192.168.146.20:/home/wzr/linux/nfs/zImage
```

执行后会显示下载进度

## TFTP

在Ubuntu中将文件放到之前建立的/home/wzr/linux/tftpboot目录下，并且给予权限

![image-20210130154356358](https://gitee.com/wziru/BlogPicGo/raw/master/img/20210130154356.png)

在uboot中通过tftp命令下载该文件到指定位置，命令格式如下:

```shell
tftp [loadAddress] [bootfilename]
```

loadAddress为指定存放的DRAM地址，[bootfilename]则是Ubuntn传输文件名

现在使用tftp命令来将 zImage 下载到开发板DRAM的 0X80800000 地址处：

```shell
=>tftp 80800000 zImage
```

执行后会显示下载信息

![image-20210130154747208](https://gitee.com/wziru/BlogPicGo/raw/master/img/20210130154747.png)



