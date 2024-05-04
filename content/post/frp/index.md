---
title: frp 内网穿透工具使用及其容器化

# Summary for listings and search engines
summary: frp 内网穿透

# Link this post with a project
projects: []

# Date published
date: '2024-05-04T00:00:00Z'

# Date updated
lastmod: '2024-05-04T00:00:00Z'

# Is this an unpublished draft?
draft: false

# Show this page in the Featured widget?
featured: false

# Featured image
# Place an image named `featured.jpg/png` in this page's folder and customize its options here.
# image:
#   caption: 'Image credit: [**Unsplash**](https://unsplash.com/photos/CpkOjOcXdUY)'
#   focal_point: ''
#   placement: 2
#   preview_only: false

authors:
  - Knight1357

tags:
  - frp
  - 内网穿透
  - Docker
  - 容器

categories:
  - 内网穿透
  - 教程
  - Docker
---

# frp 是什么

## 概述

frp 是一个开源、简洁易用、高性能的内网穿透和反向代理软件，支持 tcp, udp, http, https等协议。

frp 项目官网是:  https://github.com/fatedier/frp

## 工作原理
服务端运行，监听一个主端口，等待客户端的连接；- 客户端连接到服务端的主端口，同时告诉服务端要监听的端口和转发类型；- 服务端fork新的进程监听客户端指定的端口；- 外网用户连接到客户端指定的端口，服务端通过和客户端的连接将数据转发到客户端；- 客户端进程再将数据转发到本地服务，从而实现内网对外暴露服务的能力。

# 准备工作

想要配置 frp 穿透，首先必须先要有一台具有公网IP (即：可以外网访问)的服务器。如果没有，接下来的教程就不用看了。配置教程主要分为两个部分:
- 一是服务器端(外网服务器)的配置
- 二是客户端(内网服务器)配置。

## 下载 frp 源码
```sh
wget https://github.com/fatedier/frp/releases/download/v0.33.0/frp_0.33.0_linux_amd64.tar.gz 
```

## 服务器端配置

1. 解压 frp 压缩包
```sh
tar -zxvf frp_0.33.0_linux_amd64.tar.gz
```
2. 改名
```sh
mv frp_0.33.0_linux_amd64.tar.gz frp
```
3. 进入目录
```sh
cd frp
```

其中 frps.XXX 为服务器端相关文件

4. 打开 frps.ini 服务器端配置文件
```sh
vim frps.ini
```

将文件修改内容如下
```sh
[common]
# frp监听的端口，默认是7000，可以改成其他的
bind_port = 7000
# 授权码，请改成更复杂的，这个token之后在客户端会用到
token = e10adc3949ba59abbe56e057f20f883e
# 开启HTTP
#vhost_http_port = 8088
# 去除TCP速度限制
tcp_mux = false

# frp管理后台端口，请按自己需求更改
dashboard_port = 7500
# frp管理后台用户名和密码，请改成自己的
dashboard_user = admin
dashboard_pwd = password
enable_prometheus = true

# frp日志配置
log_file = /home/frp/frp/frps.log
log_level = info
log_max_days = 3
```

5. 启动服务器端 frpc
```sh
./frps -c frps.ini
```
后台运行指令
```sh
screen
./frps -c frps.ini
```

6. 防火墙开放端口
开启云服务器 7000 和 7500 两个端口，之前`frps.ini` 中配置的端口

7. 验证服务端是否启动成功
访问 `云服务器IP:7500` 输入 `用户名` 和 `密码` ，看是否能成功登录管理页面
## 客户端配置

1. 解压 frp 压缩包
```sh
tar -zxvf frp_0.33.0_linux_amd64.tar.gz
```

2. 改名
```sh
mv frp_0.33.0_linux_amd64.tar.gz frp
```

3. 进入目录
```sh
cd frp
```

其中 frpcXXX 为客户端相关文件

4. 打开 frpc.ini 客户端配置文件
```sh
vim frps.ini
```

将文件修改内容如下：
```sh
# 客户端配置
[common]
server_addr = 服务器ip
server_port = 7000 
# 与frps.ini的bind_port一致
token = e10adc3949ba59abbe56e057f20f883e  
# 与frps.ini的token一致

# 配置ssh服务
[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
# 这个自定义，之后再ssh连接的时候要用
remote_port = 6000  

# 配置http服务，可用于小程序开发、远程调试等，如果没有可以不写下面的
[web]
type = http
local_ip = 127.0.0.1
local_port = 8080
# web域名
subdomain = test.xxxx.pw  
remote_port = 自定义的远程服务器端口，例如8080
```

5. 开启防火墙

6. 启动客户端

```sh
./frpc -c frpc.ini
```
后台运行客户端
```sh
screen
./frpc -c frpc.ini
```

## 测试内网穿透是否成功
找另外一个电脑，在终端执行
```sh
ssh 用户名@云服务器IP -p 端口号
```

# 容器化 frp

在了解 frp 使用和原理后，我们可以对 frp 进行容器化，方便后续部署frp

## 服务端容器化
1. 新建项目文件夹
2. 下载 frp 压缩包
3. 编写 frps 的 Dockerfile 文件
```Dockerfile
FROM alpine:latest
ADD ./frps.tar.gz /usr/local/
CMD ["/usr/local/frps/frps","-c","/usr/local/frps/frps.ini"]
```
4. 构建 Docker 镜像
5. 运行 Docker 镜像
```sh
docker run -it -v /usr/local/frps/frps.ini --restart=always --network=host  --name frps frps:latest
```
6. 修改 `/usr/local/frps/frps.ini` 配置文件
```sh
[common]
# frp监听的端口，默认是7000，可以改成其他的
bind_port = 7000
# 授权码，请改成更复杂的，这个token之后在客户端会用到
token = e10adc3949ba59abbe56e057f20f883e
# 开启HTTP
#vhost_http_port = 8088
# 去除TCP速度限制
tcp_mux = false

# frp管理后台端口，请按自己需求更改
dashboard_port = 7500
# frp管理后台用户名和密码，请改成自己的
dashboard_user = admin
dashboard_pwd = password
enable_prometheus = true

# frp日志配置
log_file = /usr/local/frps.log
log_level = info
log_max_days = 3
```


## 客户端容器化
1. 新建项目文件夹
2. 下载 frp 压缩包
3. 编写 frpc 的 Dockerfile 文件
```Dockerfile
FROM alpine:latest
ADD ./frpc.tar.gz /usr/local/
CMD ["/usr/local/frpc/frpc","-c","/usr/local/frpc/frpc.ini"]
```
4. 构建 Docker 镜像
5. 运行 Docker 镜像
```sh
docker run -it -v /usr/local/frpc/frpc.ini --restart=always --network=host  --name frpc frpc:latest
```
6. 修改 `/usr/local/frpc/frpc.ini` 配置文件
```sh
# 客户端配置
[common]
server_addr = 127.0.0.1
server_port = 7000 
# 与frps.ini的bind_port一致
token = e10adc3949ba59abbe56e057f20f883e  
# 与frps.ini的token一致

# 配置ssh服务
[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 22
# 这个自定义，之后再ssh连接的时候要用
remote_port = 6000  
```

## 测试
找另外一个电脑，在终端执行
```sh
ssh 用户名@云服务器IP -p 端口号
```

