---
title: 3.1 故障注入测试服务弹性
description: ""
lead: ""
date: 2020-10-13T15:21:01+02:00
lastmod: 2020-10-13T15:21:01+02:00
draft: false
images: []
menu:
  docs:
    parent: "architecture"
weight: 144
toc: true
---

<img src="/images/netCommunication/3-1-1.svg"></img>
背景：电商网站业务团队需要模拟访问库存服务存在延迟故障时网站系统的行为，以测试服务弹性，网站用户的优化访问体验。

通过配置绑定stock服务的VritualService，设置访问stock服务的故障注入策略：100%的请求会有7秒的固定延迟。
```yaml
kubectl apply -f - <<EOF
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: stock-vs
  namespace: base
spec:
  hosts:
    - stock.base.svc.cluster.local
  http:
    - route:
        - destination:
            host: stock.base.svc.cluster.local
      fault:
        delay:
          fixedDelay: 7000ms
          percentage:
            value: 100
EOF
```

配置完成后，在Demo网站页面点击“ADD TO CART”加入购物车或者点击“YOUR CART”调用购物车服务，购物车服务会调用stock服务查询库存，访问stock服务有7秒的固定延迟故障注入策略，以及在购物车页面点击“CHECKOUT”发起结算会调用order服务，order服务会调用stock服务查询库存，访问stock服务有7秒的固定延迟故障注入策略。此时页面处于加载中的状态会持续7秒一直等待故障结束，造成网站用户的不流畅浏览体验。



<img src="/images/netCommunication/3-1-2.png"></img>

网站用户的浏览体验优化可以通过配置超时timeout实现。
