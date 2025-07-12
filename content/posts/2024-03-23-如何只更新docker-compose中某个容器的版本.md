---
title: 如何只更新Docker Compose中某个容器的版本
author: likun.gong
type: post
date: 2024-03-23T15:44:45+00:00
url: /2024/03/23/如何只更新docker-compose中某个容器的版本/
categories:
  - how-to
tags:
  - how-to

---
遇到了很多次这个场景，发布版本的时候，只需要更新 Docker Compose 中某个容器的镜像版本并重启，不需要重启其他依赖的容器。

其实 Docker 已经为我们考虑到了这个问题了，并提供了相应的参数。

方法一：

<ol class="wp-block-list">
  <li>
    更新 <code>docker-compose.yml </code>中的镜像 ID
  </li>
  <li>
    重新执行 <code>docker compose up -d </code>，会自动判断那个 service 发生了变化，并重启变化的容器
  </li>
</ol>

方法二：

<ol class="wp-block-list">
  <li>
    同样是更新 <code>docker-compose.yml</code> 中的镜像 ID
  </li>
  <li>
    执行 up 时，加上 <code>--no-deps</code> 的参数，也能做到不重启其他依赖容器。这种做法的好处是可以明确指定那个容器，如果有多个容器需要重启，能做到有顺序的重启
  </li>
</ol>

<div class="hcb_wrap">
  <pre class="prism line-numbers lang-bash" data-lang="Bash"><code>docker compose up -d --no-deps container_name</code></pre>
</div>