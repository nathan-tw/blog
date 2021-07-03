---
title: "[技術雜談] 淺談 file descriptor 及 I/O Redirection"
date: 2021-06-26T01:55:05-04:00
draft: true
tags: ["Operating System"]
categories: ["programming"]
---

{{< figure src="/images/direction.jpg" attr="src: https://unsplash.com/photos/pKeF6Tt3c08">}}


## 前言

```c
fprintf(1, "hello world\n");
```
你可能好奇過C語言`fprintf`函數中，第一個參數`1`代表什麼，你也許聽過在Unix家族中**Everything is a file**，可是他究竟代表什麼意思呢？這篇我們想談談究竟什麼是`file`，以及作業系統如何達到`i/o redirection`。

## 什麼是file

> *Everything is a file*

先從這句話開始講起，在類Unix的設計中，對所有I/O資源的近用都是透過資料流的方式，也就是透過file system定義的檔案描述檔(file descriptor)來傳輸，當開啟這些資源時就會回傳一個file descriptor，代表的就是對這個file的控制，例如大家熟悉的System call `open`:

```c
int open(char *file, int flags)
```

其中`*file`代表path, `flags`代表read/write，那為什麼是回傳一個int呢？其實那個int就是file descriptor，因為每個process都有一個fd (file descriptor) table，其中包含了`fd flag`以及`open file entry`，根據[xv6 book](https://pdos.csail.mit.edu/6.828/2020/xv6/book-riscv-rev1.pdf)對其的描述是：

> *A **file descriptor** is a small integer representing a kernel-managed object that a process may read from or write to.*

從上面的例子中可以理解，一個process可以透過打開一個檔案、資料夾、裝置或是製造一個`pipe`獲取 fd。當然我們也可以複製fd，透過system call `dup`可以達到等等會說的`i/o redirection`。到這裡你可能會好奇，fd是如何對資源進行控制的呢？事實上是透過剛剛提到的`open file entry`去訪問`open file table`，以下用一張圖說明兩者的關係：

{{< figure src="/images/fd_table.png" attr="relationship between fd tables, open file table and inode table">}}

## I/O redirection 和 stdin, stdout, stderr
