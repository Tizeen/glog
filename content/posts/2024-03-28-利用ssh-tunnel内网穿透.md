---
title: 利用SSH Tunnel内网穿透
author: likun.gong
type: post
date: 2024-03-28T06:14:37+00:00
url: /2024/03/28/利用ssh-tunnel内网穿透/
categories:
  - how-to
tags:
  - how-to

---
SSH 除了平时连接远程服务器外，还有不少其他的功能，刚好最近有这个需求，利用了 SSH Tunnel 实现内网穿透，这里简单记录一下。

### 前提 {.wp-block-heading}

<ol class="wp-block-list">
  <li>
    需要访问的内网服务器 A
  </li>
  <li>
    有公网地址外网服务器 B
  </li>
</ol>

利用公网服务器 B 来暴露服务器 A 中的端口，需要 A 主动通过 SSH 连接服务器 B

### 配置过程 {.wp-block-heading}

SSH 原生支持远程端口转发，在内网服务器 A 上执行以下命令：

<div class="hcb_wrap">
  <pre class="prism line-numbers lang-bash" data-lang="Bash"><code># 内网服务器 A 上执行
# ssh -fN -R 23456:127.0.0.1:22 -i xxx.key B_user@B_public_ip</code></pre>
</div>

命令解释：

<ol class="wp-block-list">
  <li>
    23456 端口，是监听在公网服务器 B 上的
  </li>
  <li>
    127.0.0.1:22 表示将内网服务器 A 的 ssh 端口映射到公网服务器 23456 端口
  </li>
  <li>
    -f 表示后台运行
  </li>
  <li>
    -N 不执行远程命令。端口转发时使用
  </li>
</ol>

这个时候在第三台服务器 C 上，就可以通过公网服务器 B 的公网地址 + 23456 端口，访问到内网服务器 A 了

<div class="hcb_wrap">
  <pre class="prism line-numbers lang-bash" data-lang="Bash"><code># 三方服务器 C 上执行
$ ssh -p 23456 A_user@B_public_ip</code></pre>
</div>

### 进一步优化 {.wp-block-heading}

SSH 的连接有时会因为网络不稳定而断开，我们可以使用 [autossh][1] 这个工具来帮我们维持 SSH 连接，在断开时自动重连。

autossh 使用方式：

<div class="hcb_wrap">
  <pre class="prism line-numbers lang-bash" data-lang="Bash"><code># autossh -M 10086 -fCNR 23456:127.0.0.1:22 -i xxx.key B_user@B_public_ip</code></pre>
</div>

参数解释：

<ol class="wp-block-list">
  <li>
    -M 指定的端口，也是开放在公网服务器 B 上，用来监听连接的状态，在断开时自动重连的
  </li>
  <li>
    其他都是 SSH 自身的参数
  </li>
</ol>

### 其他SSH Tunnel知识 {.wp-block-heading}

上述内容是将本地内网端口暴露到公网服务器上，叫做 **SSH远程端口转发** 

SSH 本地端口转发，大概命令如下：

<div class="hcb_wrap">
  <pre class="prism line-numbers lang-bash" data-lang="Bash"><code>ssh -L 9000:remotehost:80 user@host</code></pre>
</div>

总结一下：

<ol class="wp-block-list">
  <li>
    SSH 本地端口转发，将远程主机的服务端口暴露到本地主机，用户可以在本地主机访问监听端口，就像访问远程服务一样。
  </li>
  <li>
    SSH 远程端口转发，将本机的端口通过 SSH 连接暴露到远程主机，其他机器可以通过远程主机访问本地的服务。
  </li>
  <li>
    SSH 还可以利用 SSH 隧道，在本地启动一个 socks 代理，使用 SSH 通道和远端服务器通信。
  </li>
</ol>

SSH 能支持端口转发，利用的是 **TCP Forwarding** ，TCP Forwarding 可以将一个 TCP 连接上的数据转发到另一个 TCP 连接上。

 [1]: https://github.com/Autossh/autossh