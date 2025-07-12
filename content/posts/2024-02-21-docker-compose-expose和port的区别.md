---
title: Docker compose中expose和port的区别
author: likun.gong
type: post
date: 2024-02-21T15:48:19+00:00
url: /2024/02/21/docker-compose-expose和port的区别/
categories:
  - what-is
tags:
  - what-is

---
<ul class="wp-block-list">
  <li>
    expose 指定的端口，是提供给 compose 中其他 service 用的，不会映射到宿主机上。
  </li>
  <li>
    port 指定的端口，则会映射到宿主机上。
  </li>
</ul>

#### expose {.wp-block-heading}

<div class="hcb_wrap">
  <pre class="prism line-numbers lang-js" data-lang="JavaScript"><code>services:
  myapp1:
    .
    expose:
      - "3000"
      - "8000"</code></pre>
</div>

#### port {.wp-block-heading}

<div class="hcb_wrap">
  <pre class="prism line-numbers lang-js" data-lang="JavaScript"><code>services:
  myapp1:
    ...
    ports:
    - "3000"                 # 容器中的3000端口映射到主机一个随机端口
    - "3001-3005"            # 容器中的3001-3005端口映射到主机一个随机端口段
    - "8000:8000"            # 容器中的8000端口映射到主机的8000端口
    - "9090-9091:8080-8081"  # 容器中的8080-8081映射到主机的9090-9091
    - "127.0.0.1:8002:8002"  # 容器中的8002端口映射主机127.0.0.1地址的8002
    - "6060:6060/udp"        # 限制容器中6060端口的udp协议到主机的6060端口  </code></pre>
</div>