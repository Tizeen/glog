---
title: Tracepath Traceroute MTR
author: likun.gong
type: post
date: 2024-11-07T15:55:28+00:00
url: /2024/11/07/tracepath-traceroute-mtr/
wp_last_modified_info:
  - 2024-11-07 @ 22:35
wplmi_shortcode:
  - '[lmt-post-modified-info]'
categories:
  - what-is
tags:
  - what-is

---
最近工作配合排查了好几次网络问题，于是经常用到了 tracepath、traceroute 和 mtr，三个工具都是跟踪并查找网络线路的，但是又各有差异，这里简单记录一下。

tracepath 利用 **UDP** 协议，通过回来的数据包探测网络线路，并且在执行时不需要 root 权限，在权限不够的情况下，使用 tracepath 是一个很好的选择。

traceroute 则是利用 ICMP 协议来探测网络线路，同时可以通过参数指定使用 TCP、UDP协议来探测，可以看下述例子。<figure class="wp-block-image size-large">

<img loading="lazy" decoding="async" width="1024" height="614" src="https://glog.likungong.com/wp-content/uploads/2024/11/1730992444034-1024x614.jpg" alt="" class="wp-image-275" srcset="https://glog.likungong.com/wp-content/uploads/2024/11/1730992444034-1024x614.jpg 1024w, https://glog.likungong.com/wp-content/uploads/2024/11/1730992444034-300x180.jpg 300w, https://glog.likungong.com/wp-content/uploads/2024/11/1730992444034-768x460.jpg 768w, https://glog.likungong.com/wp-content/uploads/2024/11/1730992444034.jpg 1452w" sizes="auto, (max-width: 1024px) 100vw, 1024px" /> </figure> 

解读：

<ol class="wp-block-list">
  <li>
    * * * 表示没有返回数据包，这在一些商业设备中很常见，它们屏蔽了来自 traceroute 的探测
  </li>
  <li>
    ip 后边的 ms 则是源到这个点的 RTT，每次 traceroute 会发送 3 个数据包来验证
  </li>
</ol>

mtr 是 My Traceroute 缩写，它是 traceroute 和 ping 的结合体，除了能看到网络经过的各个点，它还可以显示到目的地的路线中不断更新的延迟和丢包信息，可以实时看到路径上发生的情况，协助排除网络问题。<figure class="wp-block-gallery has-nested-images columns-default is-cropped wp-block-gallery-5 is-layout-flex wp-block-gallery-is-layout-flex"> <figure data-wp-context="{"imageId":"686e6d3a4a7dd"}" data-wp-interactive="core/image" class="wp-block-image size-full is-style-default wp-lightbox-container">

<img loading="lazy" decoding="async" width="2560" height="716" data-wp-class--hide="state.isContentHidden" data-wp-class--show="state.isContentVisible" data-wp-init="callbacks.setButtonStyles" data-wp-on-async--click="actions.showLightbox" data-wp-on-async--load="callbacks.setButtonStyles" data-wp-on-async-window--resize="callbacks.setButtonStyles" data-id="279" src="https://glog.likungong.com/wp-content/uploads/2024/11/1730992747686-1-scaled.jpg" alt="" class="wp-image-279" srcset="https://glog.likungong.com/wp-content/uploads/2024/11/1730992747686-1-scaled.jpg 2560w, https://glog.likungong.com/wp-content/uploads/2024/11/1730992747686-1-300x84.jpg 300w, https://glog.likungong.com/wp-content/uploads/2024/11/1730992747686-1-1024x287.jpg 1024w, https://glog.likungong.com/wp-content/uploads/2024/11/1730992747686-1-768x215.jpg 768w, https://glog.likungong.com/wp-content/uploads/2024/11/1730992747686-1-1536x430.jpg 1536w, https://glog.likungong.com/wp-content/uploads/2024/11/1730992747686-1-2048x573.jpg 2048w" sizes="auto, (max-width: 2560px) 100vw, 2560px" /> 
			<button
			class="lightbox-trigger"
			type="button"
			aria-haspopup="dialog"
			aria-label="放大"
			data-wp-init="callbacks.initTriggerButton"
			data-wp-on-async--click="actions.showLightbox"
			data-wp-style--right="state.imageButtonRight"
			data-wp-style--top="state.imageButtonTop"
		> <svg xmlns="http://www.w3.org/2000/svg" width="12" height="12" fill="none" viewBox="0 0 12 12"> <path fill="#fff" d="M2 0a2 2 0 0 0-2 2v2h1.5V2a.5.5 0 0 1 .5-.5h2V0H2Zm2 10.5H2a.5.5 0 0 1-.5-.5V8H0v2a2 2 0 0 0 2 2h2v-1.5ZM8 12v-1.5h2a.5.5 0 0 0 .5-.5V8H12v2a2 2 0 0 1-2 2H8Zm2-12a2 2 0 0 1 2 2v2h-1.5V2a.5.5 0 0 0-.5-.5H8V0h2Z" /> </svg> </button></figure> </figure> 

解读：

<ol class="wp-block-list">
  <li>
    Loss% 可以实时看到我们到每一跳的丢包率
  </li>
  <li>
    Snt 是发送的数据包总数，Avg 表示平均 RTT，Last 是最后一次 RTT，Best 是最佳 RTT，Wrst 则是最差的
  </li>
  <li>
    中间的点出现了丢包，但是到最终的目的地丢包率是 0，说明我们访问是没有丢包的，这里有可能是我们发送的 ICMP 包超过了网络设备的限制，从而被丢包
  </li>
</ol>

总结：

mtr 默认使用 ICMP 协议来探测线路，也可以通过参数指定使用 TCP、UDP 协议来探测，对比 traceroute 和 tracepath 的功能更强大，并且展示的信息更多，更有利于发现网络问题，排查时可以优先考虑使用 mtr。