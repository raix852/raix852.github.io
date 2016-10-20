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

## Get started
**Launch a single instance**<br/>
```sh
$ kubectl run nginx --image=nginx:1.10.0
```
將 nginx container 透過 kubectl 指令啟動

**Get pods**<br/>
```sh
$ kubectl get pods
```
在 Kubernetes 裡，所有的 Containers 運行在 pod ，使用上述指令列出正在運行的 pods

**Expose nginx**<br/>
```sh
$ kubectl expose deployment nginx --port 80 --type LoadBalancer
```
上面指令會建立出 load balancer 且派發一個 public IP 在上面，並將 80 port 暴露給外面。任何一個 Client 透過派發的 IP 連線將會經過 load balancer 傳送到內部 container，在這邊的例子是 nginx 。

**List services**<br/>
```sh
$ kubectl get services
```
列出 services 資訊，像是 Public IP, expose port 。

## Pods
Kubernetes 的核心是 Pods ，Pods 代表的是 Logical Application 。<br/>
* **One or more containers** - 將具有依賴關係的 containers 包在同一個 pod 裡。
* **Volumes** - containers 放置資料的地方，可以被所有在 pod 裡的 containers 使用
* **Shared namespace** - Shared namespace 表示在相同 pod 裡的 containers 互相溝通，並分享相同的 Volumes
* **One IP per pod** - 在同一個 pod 裡的 containers 共享相同的 public IP 。

### Pods Configuration
在 Kubernetes 裡，建立 pods 是透過設定 yaml 格式的設定檔。如下 `monolith.yaml`：

```sh
$ cat pods/monolith.yaml
apiVersion: v1
kind: Pod
metadata:
  name: monolith
  labels:
    app: monolith
spec:
  containers:
    - name: monolith
      image: udacity/example-monolith:1.0.0
      args:
        - "-http=0.0.0.0:80"
        - "-health=0.0.0.0:81"
        - "-secret=secret"
      ports:
        - name: http
          containerPort: 80
        - name: health
          containerPort: 81
      resources:
        limits:
          cpu: 0.2
          memory: "10Mi"
```

在 `contains:` 底下標示 container 的屬性。包含

* `name:` : container 名稱
* `image:` : 要使用的 container image
* `args:` Application 執行時所需的參數
* `port:` : 需要暴露給外面的 ports
* `resources:` : 限制 cpu 和記憶體使用量。


### Create the pod
將設定好的 `monolith.yaml` 透過底下指令將 monolith pod 建立出來。

```sh
$ kubectl create -f pods/monolith.yaml
pod "monolith" created
```


### Examine pods
在 pod 剛建立出來時需要準備一段時間讓 container 啟動。透過以下指令觀察是否建立出來以及正在運行。

```sh
$ kubectl get pods
NAME       READY     STATUS    RESTARTS   AGE
monolith   1/1       Running   0          14s
```

### Describe pods
需要檢查詳細的資訊可以透過以下指令。

```sh
$ kubectl describe pods monolith
```

會列出 container 被指派的 IP，暴露的 port ，以及其他許多的 metadata 和發生的 events log。

### Port forward
設定一個或多個 local port 轉送到 pod 裡的 port。

```sh
$ kubectl port-forward monolith 10080:80
Forwarding from 127.0.0.1:10080 -> 80
Forwarding from [::1]:10080 -> 80
```

這道指令會處於 listen 狀態，開啟另一個 terminal 測試 monolith 連線。

```sh
$ curl http://127.0.0.1:10080
{"message":"Hello"}
$ curl http://127.0.0.1:10080/secure
authorization failed
```

測試 login。

```sh
$ curl -u user http://127.0.0.1:10080/login
Enter host password for user 'user': password
{"token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6InVzZXJAZXhhbXBsZS5jb20iLCJleHAiOjE0NzcxNTA3ODQsImlhdCI6MTQ3Njg5MTU4NCwiaXNzIjoiYXV0aC5zZXJ2aWNlIiwic3ViIjoidXNlciJ9.Rgm5BBG8VSiLH-u26Am1QAaXtz05nzvfWWZhHjOcaus"}
$ curl -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJlbWFpbCI6InVzZXJAZXhhbXBsZS5jb20iLCJleHAiOjE0NzcxNTA3ODQsImlhdCI6MTQ3Njg5MTU4NCwiaXNzIjoiYXV0aC5zZXJ2aWNlIiwic3ViIjoidXNlciJ9.Rgm5BBG8VSiLH-u26Am1QAaXtz05nzvfWWZhHjOcaus" http://127.0.0.1:10080/secure
{"message":"Hello"}
```

### View logs
列出 pod 裡的 container log。

```sh
$ kubectl logs monolith
2016/10/19 15:04:10 Starting server...
2016/10/19 15:04:10 Health service listening on 0.0.0.0:81
2016/10/19 15:04:10 HTTP service listening on 0.0.0.0:80
127.0.0.1:59954 - - [Wed, 19 Oct 2016 15:22:34 UTC] "GET / HTTP/1.1" curl/7.47.0
127.0.0.1:59986 - - [Wed, 19 Oct 2016 15:22:48 UTC] "GET /secure HTTP/1.1" curl/7.47.0
127.0.0.1:33138 - - [Wed, 19 Oct 2016 15:38:29 UTC] "GET /login HTTP/1.1" curl/7.47.0
127.0.0.1:33226 - - [Wed, 19 Oct 2016 15:39:26 UTC] "GET /login HTTP/1.1" curl/7.47.0
127.0.0.1:33252 - - [Wed, 19 Oct 2016 15:39:43 UTC] "GET /login HTTP/1.1" curl/7.47.0
127.0.0.1:33430 - - [Wed, 19 Oct 2016 15:41:41 UTC] "GET /secure HTTP/1.1" curl/7.47.0
127.0.0.1:33492 - - [Wed, 19 Oct 2016 15:42:20 UTC] "GET /secure HTTP/1.1" curl/7.47.0
```

加 `-f` 進入 stream 模式，監控即時 log 。

### Enter container
需要像 `docker exec` 進入 container 執行指令的話，透過底下指令。

```
$ kubectl exec monolith --stdin --tty -c monolith /bin/sh
/ #
```

## Monitor and Health Checks
