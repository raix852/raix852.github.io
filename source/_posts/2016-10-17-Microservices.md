---
title: Scalable Microservices with Kubernetes - Lesson 3
date: 2016-10-22 16:30:00
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

Kubernetes 提供了新一層超乎 deployment 的 abstractions ，讓使用者可以專注於大局。
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

```sh
$ kubectl exec monolith --stdin --tty -c monolith /bin/sh
/ #
```

## Monitor and Health Checks
Kubernetes 提供了 Health Checks 機制，讓使用者自訂檢查 applications 健康狀態和 readiness check。Readiness check 用來確認 pod 是否已經準備好可以提供服務，當 check failed 時，container 會被標記成 not ready 並從 load balancer 中移除。 Liveness probe 檢查 container 是否活著，如果檢查多次失敗， container 將會重啟。

### 問題練習
```sh
$ cat pods/monolith.yaml
...
  livenessProbe:
    httpGet:
      path: /healthz
      port: 81
      scheme: HTTP
    initialDelaySeconds: 5
    periodSeconds: 15
    timeoutSeconds: 5
  readinessProbe:
    httpGet:
      path: /readiness
      port: 81
      scheme: HTTP
    initialDelaySeconds: 5
    timeoutSeconds: 1


$ kubectl create -f pods/monolith.yaml
pod "healthy-monolith" created

$ kubectl describe pods healthy-monolith
...
  State:		Running
    Started:		Fri, 21 Oct 2016 22:37:12 +0800
  Ready:		True
  Restart Count:	0
  Liveness:		http-get http://:81/healthz delay=5s timeout=5s period=15s #success=1 #failure=3
  Readiness:		http-get http://:81/readiness delay=5s timeout=1s period=10s #success=1 #failure=3
...
```

使用 healthy-monolith pod，回答下面三個問題。

* How is the readiness probe performed?
  ```
  readinessProbe:
    httpGet:
      path: /readiness
      port: 81
      scheme: HTTP
    initialDelaySeconds: 5
    timeoutSeconds: 1
  ```
* How often is the readiness checked?
  `period=10s`
* How often is the liveness probe checked?
  `period=15s`

## Secret and ConfigMap

ConfigMap 和 Secret 相似，但是 ConfigMap 用來存放較不敏感的資料，而像密碼或金鑰之類的需要安全性的資料則使用 Secret。 Secret 會在 pod 建立時， mount 到 container 裡當作 volume，讓 container 擁有和使用。

