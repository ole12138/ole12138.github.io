---
title: "Windows内网主机端口映射以及防火墙开放端口"
date: 2021-04-03
draft: false
categories: 
- 运维
tags:
- 内网主机端口映射
- 开放防火墙端口
---

# Windows内网主机端口映射以及防火墙开放端口

公司开发的项目需要对接数据，对方扔过来内网一台主机的向日葵/doDesk远程连接账户和密钥，让我们通过这台主机访问内网上的数据库。

在对方的机子上操作有些不习惯。如果只是远程查看一下数据还好，但是想拉点数据出来测试就更不方便了。

所以就想直接远程连到对方内网上的数据库操作。

这其实需要两个步骤，

1.在对方的这台主机上做内网穿透，使外网可以访问到这台主机。

2.在对方的这台主机上做端口映射，把内网其他主机上的数据库端口映射为这台主机上的一个端口。



内网穿透用花生壳、ngrok、frp等都可以做，我会单独开一篇博文。

这里主要实现一下在windows中添加端口映射，以及为相应的端口打开防火墙。我把这个过程写成一个bat脚本，以管理员权限运行一下即可。

```bat
rem @echo off
rem ******************************************************************************************
rem 场景：
rem 远程/外网可以访问到本机（teamviewer、向日葵、doDesk等远程连接，或者frp、ngrok、花生壳等内网穿透）
rem 远程/外网想要通过本机访问本地网络上的数据库或其他端口
rem *******************************************************************************************
rem 分析：
rem 若外网主机可以访问到本机，想通过本机访问局域网其他主机上的端口，一般有两种方法：
rem 1.在本机上安装数据库客户端，外网主机和本机的客户端交互
rem 2.通过端口映射，将局域网其他主机端口映射到本机端口。
rem *******************************************************************************************
rem 功能：
rem 在本机上添加：到局域网特定主机端口的映射（数据转发/端口转发）
rem *******************************************************************************************

rem 变量设置：
rem 参考：https://blog.csdn.net/chuangxin/article/details/104100725
setlocal
rem 本地监听端口
set LISTENPORT=13306

rem 映射到的主机IP
set CONNECTADDR=192.168.1.21

rem 映射到的主机端口
set CONNECTPORT=3306

rem -----------------------------------------------------------------------------

rem 跳转到此批处理所在目录
cd /d %~dp0

rem -----------------------------------------------------------------------------

rem 使用netsh工具，必须电脑上安装ipv6
rem xp需要这一步骤
netsh interface ipv6 install

rem 设置端口映射
rem 参考：https://segmentfault.com/a/1190000037584182
rem listenaddress：监听地址，0.0.0.0表示监听所有地址，建议用0.0.0.0,也可以填本机的本地ip地址,也可以省略不填
rem listenport：监听端口，指的是本地的端口
rem connectaddress：需要代理的ip
rem connectport：需要转发的端口
netsh interface portproxy add v4tov4 listenaddress=0.0.0.0 listenport=%LISTENPORT% connectaddress=%CONNECTADDR% connectport=%CONNECTPORT%


rem 列出当前所有端口转发规则
netsh interface portproxy show all

rem -----------------------------------------------------------------------------

rem 设置防火墙
rem 允许13306端口的TCP入站（下面的操作等同于在windows defender 防火墙中手动添加入站规则）
netsh advfirewall firewall add rule name="allowPort%LISTENPORT%In" dir=in action=allow protocol=TCP localport=%LISTENPORT%
rem 允许13306端口的TCP出站
netsh advfirewall firewall add rule name="allowPort%LISTENPORT%Out" dir=out action=allow protocol=TCP localport=%LISTENPORT%

rem 列出防火墙转发规则
netsh advfirewall firewall show rule name="allowPort%LISTENPORT%In"
netsh advfirewall firewall show rule name="allowPort%LISTENPORT%Out"

rem -----------------------------------------------------------------------------

pause
```



对应的写了一个删除内网主机端口映射，以及删除防火墙相应端口配置的windows的bat脚本。配一下端口，以管理员权限执行脚本就可以删除端口映射与删除添加的防火墙规则。

```bat
rem @echo off 
rem *******************************************************************************************
rem 功能：
rem 在本机上移除：到局域网特定主机端口的映射（数据转发/端口转发）
rem 需要以管理员身份运行
rem *******************************************************************************************

rem 变量设置：
rem 参考：https://blog.csdn.net/chuangxin/article/details/104100725
setlocal
rem 本地监听端口
set LISTENPORT=13306

rem -----------------------------------------------------------------------------

rem 跳转到此批处理所在目录
cd /d %~dp0

rem -----------------------------------------------------------------------------
rem 移除端口映射
rem 参考：https://segmentfault.com/a/1190000037584182
rem listenaddress：监听地址，0.0.0.0表示监听所有地址，建议用0.0.0.0,也可以填本机的本地ip地址
rem listenport：监听端口，指的是本地的端口
rem connectaddress：需要代理的ip
rem connectport：需要转发的端口
netsh interface portproxy delete v4tov4 listenaddress=0.0.0.0 listenport=%LISTENPORT%
netsh interface portproxy delete v4tov4 listenaddress=* listenport=%LISTENPORT%

rem 或者清空端口映射列表
rem netsh interface portproxy reset

rem 列出当前所有端口转发规则
netsh interface portproxy show all

rem -----------------------------------------------------------------------------

rem 设置防火墙
rem 参考：http://www.voidcn.com/article/p-gtkxnxtm-bqm.html
rem 参考：https://blog.csdn.net/mystudyblog0507/article/details/79617629

rem 删除13306端口的TCP入站（下面的操作等同于在windows defender 防火墙中手动删除入站规则）

netsh advfirewall firewall delete rule name="allowPort%LISTENPORT%In"
rem 删除13306端口的TCP出站
netsh advfirewall firewall delete rule name="allowPort%LISTENPORT%Out"

rem 列出防火墙转发规则
netsh advfirewall firewall show rule name="allowPort%LISTENPORT%In"
netsh advfirewall firewall show rule name="allowPort%LISTENPORT%Out"

rem -----------------------------------------------------------------------------

pause
```

