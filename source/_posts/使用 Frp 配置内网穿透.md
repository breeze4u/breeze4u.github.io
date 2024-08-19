---
title: 使用 Frp 配置内网穿透
tags: 
  - Frp
  - 内网穿透
date: 2023-12-27
categories: 
  - 教程
---

## 版本信息

| 软件       | 版本                     |
| :-------- | :---------------------- |
| Frp      | frp_0.53.2_linux_amd64 |
| 服务端Linux | CentOS 8.2 |
| 客户端Linux | CentOS 6.8 |

## 一、安装 Frp

- [下载地址](https://github.com/fatedier/frp/releases)
- [参考配置教程](https://blog.csdn.net/weixin_43804047/article/details/135174832)

下载完毕后上传源码包到云服务器和实验室服务器
## 二、服务器端配置
### 1. 编辑`frps.toml`

 ```
 [common]
# frp监听的端口，默认是7000，可修改
bind_port = 7000

# frp面板端口、用户名和密码，请改成更复杂的。
dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = admin123456

# 授权码，请改成更复杂的，这个token之后在客户端会用到
token = e10adc3949ba59abbe56e057f20f883e

# 设置HTTP及https协议下代理端口(非重要)
# vhost_http_port = 7080
# vhost_https_port = 7081

# 去除TCP速度限制
tcp_mux = false

# enable_prometheus = true

# frp日志配置
log_file = /home/$USER/frp/frps.log
log_level = info
log_max_days = 3
```
### 2. 运行服务
```
./frps -c frps.toml
```
###  3. 服务端自启动配置

将 `frps, frps.toml` 文件放到系统目录下

```
mv frps /usr/bin/ # frps 可执行文件
mkdir /etc/frp
mv frps.toml /etc/frp/ # 配置文件
```

新建文件`frps.service` 内容如下

```
[Unit]
Description=frps service
After=network.target syslog.target
Wants=network.target

[Service]
Type=simple
ExecStart=/usr/bin/frps -c /etc/frp/frps.toml

[Install]
WantedBy=multi-user.target
```

将文件复制到自启动服务项

```
cp frps.service /usr/lib/systemd/system/
```

设置自启动，启动服务

```
systemctl enable frps  # 允许自启动
# 执行成功会提示“Created symlink /etc/systemd/system/multi-user.target.wants/frps.service → /usr/lib/systemd/system/frps.service.”
systemctl start frps  # 启动客户端服务
```

最后将云服务器中所有提及的端口打开
## 三、客户端配置
###  1. 编辑 `frpc.toml`

```
[common]
# 服务端公网ip、监听端口bind_port
server_addr = x.x.x.x
server_port = 7000
# 服务端的授权码
token = e10adc3949ba59abbe56e057f20f883e  
# 去掉速度限制
tcp_mux = false

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
remote_port = 2288

# [ssh] 为服务名称，下方解释：访问frp服务端的2288端口时，等同于通过中转服务器访问127.0.0.1的22端口。
# type 为连接的类型，此处为tcp
# local_ip 为中转客户端实际访问的IP
# local_port 为目标端口
# remote_port 为远程端口，记得服务端的防火墙打开这个端口
```

需要其他端口映射则添加类似配置，如

```
[hadoop]
type = tcp
local_port = 50070
remote_port = 50070
```
### 2. 客户端自启动配置

[centos6自启动配置](https://blog.csdn.net/freshboya/article/details/86663907)
## 四、dashboard web

`服务器端IP:7500`
## 五、连接实验室服务器

```
ssh username@服务器端ip -p 2288
```