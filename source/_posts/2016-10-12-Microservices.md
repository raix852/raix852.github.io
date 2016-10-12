---
title: Scalable Microservices with Kubernetes - Lesson 2
date: 2016-10-12 23:00:00
tags: [學習筆記, Udacity]
---

第二節課在教 docker 觀念以及如何使用，內容對初心者而言非常簡易上手，對我來說也可當作複習的機會。

## Why docker?
docker 所使用的 container image 不只包含程式本身，所需要的 dependencies 和執行時所需的設定檔也都包含在內。docker 本身也提供良好的 API 和 **create**, **distribute**, **run** container 在 server 上。目前支援的 OS 平台有 linux 和 windows。

docker 可以幫助系統管理者作到

* **Reproducable**
* **Consistant**

docker 可以提供類似 virtual machine 的功能，將執行程式從同一台機器上隔離出來，因此可不用擔心各種 dependencies 問題。

## docker 擅長什麼？
常見的 linux 發布版(Ex. Ubuntu, Redhat) ，擅長於

* Installing Services
* Start and Stop Services

docker 在這之上，提供了

* Installing Services
* Start and Stop Services
* Running multiple versions
* Running multiple instances

舉例，要使用 nginx 套件， linux 的套件管理程式可以方便下載，並透過 systemd 或舊的 init.d 將 nginx 啟動。但假設需要在同一台機器上啟動多個 nginx 服務或使用不同的版本，此時就會遇到瓶頸，因為同一時間只能跑一個。而 docker 則可以提供運行多個 instances 和多個版本的能力。
