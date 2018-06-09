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
# wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
# rpm -ivh epel-release-latest-7.noarch.rpm
```

- 建立pritunl仓库文件(私有仓库已经有了,可跳过) 

```
# cat /etc/yum.repos.d/pritunl.repo
[pritunl]
name=Pritunl Repository
baseurl=https://repo.pritunl.com/stable/yum/centos/7/
gpgcheck=0
enabled=1
```

- 安装pritunl

```
# yum install pritunl mongodb-server
# systemctl start mongod pritunl
# systemctl enable mongod pritunl
```

- 获取setup key

```
# pritunl setup-key
f40fd5c985aa4806847525d8bba8727f
```

> 将key输入对话框

![image](https://github.com/jinyuchen724/linux-base/raw/master/7.openvpn/vpn1.jpg)

> 将key输入到web界面的图中,保存继续,会出现登录界面,默认登录用户名和密码都是 pritunl/pritunl

![image](https://github.com/jinyuchen724/linux-base/raw/master/7.openvpn/vpn2.jpg)

> 登录后，如下页面自行设置新的用户/密码(公网IP为vpn自己识别到的IP)

![image](https://github.com/jinyuchen724/linux-base/raw/master/7.openvpn/vpn3.png)

> 添加组织

![image](https://github.com/jinyuchen724/linux-base/raw/master/7.openvpn/vpn4.png)

> 添加用户,将用户关联到组织,设置用户密码(Pin就是后面客户端连接时的密码，Name就是用户名) 

![image](https://github.com/jinyuchen724/linux-base/raw/master/7.openvpn/vpn5.png)

> 添加服务

![image](https://github.com/jinyuchen724/linux-base/raw/master/7.openvpn/vpn6.png)

> 配置servers dns及端口

![image](https://github.com/jinyuchen724/linux-base/raw/master/7.openvpn/vpn7.png)

> 将服务与组织关联 

![image](https://github.com/jinyuchen724/linux-base/raw/master/7.openvpn/vpn8.png)

> 开启服务

![image](https://github.com/jinyuchen724/linux-base/raw/master/7.openvpn/vpn9.png)

# 系统管理

> 基本的概念是组织 --> 用户 --> server 

![image](https://github.com/jinyuchen724/linux-base/raw/master/7.openvpn/vpn10.png)

上图是后台vpn server 状态图 

# 安装openvpn客户端




