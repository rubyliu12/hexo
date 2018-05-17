---
title: Linux安装FTP
categories:
  - Linux
tags:
  - FTP
abbrlink: 9c4dd72b
date: 2017-03-21 15:25:05
---

# Linux安装FTP
ubuntu开启ftp服务
安装vsftpd
1、打开"终端窗口"，输入"sudo apt-get install vsftpd"-->回车-->安装完成。

2、判断vsftpd是否安装成功
打开"终端窗口"，输入"sudo service vsftpd restart"重启vsftpd服务-->回车-->vsftpd处于运行状态，说明安装成功。

3、使用gedit修改配置文件/etc/vsftpd.conf
打开"终端窗口"，输入"sudo gedit /etc/vsftpd.conf"-->回车-->打开了vsftpd.conf文
设置：write_enable=YES

ubuntu开启SSH服务
SSH分客户端openssh-client和openssh-server
如果你只是想登陆别的机器的SSH只需要安装openssh-client（ubuntu有默认安装，如果没有则sudo apt-get install openssh-client），如果要使本机开放SSH服务就需要安装openssh-server
sudo apt-get install openssh-server
然后确认sshserver是否启动了：
ps -e |grep ssh
如果看到sshd那说明ssh-server已经启动了。
如果没有则可以这样启动：sudo /etc/init.d/ssh start
ssh-server配置文件位于/ etc/ssh/sshd_config，在这里可以定义SSH的服务端口，默认端口是22，你可以自己定义成其他端口号，如222。
然后重启SSH服务：
sudo /etc/init.d/ssh stop
sudo /etc/init.d/ssh start
然后使用以下方式登陆SSH：
ssh username@192.168.1.112 username为192.168.1.112 机器上的用户，需要输入密码。
断开连接：exit

最近一直在Ubuntu下用终端操作，看着他长长的路径名实在不爽， 动手来改改咯～
$: sudo gedit ~/.bashrc
这个文件记录了用户终端配置
找到
if [ "$color_prompt " = yes ]; then
    PS1 ='${debian_chroot:+($debian_chroot)}\[\033[01;32m\]\u@\h\[\033[00m\]:\[\033[01;34m\]\W \[\033[00m\]\$ '
else
    PS1 ='${debian_chroot:+($debian_chroot)}\u@\h:\W \$ '
将蓝色的w由小写改成大写，可以表示只显示当前目录名称.
