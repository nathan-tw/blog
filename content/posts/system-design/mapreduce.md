---
title: "[技術] 你不能不知道的軟體架構--MapReduce (一)"
date: 2021-05-23T23:37:48+08:00
draft: true
tags: ["distirbuted-system"]
categories: ["System Design"]
thumbnailImagePosition: left
thumbnailImage: /images/mapreduce/Spark-vs-Hadoop.jpeg
---

你肯定聽過大數據，甚至學會許多處理資料的方式，但在資訊爆炸的時代，企業要處理如此大量的資料通常不是依靠我們手上的小小筆電，那假如我們有了很多台機器然後呢？

<!--more-->

有什麼辦法是集結眾多機器的力量加速任務處理的呢？最近疫情嚴重在家讀[<span style="color:#3D65A8">MIT 6.824</span>](https://pdos.csail.mit.edu/6.824/schedule.html)，對 Google 的 AI 大神[<span style="color:#3D65A8">Jeff Dean</span>](https://en.wikipedia.org/wiki/Jeff_Dean)提出的`MapReduce`又認識了許多，之後許多知名的分散運算叢集都是基於這個概念，例如大家耳熟能詳的 Hadoop 和 Spark，一起來看看知名的`MapReduce`吧。

## 什麼是 MapReduce

我們先想像有一個任務是要算出一篇很長的文章中字詞出現頻率，如果在同一台電腦我們會怎麼做呢？我們也許會遍歷整篇文章，並將字詞做紀錄，但若資料太大時這樣的方式並不可行，為了加速我們將檔案切分，並且產為經由 Map function 產生 key-value pair 如下：

`hello world -> {hello:1, world:1}`

接著將這些 key-value pair 存成中介檔案後排序，Reduce function 則根據 key 將所有 value 加總，如果以上流程在一台電腦上完成，會像下圖，前兩列中的 A、B、C 其實分別是Ａ:1、 B:1、C:1：

{{< image src="/images/mapreduce/mapreduce_sequence.jpeg" title="mapreduce in sequence, src: https://zhuanlan.zhihu.com/p/260752052">}}

上面提到的任務如果在一個分散式的世界呢？[<span style="color:#3D65A8">Google 提出的論文</span>](https://pdos.csail.mit.edu/6.824/papers/mapreduce.pdf)中有張重要的圖是這樣描述的：

{{< image src="/images/mapreduce/mapreduce.png" title="mapreduce in distributed systems">}}

MapReduce 是一種軟體架構，由許多台機器組成，其中一台擔任 Master，負責分派任務和處理來自 Worker 的 RPC 請求，其他則擔任 Worker，負責處理使用者定義的 Map 和 Reduce 函式
也就是說只有可切分的大型任務才能應用 MapReduce。而上面提到的例子如果以分散式系統實現，就會如下圖：

{{< image src="/images/mapreduce/mapreduce_distributed.jpeg" title="mapreduce in distributed systems, src: https://zhuanlan.zhihu.com/p/260752052">}}

整個概念有點類似前陣子完成的[<span style="color:#3D65A8">Worker Pool</span>]({{< ref "worker-pool.md" >}})，但是將 thread 改成分散於各個系統的 Process。機器越多，系統擴展越困難，而 MapReduce 優雅的解決了這個問題，為了套用於許多不同的任務型態，mapreduce 核心必須定義一致性的接口，這裏我們可以看到論文 2.2 提到的：

```
map (k1, v1) → list(k2, v2)
reduce (k2, list(v2)) → list(v2)
```

在任務經過切分後，每個任務會交由一個 mapper 處理，而分配的機制則是誰有空誰就去處理(queue)。在 word count 的例子中，(k1, v1)代表切分後檔案名稱及內容，就是`{filename, content}`，而(k2, v2)則代表詞與頻率，也就是`{hello: 1}`。至於 k2 還有另一個用處，就是決定他要由哪個 reducer 處理，在產出中介檔案時就會依照 mr-x-y 命名，其中 x 代表 map 函數的 index，y 代表 k2 經過 hash 的值，這個值也同時是 reducer 的 index。

{{< image src="/images/mapreduce/mapandreduce.png" title="map function works in detail">}}

可以清楚的看到切分後的小任務分別被機器執行，其實他們是在一個 queue 中等待分發，但這裡為了顯示不同檔案對應的流程才這樣畫，另一方面是想表示一個重要的概念，<span style="color:#FF5151">只有能夠切分的任務才能使用 mapReduce</span>。

## 結論

以下整理幾個重點：

-   在分散式的世界中，效率是一致性的敵人(這裡的一致性指的是各個機器間的狀態)。
-   master 負責分配任務，而任務分成 map 及 reduce 兩類。
-   mapReduce 核心負責歸類和排序，使用者則定義 map 和 reduce 確切要做的事。
-   只有能夠切分的任務才能使用 mapReduce。

## Reference

-   MapReduce Paper: https://pdos.csail.mit.edu/6.824/papers/mapreduce.pdf
-   Hadoop: https://en.wikipedia.org/wiki/Apache_Hadoop
-   Spark: https://en.wikipedia.org/wiki/Apache_Spark
-   知乎講解: https://zhuanlan.zhihu.com/p/260752052
-   我的 Lab1: https://github.com/nathan-tw/6.824
