---
title: "[技術雜談] 你不能不知道的軟體架構--MapReduce (二)"
date: 2021-06-01T16:52:44+08:00
draft: false
---

## 前言

繼上一篇[<span style="color:#3D65A8">[技術雜談] 你不能不知道的軟體架構--MapReduce (一)</span>]({{< ref "/posts/mapreduce.md" >}})提到的MapReduce概念，這篇講解的是如何完成[<span style="color:#3D65A8">MIT 6.824</span>](https://pdos.csail.mit.edu/6.824/schedule.html)的Lab1，自幹一個MapReduce核心。

## 題目要求

* 設計一個分散式的MapReduce系統，其中包含兩個程式：Coordinator和Worker。
* 系統架構中只有一個Coordinator，一或多個Worker平行運行。
* 兩者之間的溝通透過RPC (Remote Procedure Call)。
* Map階段將中介的key (前篇提到的k2)利用hash分成nReduce組。
* 第X組的Reduce任務輸出命名為`mr-out-X`。
* 中介檔案 (Map的產物)命名為`mr-X-Y`。
* 主程序會隔三差五的呼叫`Done`確認任務是否全數完成。

## 系統架構設計

{{< figure src="/images/6.824lab1.png" width="500" attr="my mapReduce system design">}}







## 測試方式

## 結論

## 心得

與其說是講解，不如說是在疫情期間為自己紀錄學習歷程，最近對分散式系統越發著迷，標題寫著`你不能不知道...`，但其實潛台詞是在責怪我自己，身為一個SRE、聽過blockchain、玩過Machine Learning、知道nosql的各種理論，卻不知道他們每一個都和分散式系統密不可分，尤其blockchain本身就是一個分散式系統 (在6.824後段課程會講解)。下一篇想寫一些關於github action如何部署hugo的網頁 (也就是此部落格的方式)。再次感謝看到這裡的你，祝你平安。