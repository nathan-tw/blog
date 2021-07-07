---
title: "[技術雜談] 淺談 file descriptor 及 I/O Redirection"
date: 2021-06-26T01:55:05-04:00
draft: false
tags: ["Operating System"]
categories: ["programming"]
cover:
    image: "/images/direction.jpg"
---

## 前言

```c
fprintf(1, "hello world\n");
```

你可能好奇過 C 語言`fprintf`函數中，第一個參數`1`代表什麼，你也許聽過在 Unix 家族中**Everything is a file**，可是他究竟代表什麼意思呢？這篇我們想談談究竟什麼是`file`，以及作業系統如何達到`i/o redirection`。

## 什麼是 file

> _Everything is a file_

先從這句話開始講起，在類 Unix 的設計中，對所有 I/O 資源的近用都是透過資料流的方式，也就是透過 file system 定義的檔案描述檔(file descriptor)來傳輸，當開啟這些資源時就會回傳一個 file descriptor，代表的就是對這個 file 的控制，例如大家熟悉的 System call `open`:

```c
int open(char *file, int flags)
```

其中`*file`代表 path, `flags`代表 read/write，用法例如：
```c
fd = open("/tmp/temp", O_WRONLY|O_CREAT);
```
那為什麼是回傳一個 int 呢？其實那個 int 就是 file descriptor，因為每個 process 都有一個 fd (file descriptor) table，其中包含了`fd flag`以及`open file entry`，根據[xv6 book](https://pdos.csail.mit.edu/6.828/2020/xv6/book-riscv-rev1.pdf)對其的描述是：

> _A **file descriptor** is a small integer representing a kernel-managed object that a process may read from or write to._

從上面的例子中可以理解，一個 process 可以透過打開一個檔案、資料夾、裝置或是製造一個`pipe`獲取 fd。當然我們也可以複製 fd，透過 system call `dup`可以達到等等會說的兩個process交換訊息。到這裡你可能會好奇，fd 是如何對資源進行控制的呢？事實上是透過剛剛提到的`open file entry`去訪問系統層級的`open file table`，再去inode查找自己需要的資源，以下用一張圖說明兩者的關係：

{{< figure src="/images/fd_table.png" attr="relationship between fd tables, open file table and inode table">}}

由此可知，所有的file都指向一個底層的inode。以 xv6 系統為例，在`kernel/stat.h`是這樣定義inode的資訊：

```c
#define T_DIR     1   // Directory
#define T_FILE    2   // File
#define T_DEVICE  3   // Device

 struct stat {
   int dev;     // File system's disk device
   uint ino;    // Inode number
   short type;  // Type of file
   short nlink; // Number of links to file
   uint64 size; // Size of file in bytes
 };

```

如果要獲取某個fd或是某個file所指向的inode資訊，我們可以使用以下兩個system call：

```c
int fstat(int fd, struct stat *st)
int stat(char *file, struct stat *st)
```

`fstat`是用fd獲取inode資訊，`stat`則是獲取某個path的inode資訊，而stat資訊則會被放入*st中，根據執行後的st.type，我們可以知道目前使用的資源是file, directory或是device。到這裡可能會開始困惑，為什麼要那麼麻煩呢？這樣的設計有什麼好處？下一篇我們會談到檔案系統的設計。

## I/O redirection 和 stdin, stdout, stderr

```c
int fprintf(FILE *stream, const char *format, ...)
```
如同有人可能會問不同process對同一個file做`open`會得到一樣的fd嗎？如同前面所說fd table是每個process各自擁有的，所以理論上會不同(或碰巧相同)，但有些情況卻不是碰巧，要了解原因必須先回答開頭的問題，`fprintf`中第一個參數`1`是什麼呢？相信很多人已經猜到，答案便是fd1，因為POSIX <unistd.h> 的定義是process在啟動時會先開啟三個stream: stdin, stdout, stderr，也就是我們剛剛說的fd0, 1, 2，預設是分別連接到用戶的終端設備，通常就是鍵盤和螢幕啦，如同下圖表示：

{{< figure src="/images/io_redirection.jpg" attr="stdin, stdout, stderr">}}

但也不是所有process都是這樣預設，因為如果經由一個parent process`fork`得到的child process，將會複製parent的fd table，這也是達成redirection的重點，process在安排fd時有一個原則：

> 以尚未被使用的最小int作為新的fd

所以我們可以透過抽換fd 0, 1, 2的值去完成redirection，舉我們常使用的`cat < input.txt`command來說，簡單的code其實是：

```c
char *argv[2];
argv[0] = "cat";
argv[1] = 0;
if (fork()==0) { // child process
    close(0); // 把fd0關掉every
    // 把指向input.txt資源的fd放在fd table 0的位置 (因為剛才0被關掉，所以最小是0)
    open("input.txt", O_RDONLY); 
    exec("cat", argv);
}
```

上面的例子是在shell執行`cat < input.txt`的簡單範例，因為shell本身也是一個process，所以要透過`fork`去執行其他process，當`fork`回傳0代表示child process，這也是為什麼`fork`和`exec`要分成兩個system call，如果沒辦法抽換stdin, stdout和stderr，i/o就沒有彈性，也就是cat只能吃到shell的輸入，無法吃一個file作為輸入。既然可以child會複製parent，而我們又可以抽換他，因此偉大的`pipe`程式(`|`)也被人發明出來，以簡單的指令`echo "hello world" | wc`為例：

```c
int p[2];
char *argv[2];

argv[0] = "wc";
argv[1] = 0;

pipe(p); // 創建一個pipe，將read/write fd分別放在p[0], p[1]
if(fork()==0) { // child process
    close(0); // 關掉從parent複製來的stdin
    dup(p[0]); // 把pipe的read fd複製一份放到fd0
    close(p[0]); // pipe用不到了便關掉
    close(p[1]);
    exec("/bin/wc", argv); // wc會從stdin讀東西，目前stdin是剛剛設的read pipe
} else {
    close(p[0]);
    write(p[1], "hello world\n", 12); // 把"hello world"寫進write pipe
    close(p[1]);
}

```
經由`pipe`串接不同的process，原本用法單調的程式就可以有許多組合技！下一篇寫file system對不同層的設計，讓我們如此方便的使用**Everything is a file**的概念。

## Reference

- 宅色夫老師筆記: https://hackmd.io/@sysprog/c-stream-io?type=view
- Redirection-in-bash: https://blog.hexrabbit.io/2019/10/22/Redirection-in-bash/
- Ubuntu manual page: http://manpages.ubuntu.com/manpages/bionic/zh_TW/man3/stdin.3.html





