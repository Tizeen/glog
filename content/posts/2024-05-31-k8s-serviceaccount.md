---
title: K8S ServiceAccount
author: likun.gong
type: post
date: 2024-05-31T15:39:46+00:00
url: /2024/05/31/k8s-serviceaccount/
categories:
  - how-to
  - what-is

---
近期又频繁接触 K8S，发现以前自己学的很多都已经发生变化，觉得有必要记录一下。

## ServiceAccount是什么 {.wp-block-heading}

服务账号，顾名思义是用来做身份标识的，它与平时接触的账号又有一些不一样。在 K8S 中，Pod、系统组件以及集群外的实体都可以使用 **ServiceAccount 凭证** 标识自己为对应的 ServiceAccount，以此来获取访问 K8S 其他资源的权限。

可以通过以下命令查看 ServiceAccount 

<div class="hcb_wrap">
  <pre class="prism line-numbers lang-bash" data-lang="Bash"><code>kubectl get sa -n default</code></pre>
</div>

## ServiceAccount使用场景 {.wp-block-heading}

目前接触到的使用场景有这些：

<ol class="wp-block-list">
  <li>
    利用 ServiceAccount 的 secret，生成一个 kubeconfig 作为 CD 流水线中操作 K8S 的凭证
  </li>
  <li>
    在 Pod 中利用 ServiceAccount 和 K8S 自身通信，调用 API 启动 JOB 类型的 Pod，或者拿到 configmap 中的配置
  </li>
  <li>
    将 AWS 的 role 绑定到对应的 ServiceAccount，使 Pod 拥有对应 role 的权限，比如直接访问 S3 的权限
  </li>
</ol>

### 使用ServiceAccount token生成kubeconfig {.wp-block-heading}

通过以下方法创建一个长期有效的 Token（**长期有效的 token 可能造成安全问题**，谨慎使用），主要是  
`type: kubernetes.io/service-account-token`

<div class="hcb_wrap">
  <pre class="prism line-numbers lang-bash" data-lang="Bash"><code>kubectl apply -f - &lt;&lt;EOF
apiVersion: v1
kind: Secret
metadata:
  name: sa-secret
  annotations:
    kubernetes.io/service-account.name: sa
type: kubernetes.io/service-account-token
EOF</code></pre>
</div>

查看 token

<div class="hcb_wrap">
  <pre class="prism line-numbers lang-plain"><code>kubectl get sa-secret -oyaml</code></pre>
</div>

将 secret 的 yaml 打印出来，会发现有一个 `token data`，这个 token 就是标识对应的 ServiceAccount，**ServiceAccount 的权限，需要通过 RBAC（role、clusterrole） 来控制。**

<div class="hcb_wrap">
  <pre class="prism line-numbers lang-plain"><code>apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: admin
  namespace: sa
rules:
- apiGroups:
  - &#39;*&#39;
  resources:
  - &#39;*&#39;
  verbs:
  - &#39;*&#39;</code></pre>
</div>

注意 secret 中的 token 是使用 base64 编码过的，在 kubeconfig 中使用前要先解码。

我们需要从已有的 kubeconfig 中知道集群的 ApiServer 地址和 CA 信息，再配置对应的 context 上下文，将集群和 ServiceAccount 绑定起来，最终的 kubeconfig 样子大概如下：

<div class="hcb_wrap">
  <pre class="prism line-numbers lang-json" data-lang="JSON"><code>apiVersion: v1
kind: Config
current-context: sa-user-cluster-name
contexts:
- context:
    cluster: cluster-name
    user: sa-user
  name: sa-user-cluster-name
clusters:
- cluster:
    certificate-authority-data: cluster-ca
    server: api-server-url
  name: cluster-name
users:
- user:
    token: sa-token
  name: sa-user</code></pre>
</div>

最后可以使用这个 kubeconfig 连接集群

<div class="hcb_wrap">
  <pre class="prism line-numbers lang-plain"><code>kubectl --kubeconfig get pods </code></pre>
</div>

### Pod中引入ServiceAccount {.wp-block-heading}

在 Pod 中引入 ServiceAccount，代码中即可使用 K8S API 获取 ServiceAccount 的token，后续使用次 token 作为与 K8S API 通信的凭证，来读取 configmap 、secret 或者其他操作。

<div class="hcb_wrap">
  <pre class="prism line-numbers lang-json" data-lang="JSON"><code>apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  serviceAccountName: sa-user</code></pre>
</div>

对比通过 volume 将 configmap 挂载到 pod 中的方式，通过 ServiceAccount 读取主要有以下优势：

<ol class="wp-block-list">
  <li>
    实时更新（但还需要程序配合热更新），volume 挂载的方式需要重启才能更新
  </li>
  <li>
    更安全。只有授权的 pod 才能访问对应资源，而 volume 的方式，任何能访问 Pod 的进程都能访问到挂载目录中的数据
  </li>
</ol>

### 将AWS role绑定给ServiceAccount {.wp-block-heading}

AWS 的配置项：

<ol class="wp-block-list">
  <li>
    EKS 需要打开 OIDC
  </li>
  <li>
    创建一个 Policy ，配置对应需要的权限，例如 S3 的 GetObject 权限
  </li>
  <li>
    创建一个 Role，Trusted entity type 需要使用 Web identity，并且选择对应集群的 OIDC 地址，在 <strong>Trusted </strong>relationships 中需要配置对应的关系<br /><img loading="lazy" decoding="async" width="1000" height="341" class="wp-image-155" style="width: 1000px;" src="https://glog.likungong.com/wp-content/uploads/2024/05/1717169262112-scaled.jpg" alt="" srcset="https://glog.likungong.com/wp-content/uploads/2024/05/1717169262112-scaled.jpg 2560w, https://glog.likungong.com/wp-content/uploads/2024/05/1717169262112-300x102.jpg 300w, https://glog.likungong.com/wp-content/uploads/2024/05/1717169262112-1024x349.jpg 1024w, https://glog.likungong.com/wp-content/uploads/2024/05/1717169262112-768x262.jpg 768w, https://glog.likungong.com/wp-content/uploads/2024/05/1717169262112-1536x523.jpg 1536w, https://glog.likungong.com/wp-content/uploads/2024/05/1717169262112-2048x698.jpg 2048w" sizes="auto, (max-width: 1000px) 100vw, 1000px" />
  </li>
  <li>
    在 ServiceAccount 中添加注解<br /><code>kubectl annotate serviceaccount -n cloud process-account eks.amazonaws.com/role-arn=arn:aws:iam::xxx:role/xxx-role</code>
  </li>
  <li>
    在 Pod 中引入 ServiceAccount，就会使得 Pod 拥有 Role 中定义的 AWS 权限，可以直接访问 S3 了
  </li>
</ol>