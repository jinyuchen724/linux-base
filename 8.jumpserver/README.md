# 系统功能概述

1.Jumpserver 是全球首款完全开源的堡垒机，使用 GNU GPL v2.0 开源协议，是符合 4A 的专业运维审计系统。  
2.Jumpserver 使用 Python / Django 进行开发，遵循 Web 2.0 规范，配备了业界领先的 Web Terminal 解决方案，交互界面美观、用户体验好。  
3.Jumpserver 采纳分布式架构，支持多机房跨区域部署，中心节点提供 API，各机房部署登录节点，可横向扩展、无并发访问限制。  

# 部署环境

| 主机   |   角色   |   操作系统 |   软件版本  |    备注  |
| ------ | ----- | ----- | ------- | ------ |
| jumpserverhost-1(10.1.1.2)  | server-core,coco | Centos 7.2(x86-64) | 1.3.3.2  |  主节点|
| jumpserverhost-2(10.1.1.3)  | 只需ssh | Centos 7.2(x86-64) |   |  客户节点|

# 架构说明及管理文档

```
http://docs.jumpserver.org/zh/docs/admin_instruction.html
```

# 安装

> 官方的安装文档十分详细,这里不赘述了。
- 根据官方文档安装即可

```
http://docs.jumpserver.org/zh/docs/step_by_step.html
```

# 用户使用文档

```
http://docs.jumpserver.org/zh/docs/user_guide.html
```

# API调用


```
http://10.1.1.2/docs/

包括不限于:
1.创建删除节点
2.创建删除资产
3.创建删除对应权限

python调用API
见jump.py
```

# FAQ

- sftp传输文件

使用sftp登录跳板机

```
       #登录端口 #账号   #跳板机地址
$ sftp -P2222 chenjinyu@10.1.1.2
chenjinyu@10.1.1.2's password: 
Please enter 6 digits.
[MFA auth]: 608840       #google验证码

# 显示有权限上传下载文件的主机
sftp>ls
jumpserverhost-2

sftp> cd jumpserverhost-2 #这里是你有权限的资产
sftp> cd SA                                #这里是你的账号目录
sftp> ls                                   #这里可以使用get/put上传下载文件
hosts        hsperfdata_www                                                                                       
initos.log   initosPuppet7.sh                                                                                     
```

- 历史回溯

![image](https://github.com/jinyuchen724/linux-base/blob/master/8.jumpserver/jump1.png)

![image](https://github.com/jinyuchen724/linux-base/blob/master/8.jumpserver/jump2.png)

