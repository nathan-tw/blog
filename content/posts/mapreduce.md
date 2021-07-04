---
title: "[技術雜談] 你不能不知道的軟體架構--MapReduce (一)"
date: 2021-05-23T23:37:48+08:00
draft: false
tags: ["Distirbuted System"]
cover:
    image: "/images/Spark-vs-Hadoop.jpeg"
---

## 前言

你肯定聽過大數據，甚至學會許多處理資料的方式，但在資訊爆炸的時代，企業要處理如此大量的資料通常不是依靠我們手上的小小筆電，那假如我們有了很多台機器然後呢？有什麼辦法是集結眾多機器的力量加速任務處理的呢？最近疫情嚴重在家讀[<span style="color:#3D65A8">MIT 6.824</span>](https://pdos.csail.mit.edu/6.824/schedule.html)，對Google的AI大神[<span style="color:#3D65A8">Jeff Dean</span>](https://en.wikipedia.org/wiki/Jeff_Dean)提出的`MapReduce`又認識了許多，之後許多知名的分散運算叢集都是基於這個概念，例如大家耳熟能詳的Hadoop和Spark，本篇文章將帶你深入淺出何謂`MapReduce`，下一篇則是以Go語言自幹一個`MapReduce`核心 ，有興趣可以先去我的[<span style="color:#3D65A8">github repo</span>](https://github.com/nathan-tw/6.824)看。

## 什麼是MapReduce

我們先想像有一個任務是要算出一篇很長的文章中字詞出現頻率，如果在同一台電腦我們會怎麼做呢？我們也許會遍歷整篇文章，並將字詞做紀錄，但若資料太大時這樣的方式並不可行，為了加速我們將檔案切分，並且產為經由Map function 產生 key-value pair如下：

`hello world -> {hello:1, world:1}`

接著將這些key-value pair 存成中介檔案後排序，Reduce function則根據key將所有value加總，如果以上流程在一台電腦上完成，會像下圖，前兩列中的A、B、C其實分別是Ａ:1、 B:1、C:1：

{{< figure src="/images/mapreduce_sequence.jpeg" attr="mapreduce in sequence, src: https://zhuanlan.zhihu.com/p/260752052">}}

上面提到的任務如果在一個分散式的世界呢？[<span style="color:#3D65A8">Google提出的論文</span>](https://pdos.csail.mit.edu/6.824/papers/mapreduce.pdf)中有張重要的圖是這樣描述的：

{{< figure src="/images/mapreduce.png" attr="mapreduce in distributed systems">}}

MapReduce是一種軟體架構，由許多台機器組成，其中一台擔任Master，負責分派任務和處理來自Worker的RPC請求，其他則擔任Worker，負責處理使用者定義的Map和Reduce函式
也就是說只有可切分的大型任務才能應用MapReduce。而上面提到的例子如果以分散式系統實現，就會如下圖：

{{< figure src="/images/mapreduce_distributed.jpeg" attr="mapreduce in distributed systems, src: https://zhuanlan.zhihu.com/p/260752052">}}

整個概念有點類似前陣子完成的[<span style="color:#3D65A8">Worker Pool</span>]({{< ref "/posts/worker-pool.md" >}})，但是將thread改成分散於各個系統的Process。機器越多，系統擴展越困難，而MapReduce優雅的解決了這個問題，為了套用於許多不同的任務型態，mapreduce核心必須定義一致性的接口，這裏我們可以看到論文2.2提到的：
```
map (k1, v1) → list(k2, v2)
reduce (k2, list(v2)) → list(v2)
```
在任務經過切分後，每個任務會交由一個mapper處理，而分配的機制則是誰有空誰就去處理(queue)。在word count的例子中，(k1, v1)代表切分後檔案名稱及內容，就是`{filename, content}`，而(k2, v2)則代表詞與頻率，也就是`{hello: 1}`。至於k2還有另一個用處，就是決定他要由哪個reducer處理，在產出中介檔案時就會依照mr-x-y命名，其中x代表map函數的index，y代表k2經過hash的值，這個值也同時是reducer的index。好的我知道許多人到這裡可能完全混亂，以下我用一張圖說明：

{{< figure src="/images/mapandreduce.png" attr="map function works in detail">}}

可以清楚的看到切分後的小任務分別被機器執行，其實他們是在一個queue中等待分發，但這裡為了顯示不同檔案對應的流程才這樣畫，另一方面是想表示一個重要的概念，<span style="color:#FF5151">只有能夠切分的任務才能使用mapReduce</span>。

## 結論

以下整理幾個重點：
* 在分散式的世界中，效率是一致性的敵人(這裡的一致性指的是各個機器間的狀態)。
* master負責分配任務，而任務分成map及reduce兩類。
* mapReduce核心負責歸類和排序，使用者則定義map和reduce確切要做的事。
* 只有能夠切分的任務才能使用mapReduce。

## 心得
疫情肆虐，在家的各位好嗎？也許很多人的生涯規劃被打亂，期待許久的聚餐也迫於無奈取消，旅遊或生活都淪為遺憾。但在家依然有許多事可以做的，例如寫一篇廢廢的部落格聊聊近日所學，下一篇我會談談最近在看的課程[MIT6.824](https://pdos.csail.mit.edu/6.824/schedule.html)中lab1的實作，著墨較多於系統設計，如果看不懂go也沒關係。
感謝你看到這裏，願你平安。

## Reference

- MapReduce Paper: https://pdos.csail.mit.edu/6.824/papers/mapreduce.pdf
- Hadoop: https://en.wikipedia.org/wiki/Apache_Hadoop
- Spark: https://en.wikipedia.org/wiki/Apache_Spark
- 知乎講解: https://zhuanlan.zhihu.com/p/260752052
- 我的Lab1: https://github.com/nathan-tw/6.824