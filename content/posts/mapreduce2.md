---
title: "[技術雜談] 你不能不知道的軟體架構--MapReduce (二)"
date: 2021-06-01T16:52:44+08:00
draft: false
---


{{< figure src="/images/distributed_network.jpg"  attr="distributed network, src: https://unsplash.com/photos/OKOOGO578eo">}}


## 前言

繼上一篇[<span style="color:#3D65A8">[技術雜談] 你不能不知道的軟體架構--MapReduce (一)</span>]({{< ref "/posts/mapreduce.md" >}})提到的MapReduce概念，這篇講解的是如何完成[<span style="color:#3D65A8">MIT 6.824</span>](https://pdos.csail.mit.edu/6.824/schedule.html)的Lab1，自幹一個MapReduce核心。雖然網路上許多人說這個 Lab不難，但其實我花了不少的時間(汗)，而且因為是由golang完成的，對goroutine與channel又必須有一些了解，所以我認為沒學過go的人要完成是蠻有挑戰的，也因為這樣，以下的軟體結構我盡量以圖來說明，程式碼的部分瀏覽即可。

## 題目要求

首先我們再回顧一下論文中的圖：


{{< figure src="/images/mapreduce.png"  attr="mapReduce in paper">}}

我將[<span style="color:#3D65A8">Lab1</span>](https://pdos.csail.mit.edu/6.824/labs/lab-mr.html)中的幾個重點列出，由於並非全部要求，若想完成Lab還是需要看原網址的描述。

* 設計一個分散式的MapReduce系統，其中包含兩個程式：Coordinator和Worker。
* 系統架構中只有一個Coordinator，一或多個Worker平行運行。
* 兩者之間的溝通透過RPC (Remote Procedure Call)。
* Map階段將中介的key (前篇提到的k2)利用hash分成nReduce組。
* 第X組的Reduce任務輸出命名為`mr-out-X`。
* 中介檔案 (Map的產物)命名為`mr-X-Y`。
* 主程序會隔三差五的呼叫`Done`確認任務是否全數完成。

## 系統架構設計

根據以上幾點要求，我的想法是：

* 用一個會block的queue (channel in golang)作為任務佇列。
* 將任務分為兩階段：Map和Reduce，當所有任務的Map階段完成後，在最新一次`Done`被呼叫時更新狀態為Reduce階段。
* Worker不停向Master (Coordinator)要求任務，並更新自身狀態

設計上如下圖：

{{< figure src="/images/6.824lab1.png"  width=500 attr="my mapReduce design">}}

基本上如果只是要理解設計的話到這就結束了，接下來會談在程式碼上如何實作。我們先從`Coordinator`和`Worker`的主程式開始看起：

```go

// main/mrcoordinator.go
func main() {
	if len(os.Args) < 2 {
		fmt.Fprintf(os.Stderr, "Usage: mrcoordinator inputfiles...\n")
		os.Exit(1)
	}

	// 第一個參數是*.txt，代表切分後的文件
	// 第二個參數則是worker數量(nReduce)
	m := mr.MakeCoordinator(os.Args[1:], 10)
	for m.Done() == false {
		time.Sleep(time.Second)
	}

	time.Sleep(time.Second)
}
```

```go
// main/mrworker.go

func main() {
	if len(os.Args) != 2 {
		fmt.Fprintf(os.Stderr, "Usage: mrworker xxx.so\n")
		os.Exit(1)
	}

	mapf, reducef := loadPlugin(os.Args[1])

	mr.Worker(mapf, reducef)
}

```

可以從上面的程式看到`Coordinator`啟動後會每過一秒確認一次任務完成了嗎，而這個Lab還有一個要求是測試的程式`mian/test-mr.sh`要在10s內完成。而下方的`Worker`很簡單，就是單純的把`mapf`和`reducef`傳入即可。接者我們看看`Coordinator`本身的設計：

```go

type Coordinator struct {
	TaskChan chan Task // 任務等待的quque
	Files []string // 要處理的文件
	MapNum int  // nMap, 要處理的map任務數量，其實就是files數量
	ReduceNum int // nReduce, reduce任務數量
	TaskPhase int // 0: mapPhase, 1: reducePhase
	TaskState []TaskState // 任務狀態
	Mutex sync.Mutex // 互斥鎖, 以防coordinator的狀態被不同worker同時讀寫
	IsDone bool //任務是否全數完成
}

```

簡單來說，由於`Coordinator`要處理讀取任務(input)和`Worker`請求(output)兩者，就必須兼具分派任務且維持本身資訊穩定的特性，也因此需要`Mutex`來處理。更細部的處理，例如`Task`和`TaskState`的設計可以到我的[<span style="color:#3D65A8">專案</span>](https://github.com/nathan-tw/6.824)看，這裡就不贅述，接著看`Worker`的設計：

```go
func Worker(mapf func(string, string) []KeyValue, reducef func(string, []string) string) {
	for {
		reply := requestTask() // 只有在任務全數完成時，回傳會是true
		if reply.TaskDone {
			break
		}

		err := do(mapf, reducef, reply.Task)
		if err != nil {
			reportTask(reply.Task.TaskIndex, false) // 回傳任務, 如果任務出錯了, arg2給予false
			continue
		}
		reportTask(reply.Task.TaskIndex, true)
	}
}

```

## 結論

## 心得

與其說是講解，不如說是在疫情期間為自己紀錄學習歷程，最近對分散式系統越發著迷，標題寫著`你不能不知道...`，但其實潛台詞是在責怪我自己，明明想成為SRE，知道blockchain、用過nosql、學過data science，卻不知道他們每一個都和分散式系統密不可分，尤其blockchain本身就是一個分散式系統 (在6.824後段課程會講解)，想想還是應把基礎慢慢不足，現階段得趕快知道我必須讀什麼，當兵才不會太無聊。下一篇想寫一些關於github action如何部署hugo的網頁 (也就是此部落格的方式)。再次感謝看到這裡的你，祝你平安。