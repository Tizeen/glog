---
title: 如何指定docker compose中容器的网段
author: likun.gong
type: post
date: 2024-04-19T01:56:19+00:00
url: /2024/04/19/如何指定docker-compose中容器的网段/
categories:
  - how-to
tags:
  - how-to

---
Docker 安装后，看到 `docker0` 网卡使用的是 `172.17.0.1/16` 网段，后续 `Docker Run` 运行起来不指定网络的容器，拿到的网络地址都是属于这个网段。

但是对于利用 Docker Compose 启动得到的容器，它们的 IP 地址并不是从 `172.17.0.1/16` 网段获取的，而是重新创建了一个虚拟网卡，分配了一个新的网给它的容器的使用。

默认 Docker Compose 会从 `172.16.0.0/12` 网络的地址范围 （`172.16.0.0/16` ~ `172.31.255.255/16）` 中挑选一个不重复的进行使用。但是有的时候 Docker Compose 挑选到的网络会和我们网络内其他网段冲突，就会导致容器和那个网络段的服务访问会有问题（路由会有问题），这个时候我们就要手动指定网段，解决网络冲突。

配置如下：

<div class="hcb_wrap">
  <pre class="prism line-numbers lang-json" data-lang="JSON"><code>version: &#39;3&#39;

services:
  frontend:
    image: frontend-app
    networks:
      - custom-network

  backend:
    image: backend-app
    networks:
      - custom-network

networks:
  custom-network:
    driver: bridge
    ipam:
      config:
        - subnet: 192.168.250.0/24</code></pre>
</div>