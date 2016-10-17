---
title: Scalable Microservices with Kubernetes - Lesson 3
date: 2016-10-12 23:00:00
tags: [學習筆記, Udacity, Kubernetes, docker]
---

第三堂課介紹 Kubernetes。

**Conway's Law**<br/>
```
Organizations which design systems ... are constrained to produce designs which are copies of the communication structures of these organizations
    — M. Conway[3]
```

## What is Kubernetes?
將程式包成 container 只解決了 5% 的問題。真實的問題是

* **App Configuration**
* **Service Discovery**
* **Managing Updates**
* **Monitoring**

Kubernetes 提供了新一層遠超乎 deployment 的 abstractions ，讓使用者可以專注於大局。
Kubernetes 可以讓我們將每台機器抽象起來，讓操作 cluster 起來就像邏輯上操作一台機器一樣。
在 Kubernetes 裡，我們可以描述一串 applications ，並如何讓他們互動，而 Kubernetes 將讓這些事情發生。

## Setting up Kubernetes for this course
課程上是使用 Google Cloud Platform(GCP)，而我是準備好一台機器上面安裝好 Kubernetes 來進行這個課程。

[Course Repo](https://github.com/udacity/ud615)

##Get started
**Launch a single instance**<br/>
```sh
kubectl run nginx --image=nginx:1.10.0
```
將 nginx container 透過 kubectl 指令啟動

**Get pods**<br/>
```
kubectl get pods
```
在 Kubernetes 裡，所有的 Containers 運行在 pod ，使用上述指令列出正在運行的 pods

**Expose nginx**<br/>
```
kubectl expose deployment nginx --port 80 --type LoadBalancer
```
上面指令會建立出 load balancer 且派發一個 public IP 在上面，並將 80 port 暴露給外面。任何一個 Client 透過派發的 IP 連線將會經過 load balancer 傳送到內部 container，在這邊的例子是 nginx 。

**List services**<br/>
```
kubectl get services
```
列出 services 資訊，像是 Public IP, expose port 。

##Pods
Kubernetes 的核心是 Pods ，Pods 代表的是 Logical Application 。<br/>
* **One or more containers** - 將具有依賴關係的 containers 包在同一個 pod 裡。
* **Volumes** - containers 放置資料的地方，可以被所有在 pod 裡的 containers 使用
* **Shared namespace** - Shared namespace 表示在相同 pod 裡的 containers 互相溝通，並分享相同的 Volumes
* **One IP per pod** - 在同一個 pod 裡的 containers 共享相同的 public IP 。
