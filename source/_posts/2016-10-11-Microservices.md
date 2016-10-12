---
title: Scalable Microservices with Kubernetes - Lesson 1
date: 2016-10-11 23:00:00
tags: [學習筆記, Udacity]
---

這門課介紹使用 docker 、 Kubernetes  ，並且會使用 Go 和 Google cloud platform 來學習建立 Microservices 。
課程網址：[Scalable Microservices with Kubernetes](https://classroom.udacity.com/courses/ud615/)

## 什麼是 Microservices？

* Moduler
* Easy to deploy
* Scale independently

Microservices 設計模式可以套用在很多地方，像是 Web service。幫助快速開發和 Continuous delivery ，並將測試工具和 infrastructure 設計推上更上一層樓。

## Twelve-Factor App
Heroku 在 2012 年提出的 [The TWELVE-FACTS APP](https://12factor.net/)，在『建構微服務』一書中也有提到這 12 條準則。

1. Codebase - One codebase tracked in revision control, many deploys
2. Dependencies - Explicitly declare and isolate dependencies
3. Config - Store config in the environment
4. Backing services - Treat backing services as attached resources
5. Build, release, run - Strictly separate build and run stages
6. Processes - Execute the app as one or more stateless processes
7. Port binding - Export services via port binding
8. Concurrency - Scale out via the process model
9. Disposability - Maximize robustness with fast startup and graceful shutdown
10. Dev/prod parity - Keep development, staging, and production as similar as possible
11. Logs - Treat logs as event streams
12. Admin processes - Run admin/management tasks as one-off processes

Twelve-Factor App 可歸類成下列三個重要因素：

* Portability - 消除執行環境的不同，像是 dependencies 和 configuration
* Deployability - 可持續佈署在不同環境，或是不同的雲服務商(AWS, GCP)
* Scalability - 根據使用者需求擴充服務能力。

## JWT
JSON Web Token，用來做：

* Authentication - 可被使用者簽證
* Information Exchange - 可打包訊息

JWT 可被眾多語言支援 encode / decode ，常用於 Authentication 。

## How does JWT work?
1. Client 端傳送使用者帳號和密碼到 Server 上
2. Server 驗證完使用者之後，建立 JWT Token
3. 傳送 JWT Token 到 Client 端，由 Client 端保存
4. Client 端傳送 request ，並附上 JWT Token
5. Server 端驗證 JWT Token 是否正確
6. 驗證成功， Server 端傳回 request 的 response
