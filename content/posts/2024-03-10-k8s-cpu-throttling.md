---
title: K8S CPU Throttling
author: likun.gong
type: post
date: 2024-03-10T03:26:38+00:00
url: /2024/03/10/k8s-cpu-throttling/
categories:
  - what-is
tags:
  - what-is

---
### 什么是CPU Throttling {.wp-block-heading}

在 K8S 中，内存有很明确的大小指标来衡量，一旦超过设定的 Limit，就会发生 OOM ，被系统 Kill 掉，这就是 PodOOM 事件。  
而 CPU 的衡量指标则有点模糊，是根据占据 CPU 的使用时间来决定的。CPU 使用超限了 Pod 也不会被杀掉，而是 CPU 使用会被限流。

CPU Throttling 直译来说就是 CPU 使用被节流了，被限制了 CPU 的使用时间。

在 K8S 中，通过调整容器中进程的 CPU 优先级(nice值，值越小，优先级越高)来达到限流的效果。

### CPU Throttling影响什么 {.wp-block-heading}

影响：

<ol class="wp-block-list">
  <li>
    CPU 处理变慢了，导致服务性能下降，response_time 响应时间增加
  </li>
</ol>

### 如何监控CPU Throttling {.wp-block-heading}

cadvisor 提供了相应指标：

<ol class="wp-block-list">
  <li>
    container_cpu_cfs_throttled_periods_total
  </li>
  <li>
    container_cpu_cfs_periods_total
  </li>
</ol>

一个 PromQL例子

<div class="hcb_wrap">
  <pre class="prism line-numbers lang-sql" data-lang="SQL"><code>sum(increase(container_cpu_cfs_throttled_periods_total{container!="filebeat",cluster!~"kube.*-(dev|test|pre|t|t2|t3|s|u|d)$"}[5m])) by (container, pod, namespace, cluster)
  /
sum(increase(container_cpu_cfs_periods_total{cluster!~"kube.*-(dev|test|pre|t|t2|t3|s|u|d)$"}[5m])) by (container, pod, namespace, cluster)
  &gt; ( 10 / 100 )</code></pre>
</div>

### 如何解决CPU Throttling {.wp-block-heading}

解决：

<ol class="wp-block-list">
  <li>
    横向增加 Pod 数量，降低每个 Pod 的访问量，以降低 CPU 使用时间
  </li>
  <li>
    纵向增加 CPU 资源限制（调大 Limit 值）
  </li>
</ol>