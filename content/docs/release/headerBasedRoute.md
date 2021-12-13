---
title: trafficRoute
description: "Doks comes with commands for common tasks."
lead: "Doks comes with commands for common tasks."
date: 2020-10-13T15:21:01+02:00
lastmod: 2020-10-13T15:21:01+02:00
draft: false
images: []
menu:
  docs:
    parent: "release"
weight: 141
toc: true
---
## 2.2 基于流量特征内容的路由

<img src="/images/releaseAndObserve/2-2-6.svg"></img>

背景：网站计划推出会员积分抵扣运费的活动以发展会员。电商网站策划了会员积分抵扣订单金额的新功能，当前部署的order服务由v1 deployment提供，没有运费抵扣的功能；网站新开发了order服务的v2版本，有积分抵扣运费的功能。网站希望可以于请求的header中是否会员的cookie信息进行路由，会员路由至order v2（有运费抵扣功能），非会员路由至order v1（无运费抵扣功能）。

提交以下yaml文件至集群-广州，部署order v2至集群

```yaml
kubectl apply -f - <<EOF
apiVersion: apps/v1
kind: Deployment
metadata:
  name: order-v2
  namespace: base
  labels:
    app: order
    version: v2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: order
      version: v2
  template:
    metadata:
      labels:
        app: order
        version: v2
    spec:
      containers:
        - name: order
          image: ccr.ccs.tencentyun.com/zhulei/testorder2:v1
          imagePullPolicy: Always
          env:
            - name: POD_NAME
              valueFrom:
                fieldRef:
                  fieldPath: metadata.name
            - name: REGION
              value: "guangzhou"
          ports:
            - containerPort: 7000
              protocol: TCP
EOF
```
部署完成后，由于还未配置路由规则，此时访问order服务的流量会被随机路由至v1版本或v2版本。

<img src="/images/releaseAndObserve/2-2-1.png"></img>
配置基于流量特征内容的路由规则前先需要通过DestinationRule定义order服务的两个版本。
<img src="/images/releaseAndObserve/2-2-2.png"></img>
<img src="/images/releaseAndObserve/2-2-3.png"></img>

两个版本定义完成后，通过VirtualService定义按流量特征进行路由，请求的header-cookie中vip=false时路由至order服务的v1 subset，vip=true时路由至order服务的v2 subset。即会员的请求路由至order v2，非会员的请求路由至order v1。提交以下yaml资源至集群-广州，即可完成上述配置。

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
    - match:
        - headers:
            cookie:
              exact: vip=true
      route:
        - destination:
            host: order.base.svc.cluster.local
            subset: v2
EOF
```

配置完成后，可登陆会员帐号（ID：1-3）加购和买单，发现有运费抵扣功能，流量被路由到了order v2版本；登陆非会员账号（ID：4-5）加购和买单，发现无运费抵扣功能，根据header中的VIP字段信息，请求被路由到了最初部署的order v1版本。版本信息也可通过左下角悬浮窗中的信息观察。

<img src="/images/releaseAndObserve/2-2-4.png"></img>
<img src="/images/releaseAndObserve/2-2-5.png"></img>
<img src="/images/releaseAndObserve/2-2-6.svg"></img>
