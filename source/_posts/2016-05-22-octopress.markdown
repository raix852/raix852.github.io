---
layout: post
title: "Octopress 安裝和佈署到 Github"
date: 2016-05-22 20:33:29 +0800
comments: true
tags:
  - Octopress
  - Github
---

時常在逛網路 blog 時可以看到有不少人架設在自己的 Github 上，查了一下才了解 Github 有提供 [Github Pages](https://pages.github.com/) 的功能，可以讓使用者免費架設自己的網站。不過當然有一些限制，如下：

1. 只能使用靜態網頁，無法透過 ajax 方式來交換前後端資料，動態生成出網頁。
2. 每個月有 100 GB 和 100,000 requests 限制。
3. Source repositories 有 1 GB 限制。
4. 連線只能透過 http ，因此不適合傳送敏感資料。

然而以上限制對一般使用者來說，並不太會碰到。因此可以好好放心的架設自己的 blog 在 Github 上面，接著再透過 git 強大的版本控制功能，對文章內容可以好好保存修改紀錄。

在 Github 官網上官方推薦使用 [Jekyll](http://jekyllrb.com/) ，Github Pages 也是透過 Jekyll 做出。不過在架設的教學文章上，還是 Octopress 居多數，因此採用 Octopress 作為我的 blog 基礎。至於其中差異，我就沒有打算深入比較。

## 安裝
在我常用的機器上，有灌 Ubuntu 16.04 的一般桌機和 Mac 筆電，安裝時參考[Octopress](http://octopress.org/docs/) 官網文件提供的安裝方式。在安裝在 ubuntu 環境上時有遇上一些問題，而在 mac 上安裝就很順利 。因此對 Ubuntu 上提供詳細的安裝步驟。

### Ubuntu 16.04 環境
1. 安裝 git。

    ```sh
    sudo apt-get install -y git
    ```

2. 安裝 ruby 和 rbenv。

    ```sh
    sudo apt-get install -y ruby2.3 ruby2.3-dev rbenv
    ```

3. 下載 Octopress。

    ```sh
    git clone git://github.com/imathis/octopress.git octopress
    cd octopress
    ```

4. 安裝 dependencies，這邊需要額外安裝 nodejs ，不然在`bundle install`時會出現錯誤。

    ```sh
    sudo apt-get install -y nodejs
    gem install bundler
    rbenv rehash    # If you use rbenv, rehash to be able to run the bundle command
    bundle install
    ```

5. 安裝 Octopress 預設的 theme。

    ```sh
    rake install
    ```

至此 Octopress 就算安裝完成，接下來就開始來設定 blog 吧。

## 設定
將 Octopress 設定架設到 Github 上。

1. 建立以自己為域名(Ex. `your_username.github.io`)的 Github repository。

2. Octopress 設定 Github repository，輸入自己的 repository (Ex. `git@github.com:your_username/your_username.github.io.git`)。

    ```sh
    rake setup_github_pages
    ```

3. 產生樣板網頁，在這步驟中`rake generate`會根據設定產生樣板網頁出來，而`rake deploy`會將網頁佈署到 Github 上。完成之後就可以開啟自己的 Blog 出來看囉。

    ```sh
    rake generate
    rake deploy
    ```

4. 到這步就可以開始變更 blog 設定，新增文章，並將修改 commit。

    ```sh
    git add .
    git commit -m 'your message'
    git push origin source
    ```

Octopress 除了提供架設在 Github 上以外，還有提供架設在 [heroku](https://www.heroku.com/) 和透過 rsync 的方式，有需求的人就請操考官方文章操作吧。

## 產生 About
想要在 blog 上有一個分頁是 "About" 來介紹嗎？以下兩個步驟可以輕鬆產生。

1. 產生 about 頁面

    ```sh
    rake new_page["about"]
    ```
    這步驟會產生新的檔案在 `source/about/index.markdown` ，接下來就編輯這份文件來寫內容。

2. 加入上方分頁連結。

    編輯`source/_includes/custom/navigation.html`，加入第4行的內容。

    ```html
    <ul class="main-navigation">
        <li><a href="{{ root_url }}/">Blog</a></li>
        <li><a href="{{ root_url }}/blog/archives">Archives</a></li>
        <li><a href="{{ root_url }}/about">About</a></li>
    </ul>
    ```

    以上就可產生 About 頁面囉。

## 產生新文章並發表
### 產生新文章
使用 Octopress 提供的指令來生出範本。

```sh
rake new_post["title"]
```

新的範本會產生在`source/_posts/`下，開啟就可以開始透過 markdown 語法編輯文章內容。

### 發表
編輯完成後別急著使用 deploy 指令開始發表，Octopress 有提供預覽的功能，透過以下指令來預覽。

```sh
rake generate
rake preview
```

接著使用瀏覽器開起`localhost:4000`，就能搶先看文章呈現的樣貌。

確認都沒有問題後，就能 deploy 出去啦。

```sh
rake deploy
```

上傳完之後要記得還需要 commit 將紀錄保存下來呦。
