---
title: Data at rest encryption
date: 2016-7-01 23:10:37
tags: [Encryption, Filesystem, Linux]
---

目前一般的 data at rest 有三種方式，分別為：

1. Encrypted drives

	獨立於作業系統之外，由硬體提供加密的功能。避免硬體遺失時資料被輕易拿出，缺點是無法避免系統運行時被惡意使用者或惡意程序拿取資料。

2. Full disk encryption

	在這種模式下不需要硬體支援，可針對 disk 或 volume 整個加密。實作方式各家 filesystem 都有不同，所以也有不同的限制，像是有的可以對 /root 資料夾加密，有的不行。缺點和 Encrypted drives 一樣無法避免 disk 或 volume 運行起來時被惡意使用者或惡意程序拿取資料。

3. Filesystem encrytion

	又稱為 file/folder encryption 。此種方式可以讓使用者分別對某個檔案或資料夾加密，加密時也可選擇不同的 key ，因此優點就是可以避免上面兩種方式的缺點，讓惡意使用者或程序難以解讀內容。缺點是針對密碼控管需要注意，特別是加密很多東西時。

	其實我初看到 Filesystem encryption 名詞時還以為是對整個 filesystem 加密，不過後來查 wiki 看到別稱才明白是指什麼。

## 參考資料
[Disk encryption](https://en.wikipedia.org/wiki/Disk_encryption) <br/>
[Filesystem-level encryption](https://en.wikipedia.org/wiki/Filesystem-level_encryption)
