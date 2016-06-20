---
title: 使用 Openssl 建立憑證
date: 2016-06-07 17:10:37
tags: [linux, tls]
---

最近工作上需要建立程式之間的 TLS 連線，而 TLS 之間的連線步驟需要驗證憑證 (Certificate) ，讓之間的加密連線可以成立。不過由於處於測試階段，需要自己產生憑證出來。因此學習使用 Openssl 來產生憑證，也藉此讓自已對 TLS 連線和憑證機制有更清楚的了解。

## TLS 介紹
傳輸層安全協議（Transport Layer Security，縮寫：TLS），前身為安全通訊協定（Secure Sockets Layer，縮寫：SSL），由網景公司 (Netscape) 開發出來用在 http 協定上，後來由 IETF 將 SSL 進行標準化，就是後來的 TLS 。

TLS 在建立連線時，會經過以下的步驟：

1. client 送出 ClientHello 訊息給 server ，裡面包含可支援的 TLS protocol，一個由 client 端隨機產生的數字(隨機數字只用在此次連線期間，避免被封包重播攻擊)，以及支援的加密和壓縮演算法。
2. server 回應 ServerHello 訊息，提供要使用的 TLS protocol ，一個由 server 端隨機產生的數字，以及要使用的加密和壓縮演算法。
3. server 根據上一部選擇的方式，提供憑證給 client
4. client 驗證 server 提供的憑證，將憑證內的 public key 取出
5. 利用 server 提供的 public key ，將 client 的憑證，沒有憑證則隨機產生的 public key ，加密送回。
6. 雙方使用交換出來的資訊，產生一組通訊用的加解密方式，有對稱式或非對稱式，取決於採用哪種演算法。

