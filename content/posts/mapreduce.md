---
title: "[技術雜談] 你不能不知道的軟體架構--MapReduce (一)"
date: 2021-05-23T23:37:48+08:00
draft: false
---

{{< figure src="/images/Spark-vs-Hadoop.jpeg" attr="spark and hadoop">}}

## 前言

你聽過大數據、區塊鏈或是NOSQL資料庫嗎？如果有，那你絕對不能不了解分散式系統如何運作。在資訊爆炸的時代，企業要處理如此大量的資料通常不是依靠我們手上的小小筆電，那假如我們有了很多台機器然後呢？有什麼辦法是集結眾多機器的力量加速任務處理的呢？那就是Google的AI大神[<span style="color:#3D65A8">Jeff Dean</span>](https://en.wikipedia.org/wiki/Jeff_Dean)提出的`MapReduce`，之後許多知名的分散運算叢集都是基於這個概念，例如大家耳熟能詳的Hadoop和Spark，本篇文章將帶你深入淺出何謂`MapReduce`，下一篇則是以Go語言自幹一個`MapReduce`核心。

## 什麼是MapReduce

我們先想像有一個任務是要算出一篇很長的文章中字詞出現頻率，如果在同一台電腦我們會怎麼做呢？我們也許會遍歷整篇文章，並將字詞做紀錄，但若資料太大時這樣的方式並不可行，為了加速我們將檔案切分，並且產為經由Map function 產生 key-value pair如下：

`hello world -> {hello:1, world:1}`

接著將這些key-value pair 存成中介檔案後排序，Reduce function則根據key將所有value加總，如果以上流程在一台電腦上完成，會像下圖，前兩列中的A、B、C其實分別是Ａ:1、 B:1、C:1：

{{< figure src="/images/mapreduce_sequence.png" attr="mapreduce in sequence">}}

上面提到的任務如果在一個分散式的世界呢？[<span style="color:#3D65A8">Google提出的論文</span>](https://pdos.csail.mit.edu/6.824/papers/mapreduce.pdf)中有張重要的圖是這樣描述的：

{{< figure src="/images/mapreduce.png" attr="mapreduce in distributed systems">}}


MapReduce是一種軟體架構，由許多台機器組成，其中一台擔任Master，負責分派任務和處理來自Worker的RPC請求，其他則擔任Worker，負責處理使用者定義的Map和Reduce函式，也就是說只有可切分的大型任務才能應用MapReduce。整個概念有點類似前陣子完成的[<span style="color:#3D65A8">Worker Pool</span>]({{< ref "/posts/worker-pool.md" >}})，但是將thread改成分散於各個系統的Process。機器越多，系統擴展越困難，而MapReduce優雅的解決了這個問題，畢竟一致性是效率的敵人，後續幾篇我們將會談到。｀


## 結論

疫情肆虐，在家的各位好嗎？也許很多人的生涯規劃被打亂，期待許久的聚餐也迫於無奈取消，旅遊或生活都淪為遺憾。但在家依然有許多事可以做的，例如寫一篇廢廢的部落格聊聊近日所學，
感謝你看到這裏，願你平安。

## Reference

- MapReduce Paper: https://pdos.csail.mit.edu/6.824/papers/mapreduce.pdf
- Hadoop: https://en.wikipedia.org/wiki/Apache_Hadoop
- Spark: https://en.wikipedia.org/wiki/Apache_Spark