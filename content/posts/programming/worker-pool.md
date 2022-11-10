---
title: "[技術] Worker Pool併發處理模型"
date: 2021-04-19T05:03:58-04:00
draft: false
tags: ["Go"]
categories: ["Programming"]
---

前陣子在準備實習的面試，看到別人的心得被問到如何以Golang實做一個Worker Pool，於是自己嘗試寫了一個。

## 什麼是Worker Pool

![worker-pool](/images/worker-pool/worker-pool.png)

Worker Pool是一個以multithread組成的任務處理模型，producer產生許多任務，並交由workers並行處理這些任務，最後將任務結果蒐集起來，這樣的作法可以有效的運用電腦資源，並快速處理重複性高且獨立的作業。

## Worker Pool 設計
 
### Type 定義

Worker Pool可以有一定數量的Workers(PoolSize)，並且可以指定一次處理的任務數量(tasksSize)，交由tasksChan送給worker，處理後再經由resultsChan存起來，待Results被呼叫時一一取出。而Task則是定義了id, error 和如何process的function。

```go
type Task struct {
	Id  int
	Err error
	f   func() error
}

type WorkerPool struct {
	PoolSize    int
	tasksSize   int
	tasksChan    chan Task
	resultsChan chan Task
	Results     func() []Task
}
```

### 功能實做

每一個任務處理完都會回傳error，如順利完成則回傳nil，至於任務如何執行則是依據新增任務時給定。

```go
func (task *Task) Do() error {
	return task.f()
```

Worker Pool是利用buffered channel做任務的通道，所以可以經由Start將worker一一執行起來後，讓worker利用range的特性不斷監聽是否有任務可以做。

```go
func NewWorkerPool(tasks []Task, size int) *WorkerPool {
	tasksChan, resultsChan := make(chan Task, len(tasks)), make(chan Task, len(tasks))
	for _, task := range tasks {
		tasksChan <- task
	}
	close(tasksChan)
	pool := &WorkerPool{
		PoolSize: size,
		tasksSize: len(tasks),
		tasksChan: tasksChan,
		resultsChan: resultsChan,
	}
	pool.Results = pool.results
	return pool
}

func (pool *WorkerPool) Start() {  
    for i := 0; i < pool.PoolSize; i++ {  
        go pool.worker()  
    }  
}  

func (pool *WorkerPool) worker() {  
    for task := range pool.tasksChan {  
        task.Err = task.Do()  
        pool.resultsChan <- task  
    }  
}  

func (wp *WorkerPool) results() []Task {
	tasks := make([]Task, wp.tasksSize)
	for i := 0; i < wp.tasksSize; i++ {
		tasks[i] = <-wp.resultsChan
	}
	return tasks
}
```

## Reference

* https://github.com/nathan-tw/workerpool
* https://github.com/gammazero/workerpool
* https://github.com/xxjwxc/gowp
