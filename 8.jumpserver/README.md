# 系统功能概述

Jumpserver 是全球首款完全开源的堡垒机，使用 GNU GPL v2.0 开源协议，是符合 4A 的专业运维审计系统。
Jumpserver 使用 Python / Django 进行开发，遵循 Web 2.0 规范，配备了业界领先的 Web Terminal 解决方案，交互界面美观、用户体验好。
Jumpserver 采纳分布式架构，支持多机房跨区域部署，中心节点提供 API，各机房部署登录节点，可横向扩展、无并发访问限制。

# 部署环境

| 主机   |   角色   |   操作系统 |   软件版本  |    备注  |
| ------ | ----- | ----- | ------- | ------ |
| idc02-daily-jumpserverhost-112540(10.1.125.40)  | server-core,coco | Centos 7.2(x86-64) | 1.3.3.2  |  主节点|
| idc02-daily-jumpserverhost-123139(10.1.123.139)  | 只需ssh | Centos 7.2(x86-64) |   |  客户节点|

# 安装

> 官方的安装文档十分详细,这里不赘述了。
- 根据官方文档安装即可

```
http://docs.jumpserver.org/zh/docs/step_by_step.html
```




https://github.com/jinyuchen724/linux-base/blob/master/7.openvpn/03-openvpn-install-2.3.10-I601-x86_64.exe

