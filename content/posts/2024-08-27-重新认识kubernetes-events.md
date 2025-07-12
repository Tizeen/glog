---
title: 重新认识Kubernetes events
author: likun.gong
type: post
date: 2024-08-27T02:56:54+00:00
url: /2024/08/27/重新认识kubernetes-events/
wp_last_modified_info:
  - 2024-08-20 @ 18:14
wplmi_shortcode:
  - '[lmt-post-modified-info]'
categories:
  - what-is
tags:
  - what-is

---
 

## event是什么 {.wp-block-heading}

Kubernetes 中的组件状态发生变化，它就会产生一个事件，也就是 event 。在 event 中提供了集群的状态和健康信息，例如：

<ol class="wp-block-list">
  <li>
    容器创建失败
  </li>
  <li>
    Pod 在不停的被调度
  </li>
</ol>

事件存储在 Etcd 中，**默认只存储最近 1 个小时**。如果需要持久化事件，可以通过 [kubernetes-event-exporter][1] 持久化到 ElasticSearch 中。

## event的数据结构 {.wp-block-heading}

平时使用比较多的查看 event 的方法

<div class="hcb_wrap">
  <pre class="prism line-numbers lang-bash" data-lang="Bash"><code>kubectl describe node &lt;node_name&gt;

kubectl describe pod &lt;pod_name&gt;</code></pre>
</div><figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="1024" height="204" src="https://glog.likungong.com/wp-content/uploads/2024/08/图片-1-1024x204.png" alt="" class="wp-image-226" srcset="https://glog.likungong.com/wp-content/uploads/2024/08/图片-1-1024x204.png 1024w, https://glog.likungong.com/wp-content/uploads/2024/08/图片-1-300x60.png 300w, https://glog.likungong.com/wp-content/uploads/2024/08/图片-1-768x153.png 768w, https://glog.likungong.com/wp-content/uploads/2024/08/图片-1-1536x306.png 1536w, https://glog.likungong.com/wp-content/uploads/2024/08/图片-1-2048x408.png 2048w" sizes="auto, (max-width: 1024px) 100vw, 1024px" /> </figure> 

这里显示了常用的字段，如果需要查看完整的字段，可以

<div class="hcb_wrap">
  <pre class="prism line-numbers lang-bash" data-lang="Bash"><code>kubectl get events -ojson</code></pre>
</div><figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="925" height="1024" src="https://glog.likungong.com/wp-content/uploads/2024/08/图片-2-925x1024.png" alt="" class="wp-image-228" srcset="https://glog.likungong.com/wp-content/uploads/2024/08/图片-2-925x1024.png 925w, https://glog.likungong.com/wp-content/uploads/2024/08/图片-2-271x300.png 271w, https://glog.likungong.com/wp-content/uploads/2024/08/图片-2-768x850.png 768w, https://glog.likungong.com/wp-content/uploads/2024/08/图片-2.png 1234w" sizes="auto, (max-width: 925px) 100vw, 925px" /> </figure> 

过滤对应资源的时间，并且定制输出字段，例如可以查看 Pod 从创建到启动的各个时间，分析 Pod 启动的各个过程花费的时间，是否有优化的地方。例如可以在事件中查到容器启动时拉取镜像花费了多久时间。

<div class="hcb_wrap">
  <pre class="prism line-numbers lang-bash" data-lang="Bash"><code>kubectl -n cloud  get events \
    -o custom-columns=Time:.lastTimestamp,From:.source.component,Type:.type,Reason:.reason,Message:.message \
    --field-selector involvedObject.name=temporary-pod,involvedObject.kind=Pod</code></pre>
</div>

## 应该关注哪些event {.wp-block-heading}

event 中包含了很多重要的信息，我们需要关注起来，并配置对应的告警。

<ol class="wp-block-list">
  <li>
    Failed events: 容器创建失败
  </li>
  <li>
    Eviction events: Pod 因为资源不够被驱逐
  </li>
  <li>
    Volume events: 卷挂载失败
  </li>
  <li>
    Scheduling events: 调度失败
  </li>
  <li>
    Unready node events: worker node 未就绪
  </li>
</ol>

 [1]: https://github.com/resmoio/kubernetes-event-exporter