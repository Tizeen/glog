---
date: '2026-02-06T10:43:54+08:00'
draft: false
title: 'A K8S CoreDNS Problem'
author: likun.gong
categories:
  - Log
tags:
  - log
---

记录一下之前遇到的一个 CoreDNS 问题。

## 问题

服务在运行过程中，**偶尔**出现超时的情况，影响到服务的稳定性。

通过服务日志可以看到解析域名超时了，而这个解析是由 CoreDNS 处理的。

## 排查

1. 通过 CoreDNS 监控，我们并没有看到 serverfail 的请求，与此同时 CoreDNS 也没有资源压力，CPU、Mem 的使用情况也很稳定，也就是 CoreDNS 的处理没有问题
2. 查看 CoreDNS 的日志，也没有看到相应的错误日志。而我们从服务日志中得到的信息是 i/o timeout，现在不是处理超时或者处理失败导致超时，那么就有可能是网络超时了
3. 通过 Gemini 配合排查得知，在节点数量增加或者 Pod 数量较多的情况下，由于 DNS 请求使用 UDP 的协议，初步判断是流量在节点转发 UDP 请求的过程中，有的请求被丢弃了。而我们 K8S 节点是动态弹性伸缩的，出现问题的时间点确实也是在节点数量高峰期时最容易出现。
4. 通过 K8S [官方文档](https://kubernetes.io/docs/tasks/administer-cluster/nodelocaldns/)，我们也发现，官方提供一个 CoreDNS 的优化方案，那就是通过 NodeLocalDNS 来做一层缓存，在所有节点上跑一个 NodeLocalDNS，缓存 DNS 解析记录，直接返回解析结果，避免所有的 DNS 请求都走到 CoreDNS 去处理

## 处理

既然知道了问题所在，那就是部署 NodeLocalDNS 了，通过指引部署即可，部署之后可以 Grafana 观察解析指标，以及观察服务访问日志，发现再也没有出现解析超时的问题。