[ConfigMap](http://kubernetes.io/docs/user-guide/configmap/)
[Secrets](http://kubernetes.io/docs/user-guide/secrets/)

```sh
# Create secret
$ kubectl create secret generic tls-certs --from-file=tls/
# Use secure-monolith pod
$ kubectl create -f pods/secure-monolith.yaml
```

以下使用 nginx reverse proxy 為例子，將 https(443 port) 連線轉送到 http(80 port)。
### Create secret
https 連線需要 Certificate ，使用 secret 將 Certificate 封裝起來。

```sh
$ kubectl create secret generic tls-certs --from-file=tls/
```
### Describe secret
kubectl will create a key for each file in the tls directory under the tls-certs secret bucket. Use the `kubectl describe` command to verify that:

```sh
$ kubectl describe secrets tls-certs
```
### Create ConfigMap
將 nginx reverse proxy 的設定透過 ConfigMap 封裝起來。

```sh
$ kubectl create configmap nginx-proxy-conf --from-file=nginx/proxy.conf
```

### Describe ConfigMap
透過 `kubectl describe configmap` 獲得詳細的資訊。

```sh
$ kubectl describe configmap nginx-proxy-conf
```

### 操作練習
使用 secure-monolith pod 作為這次練習。

```sh
$ cat pods/secure-monolith.yaml

apiVersion: v1
kind: Pod
metadata:
  name: "secure-monolith"
  labels:
    app: monolith
spec:
  containers:
    - name: nginx
      image: "nginx:1.9.14"
      lifecycle:
        preStop:
          exec:
            command: ["/usr/sbin/nginx","-s","quit"]
      volumeMounts:
        - name: "nginx-proxy-conf"
          mountPath: "/etc/nginx/conf.d"
        - name: "tls-certs"
          mountPath: "/etc/tls"
    - name: monolith
      image: "udacity/example-monolith:1.0.0"
      ports:
        - name: http
          containerPort: 80
        - name: health
          containerPort: 81
      resources:
        limits:
          cpu: 0.2
          memory: "10Mi"
      livenessProbe:
        httpGet:
          path: /healthz
          port: 81
          scheme: HTTP
        initialDelaySeconds: 5
        periodSeconds: 15
        timeoutSeconds: 5
      readinessProbe:
        httpGet:
          path: /readiness
          port: 81
          scheme: HTTP
        initialDelaySeconds: 5
        timeoutSeconds: 1
  volumes:
    - name: "tls-certs"
      secret:
        secretName: "tls-certs"
    - name: "nginx-proxy-conf"
      configMap:
        name: "nginx-proxy-conf"
        items:
          - key: "proxy.conf"
            path: "proxy.conf"
```

在 secure-monolith pod 設定裡，可以看到使用兩個 container ，分別為 nginx 和 monolith ， nginx 會作為 reverse proxy 將連線導入 monolith 。

在 nginx 設定裡，有設定 `lifecycle` 當 nginx container 要關閉時，先執行所設定的 command ，清理剩餘的工作。在 `volumeMounts` 裡，設定先前建立的 secret 和 ConfigMap ，將之 mount。

### Create secure-monolith pod
```sh
Create the secure-monolith Pod using kubectl.

kubectl create -f pods/secure-monolith.yaml
kubectl get pods secure-monolith
```

### Check connection
使用先前學過的 port-forward 指令，將 local 10443 port 開通到 nginx 443 port ，測試 TLS 連線能不能相通。

```sh
kubectl port-forward secure-monolith 10443:443

curl --cacert tls/ca.pem https://127.0.0.1:10443

kubectl logs -c nginx secure-monolith
```

## Services
不管是使用 pod 還是 docker container ，都會遇到一個問題是，container 會因各種原因需要進行重啟，而重啟時 IP 會重新指派，因此無法仰賴固定的 IP 。因此 Kubernetes 引入 Services 當作 pod 的 abstraction ，由 Services 當作是 pods 前端，提供穩定的連接，因此後端的 pods 的更動，就不會影響整體。

Persistent Endpoint for Pods<br/>
* Use labels to select pods
* Internal or External IP

### Create Services
```sh
$ cat services/monolith.yaml
kind: Service
apiVersion: v1
metadata:
  name: "monolith"
spec:
  selector:
    app: "monolith"
    secure: "enabled"
  ports:
    - protocol: "TCP"
      port: 443
      targetPort: 443
      nodePort: 31000
  type: NodePort
```

Services 設定裡透過 `selector` 將有符合 label 的 pods 選出來聚在一起。並把所有 pods 443 port 暴露出來，讓 local 31000 port 轉到 pods 443 port。

接著建立 services 。<br/>
```sh
$ kubectl create -f services/monolith.yaml
```

這時 services 沒有選擇到任何東西 ，因為沒有一個 pod 的 label 符合，可以透過以下指令檢查。<br/>
```sh
$ kubectl get pods -l "app=monolith,secure=enabled"
```

將 secure-monolith pod 加入 label。 <br/>
```sh
$ kubectl label pods secure-monolith "secure=enabled"
$ kubectl get pods -l "app=monolith,secure=enabled"
NAME              READY     STATUS    RESTARTS   AGE
secure-monolith   2/2       Running   0          5h
```

此時 services 就可以接管 secure-monolith pod 。