## 使用 Openssl 產生憑證
以下開始描述使用 openssl 來產生可供 TLS 連線所需的憑證，主要參考 [OpenSSL Certificate Authority](https://jamielinux.com/docs/openssl-certificate-authority/index.html) 這個網站的步驟來一步一步執行，根據自己的需求修改設定。

### 建立 root 憑證
一般公開網路上憑證不會是自己產生的。因為網路傳輸過程中誰也不曉得會不會被惡意修改。因此為了有可靠的信任機制，於是就有了憑證機構 (Certificate Authority, CA) ，由憑證機構產生憑證。最上層憑證機構通常由被信任的第三方公正機關來擔任，然後對個人或公司團體產生的憑證簽章，如此才能確保憑證可以偵測出來被修改過。

以瀏覽器為例，存有一串信任的憑證機構，藉由檢視連線網站的憑證是否由這些憑證機構產生，來判別網站可否受信任，不信任情況下就會出現不安全連線的提示。

而對公司企業內的私人網路來說， 不需要透過第三方憑證機構，因此會由自己產生 root 憑證，往下建立一連串的認證階層。也因此 root 憑證必須保護好，放置在與網路隔絕的環境並只有特定人士可以存取。一般網路上的認證機構的 root 憑證會互相簽章，表示彼此信任，而自己產生的 root 憑證因沒有其他機關互相簽章，因此會是自我簽章的憑證。

#### 步驟：
1. 產生 root key。

	```sh
	mkdir /root/ca
	cd /root/ca
	openssl genrsa -aes256 -out private/ca.key.pem 4096 
	chmod 400 private/ca.key.pem
	```
2. 產生自我簽章的 root 憑證。

	```sh
	openssl req -config openssl.cnf \
      -key private/ca.key.pem \
      -new -x509 -days 7300 -sha256 -extensions v3_ca \
      -subj "/C=TW/ST=Taipei/O=RAIX/OU=RAIX.IO/CN=raix.io.rootca/emailAddress=raix@mail.com" \
      -out certs/ca.cert.pem 
	chmod 444 certs/ca.cert.pem
	```
	openssl 設定檔參照 [config](https://jamielinux.com/docs/openssl-certificate-authority/appendix/root-configuration-file.html) ，root 憑證過期日期建議設定長一點，因為是最重要的環節，不太適合頻繁的過期造成整串憑證失效。
	
	另外因為是產生自我簽章的憑證，和接下來產生 intermediate 或 client/server 的 Certificate signing request (CSR) 指令多了 `-x509` 參數。

3. 驗證 root 憑證

	```
	openssl x509 -noout -text -in certs/ca.cert.pem
	```

### 建立 intermediate 憑證
intermediate 憑證介於 root 和 client server 之間，可避免 root 憑證直接被使用，也可用來分群組，區分可使用的範圍。

#### 步驟
1. 設置相關資料夾和資訊
	
	```sh
	mkdir /root/ca/intermediate 
	cd /root/ca/intermediate 
	mkdir certs crl csr newcerts private 
	chmod 744 private 
	touch index.txt 
	echo 1000 > serial 
	echo 1000 > crlnumber
	```
2. 產生 intermediate key
	
	```sh
	openssl genrsa -aes256 \
      -out private/intermediate.key.pem 4096 
	chmod 400 private/intermediate.key.pem
	```
3. 產生 intermediate CSR

	```sh
	openssl req -config openssl-im.cnf -new -sha256 \
      -subj "/C=TW/ST=Taipei/O=RAIX/OU=RAIX.IO/CN=raix.io.imca/emailAddress=raix@mail.com"\
      -key private/intermediate.key.pem \
      -out csr/intermediate.csr.pem 
	```
	
	[Certificate signing request (CSR)](https://en.wikipedia.org/wiki/Certificate_signing_request) 裡紀錄個人或組織的資訊，並且提供給憑證機構簽章。
	
4. 對 intermediate CSR 簽上 root 憑證，產生 intermediate 憑證

	```sh
	openssl ca -config /root/ca/openssl.cnf -extensions v3_intermediate_ca \
      -days 3650 -notext -md sha256 \
      -in csr/intermediate.csr.pem \
      -out certs/intermediate.cert.pem 
	chmod 444 certs/intermediate.cert.pem
	```
5. 驗證 intermediate 憑證

	```sh
	openssl x509 -noout -text \
      -in intermediate/certs/intermediate.cert.pem 
	openssl verify -CAfile certs/ca.cert.pem \
      intermediate/certs/intermediate.cert.pem
	```
6. 產生憑證鍊 (certificate chain)

	```sh
	cat intermediate/certs/intermediate.cert.pem \
      certs/ca.cert.pem > intermediate/certs/ca-chain.cert.pem
	chmod 444 intermediate/certs/ca-chain.cert.pem
	```
	當應用程式要驗證 intermediate 憑證時，也需要同時驗證 root 憑證，因此需要產生憑證鍊來完成一整串驗證。	

### 建立 client/server 憑證
最後就是對要使用的 client/server 產生憑證，讓彼此之間 TLS 連線可以成立，確保處於安全的連線下。

#### 步驟
1. 產生 client/server key

	```sh
	cd /root/ca/intermediate/
	openssl genrsa -out private/client.raix.io.key.pem 2048 
	chmod 444 private/client.raix.io.key.pem
	```
2. 產生 client/server CSR

	```sh
	openssl req -config openssl-im.cnf \
      -key intermediate/private/client.raix.io.key.pem \
      -subj "/C=TW/ST=Taipei/O=RAIX/OU=RAIX.IO/CN=client.raix.io/emailAddress=raix@mail.com" \
      -new -sha256 -out client.raix.io.csr.pem
	```
3. 產生 client/server 憑證

	```sh
	openssl ca -config openssl-im.cnf \
      -extensions server_cert -days 375 -notext -md sha256 \
      -in intermediate/csr/client.raix.io.csr.pem \
      -out intermediate/certs/client.raix.io.cert.pem 
	chmod 444 certs/client.raix.io.cert.pem 
	```
4. 驗證憑證

	```sh
	openssl x509 -noout -text \
      -in certs/client.raix.io.cert.pem
	openssl verify -CAfile intermediate/certs/ca-chain.cert.pem \
      certs/client.raix.io.cert.pem
	```


## 參考資料
Wiki: [Transport Layer Security](https://en.wikipedia.org/wiki/Transport_Layer_Security) <br/>
Wiki: [Certificate authority](https://en.wikipedia.org/wiki/Certificate_authority) <br/>
Wiki: [Certificate signing request](https://en.wikipedia.org/wiki/Certificate_signing_request) <br/>
[OpenSSL Certificate Authority](https://jamielinux.com/docs/openssl-certificate-authority/index.html)

