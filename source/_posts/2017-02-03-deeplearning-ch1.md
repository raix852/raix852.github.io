---
title: Deep learning 讀書筆記 - CH. 1
date: 2017-2-03 10:30:00
tags: [Machine Learning, Deep Learning]
---

## 目標

今年的目標是進一步學習 Deep learning ，利用 [Deep Learning - An MIT Press Book](http://www.deeplearningbook.org/) 當作學習的教材。

## 第一章 - Introduction
在早期 AI 發展的時代，電腦擅長於處理人們不擅長的事物，像是計算複雜的公式、重複許多次的作業。但是真正有挑戰的是對人們擅長的事物但是難以正規地描述，像是辨識物品、了解語言。

要處理這些對人們擅長但是電腦難以描述的問題，解決方案是讓電腦可以從經驗上學習，並且能透過建立層層的概念起來了解世界。一層一層的概念從簡單的開始逐漸堆積，每一層由前一層較簡單的概念轉成複雜的概念，越上層就是越複雜的概念，以此讓電腦可以學習到複雜的概念。這些概念層會有很多 layer ，呈現的 graph 也很 deep ，因此叫做 AI Deep learning 。

早期 AI 成功是基於有限的環境和明確的條件下，例如 IBM Deep Blue(1997) 擊敗西洋棋冠軍 Garry Kasparov 。在西洋棋的環境中，只有 64 個位置和32個受規則限制棋子可以移動的地方。因此只要工程師將每條規則和公式詳細規劃出來，透過當時的運算力即可達成此一創舉。因此這種具有抽象的計算概念和有正式的規則對電腦而言就非常的簡單，而且處理的環境有限，因此可以在 1996 年時就成功擊敗人類。但是直到最近幾年，電腦識別物品才成功到達人類的精確度。

有不少 AI project 是透過人工建立的知識庫，透過條件判斷和邏輯推導公式將知識編成一條條規則，這種方式稱為 knowledge base approach 。但是沒有一個取得巨大的成功。

由於在 knowledge base 遇到的困難，產生了由 AI 自己學習自己的知識庫，脫離人工建立的模式，此種方式就稱為 Machine learning 。簡單的 logistic regression 可以用來是否需要 cesarean delivery (查一下是剖腹產)，簡易的 navie Bayes 可以用來區分是否為垃圾郵件。這些簡單的 machine learning 演算法依賴於資料的 representation ，representation 裡所含有的部分資訊稱為 feature。在 Machine learning 裡，選擇好的 representation 對演算法的表現會有很大的影響。許多問題只要提供好的一組 features 作為 representation ，就能被簡單的 Machine learning 演算法解決。

但是更多的問題是難以知道如何抽出 features 出來。假如要設計出一個能辨識出車子的程式，我們知道車子有輪子，所以我們可以用輪子當作 feature ，不幸的是我們難以描述輪子看起來會是怎樣，畢竟在圖片上輪子會因角度不同呈現不同的幾何樣貌，以及會受到光影的影響，或是被擋泥板遮住部分，因此無法成功設計成一個 feature 使用。

一個解決的方式是不只去發現問題的 representation 對應輸出結果的本身 ，而且也要自己學習出問題的 representation ，這種方式稱為 representation learning 。學習出來的 representation 通常會比人工設計出來的表現較佳，而且可以通用於較多的問題上，減少人為的介入。 Representation 對簡單的問題可以幾分鐘能就學習出一組好的 feature 出來，而人為設計則需花費無數倍的時間。

典型的  案例是 autoencoder ， autoencoder 結合了 encoder 和 decoder ， encoder 負責將 input 轉成 representation ， decoder 負責將 representation 轉回原來的樣子。 Autoencoder 學習目標就是盡可能在這兩個轉換中減少資訊的損失，並且學習出一個好的 representation 涵蓋許多好的 properties 。

當設計 futures 或演算法去學習 features ，通常目標是區分出可以解釋 observed data 的 factors of variation 。 Factor 可簡單的解釋成對原始資料的影響力。每個 factor 影響程度可能存在未知的作用造成最後觀察的結果，無法輕易的量化或運算。因此對真實世界的 artificial intelligence 來說，有多少 factors of variation 影響每一小片段觀察的結果，是主要困難點的地方。當難以學習到 representation 時， representation learning 就無法提供有用的幫助。

Deep learning 透過將 representations 轉成另一些較簡單的 representations 來解決這些困難的問題，可以經由較簡單的概念堆積成較複雜的概念。如下圖所示
![Figure 1.2:Illustration of deep learning model - Deep Learning CH.1 P.6](http://i.imgur.com/W18bldS.png)

基礎的 Deep learning model 是 feedforward deep network 或稱為 multilayer perceptron (MLP) 。一個 multilayer perceptron 是經由一個數學函式將一組 input 對應到一組 output 。這個函式由許多簡單的函式組合而成，可以將之想像成不同的函式提供新的 representations 來表示 input 。

學習正確的 representations 是一個觀點，另一個觀點是 depth 可以讓電腦學習出多步驟的程式，每一層產出的 representations 視為是一組指令執行完後所保留的 memory ，越多層可以執行的指令組越多。根據這樣的觀點，每一層的產出就不一定需要 encode 成 factors of variation 來解釋 input 。可以是儲存執行過程中的狀態。

Deep learning 的深度有兩種計量方式。第一種是有多少在序列上的指令需要執行。可以想成是由 input 到 output 所經過的最常路徑長度。另一種是由 deep probabilistic models 使用，是根據有多少 concepts 彼此關聯，根據此種方式計算出來的 representations 有可能比 graph 上的 concepts 更深。比如說有一個 AI 系統想要偵測人眼，給一張被陰影遮住半邊的臉，此系統首先要先偵測人臉，接著找出一隻眼睛的位置，如此來推測出另一眼睛可能也存在。這種方式只有兩層，一層偵測人臉，一層偵測眼睛，但是每一層裡會有 n 層來計算這些特徵。因此以第一種方式來測會是 2n 的長度。因此對 Deep learning 深度並沒有一個正確的答案，但是 deep learning 可以被指稱為比傳統 machine learning 包含更多學習函式的組成或學習到的觀念。

總結來說，deep learning 是一種 AI 的方法，一種 machine learning 的類型，一種技術讓電腦可以從經驗中、從資料中學習。 Machine learning 是目前 AI 系統中，較可以適應於複雜的世界之上。 Deep learning 更是其中的佼佼者，可以展現出強大和具有彈性的能力學習複雜交錯的概念。