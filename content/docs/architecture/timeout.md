---
title: 3.2 配置服务timeout
description: "Doks comes with commands for common tasks."
lead: "Doks comes with commands for common tasks."
date: 2020-10-13T15:21:01+02:00
lastmod: 2020-10-13T15:21:01+02:00
draft: false
images: []
menu:
  docs:
    parent: "architecture"
weight: 145
toc: true
---

<img src="/images/netCommunication/3-2-1.svg"></img>

背景：通过对stock服务配置故障注入，发现由于故障会导致网站用户的请求一直处于等待状态，为优化网站用户的浏览体验，需要为服务配置timeout。

应用以下VS，为order服务配置3秒的超时时间，cart服务不设置超时时间作为参照对比。

```yaml
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: order-vs
  namespace: base
spec:
  hosts:
    - order.base.svc.cluster.local
  http:
    - match:
        - headers:
            cookie:
              exact: vip=false
      route:
        - destination:
            host: order.base.svc.cluster.local
            subset: v1
      timeout: 3000ms
    - match:
        - headers:
            cookie:
              exact: vip=true
      route:
        - destination:
            host: order.base.svc.cluster.local
            subset: v2
      timeout: 3000ms
EOF
```

配置完成后，选择商品加入购物车，此时访问cart服务会有7秒的故障注入访问延时，且没有timeout处理，点击“CHECKOUT”发起结算调用order服务，此时虽然访问order服务也会有7秒的故障注入访问延时，但是有3秒的timeout超时处理，在调用order服务3秒内没有反应会做超时处理。

<img src="/images/netCommunication/3-2-2.png"></img>


超时配置已完成，对于服务的故障注入测试已完成，可删除关联stock服务的VirtualService资源以解除对stock服务配置的故障注入策略。

<img src="/images/netCommunication/3-2-3.png"></img>
