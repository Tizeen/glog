---
date: '2025-12-15T21:05:57+08:00'
draft: false
title: 'A K8S Network Problem'
author: likun.gong
categories:
  - Log
tags:
  - log
---

In the past few weeks, we met a problem in Kubernetes.

The pods in the k8s access the ingress domain address, sometimes the request will timeout, and it's not occur 100%.

I known we shoud use the k8s service address to intead the ingress address, but when I face the problem, we have no time to change the address and retest the service, so we need to find the reason and solve it.

We use ingress nginx controller to expose our api, I check the network chain, everything seems to be ok, the alb target health check was ok, the nginx controller state also ok, and when we access the ingress api address from cluster outside, it always success.

So I guess the problem was in the kubernetes, I found our nginx controller was expose use nodeport, the alb target has all worker nodes, this is the default service type when we use helm deploy ingress nginx.

When I check the [documents](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.5/guide/service/annotations/#nlb-target-type), I found the aws load balancer policy can be use ip mode, when we use the ip mode, the alb target would be use nginx controller pod ip, it can direct connect to nginx controller, we don't need to use node ip and use **kubeproxy** to forward the request to nginx controller.

I redeploy the nginx controller with ip mode, the problem was solve.

But why are we use the instance mode can be cause the problem? I need to know the reason.