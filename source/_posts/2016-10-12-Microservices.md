---
title: Scalable Microservices with Kubernetes - Lesson 2
date: 2016-10-12 23:00:00
tags: [學習筆記, Udacity, Kubernetes, docker]
---

第二節課在教 docker 觀念以及如何使用，內容對初心者而言非常簡易上手，對我來說也可當作複習的機會。

## Why docker?
docker 所使用的 image 不只包含程式本身，所需要的 dependencies 和執行時所需的設定檔也都包含在內。docker 本身也提供良好的 API
* **create**
* **distribute**
* **run**

container 在 server 上。目前支援的 OS 平台有 linux 和 windows。

docker 可以幫助系統管理者作到

* **Reproducable**
* **Consistant**

docker 可以提供類似 virtual machine 的功能，將執行程式從同一台機器上隔離出來，因此可不用擔心各種 dependencies 問題。

## docker 擅長什麼？
常見的 linux 發布版(Ex. Ubuntu, Redhat) ，擅長於

* **Installing Services**
* **Start and Stop Services**

docker 在這之上，提供了

* **Installing Services**
* **Start and Stop Services**
* **Running multiple versions**
* **Running multiple instances**

舉例，要使用 nginx 套件， linux 的套件管理程式可以方便下載，並透過 systemd 或舊的 init.d 將 nginx 啟動。但假設需要在同一台機器上啟動多個 nginx 服務或使用不同的版本，此時就會遇到瓶頸，因為同一時間只能跑一個。而 docker 則可以提供運行多個 instances 和多個版本的能力。

## Containers
可以讓 applications 方便運行和發布到不同平台上的技術。

* **Independent packages** - 將所需要的 dependencies 打包起來，發布到各種平台上。
* **Namespace isolation** - 提供 filesystem 的 Iiolation 和 network isolatation，讓檔案名稱或 ip/port 不會衝突。

與一般的 virtual machine 透過模擬整個 OS 層不同， container 是經由 linux kernel 由邏輯上分割出來，透過 Namespace 隔離和 cgroup 控制 resources 達成，因此可視為輕量級的 VM ，啟動和關閉都非常快速。

## Dockerfile
docker 用來建立 image 的檔案，裡面描述指令讓 docker 依指示要從哪個 base image 開始，逐漸將所需要的 dependencies 打包起來，最後完成一個包含所有套件和設定的 application image 來使用。

以下是之前做過得小型 java 7 image，使用 centos 7.2 作為程式運行的環境。
```
FROM centos:7.2.1511
MAINTAINER raix

USER root
RUN yum install -y wget \
                git \
                curl \
                zip \
                openssh-clients \
                java-1.7.0-openjdk \
                java-1.7.0-openjdk-devel \
                java-1.7.0-openjdk-headless

ENV LANG C.UTF-8
```

## Docker Hub
做出來的 image 可以發布到 docker hub 上，公開讓所有人使用，直接從 docker hub 上下載完整的 image 下來，而不用從頭建置起來，方便用來做大量佈署的動作。除了 docker hub 這個公開存放 image 的地方，也有其他公開的服務可以使用，像是 gcr (Google Container Registry) ，或是建立私有 hub ，供企業內部使用。
