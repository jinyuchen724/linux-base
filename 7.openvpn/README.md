# 系统功能概述

OpenVPN是Linux下的一款开源VPN，提供了良好的性能和友好的用户GUI，
目前OpenVPN能在 Solaris、Linux、OpenBSD、FreeBSD、NetBSD、Mac OS X与Microsoft Windows以及Android和iOS上运行，并包含了许多安全性的功能。

![image](https://github.com/jinyuchen724/linux-base/raw/master/7.openvpn/vpn1.jpg)


# 部署环境

| 主机   |   角色   |   操作系统 |   软件版本  |    备注  |
| ------ | ----- | ----- | ------- | ------ |
| hz01-prod-ops-openvpn(172.16.2.96)  | server  |  Centos 7.3(x86-64)|  pritunl-1.24.1025.78-1.el7.centos.x86_64 + openvpn-2.4.3-1.el7.x86_64 + mongodb-server-2.6.12-4.el7.x86_64 |  主节点|

# 安装

- 安装epel仓库，有的话直接跳过

```
$ wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
$ rpm -ivh epel-release-latest-7.noarch.rpm
```

- 建立pritunl仓库文件(私有仓库已经有了,可跳过) 

```
$ cat /etc/yum.repos.d/pritunl.repo
[pritunl]
name=Pritunl Repository
baseurl=http://repo.pritunl.com/stable/yum/centos/7/
gpgcheck=0
enabled=1
```

- 安装pritunl

```

```