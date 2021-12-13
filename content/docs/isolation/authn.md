---
title: authCertification
description: "Doks comes with commands for common tasks."
lead: "Doks comes with commands for common tasks."
date: 2020-10-13T15:21:01+02:00
lastmod: 2020-10-13T15:21:01+02:00
draft: false
images: []
menu:
  docs:
    parent: "isolation"
weight: 155
toc: true
---
## 4.1 认证Authentication

背景：电商网站业务团队希望限制访问生产环境（base namespace）下所有服务间的访问必须开启双向认证mTLS，以防御中间人攻击。

服务间通信的mTLS模式默认为PERMISSIVE宽容模式，即服务间的通信既可以使用mTLS加密，也可以使用plaintext明文连接。

此时在tke集群控制台登陆client的istio-proxy容器，使用明文连接对生产环境（base namespace）product服务发起请求：`curl http://product.base.svc.cluster.local:7000/product`，此时明文连接也可正常访问product服务。
<img src="/images/safeLink/4-1-1.png"></src>
<img src="/images/safeLink/4-1-2.png"></src>

限制base namespace下的服务间通信必须采用mTLS模式可以通过PeerAuthentication策略设置mTLS模式为STRICT完成。

<img src="/images/safeLink/4-1-3.png"></src>

配置完成后，在tke集群控制台登陆client的istio-proxy容器，使用明文连接对生产环境（base namespace）product服务发起请求：`curl http://product.base.svc.cluster.local:7000/product`，此时明文连接访问失败。

<img src="/images/safeLink/4-1-4.png"></src>
