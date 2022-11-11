---
title: "[技術] 淺談 file descriptor 及 I/O Redirection"
date: 2021-07-07T01:55:05-04:00
draft: false
tags: ["Operating System"]
categories: ["Linux"]
thumbnailImagePosition: left
thumbnailImage: /images/io-redirection/direction.jpg
---

```c
fprintf(1, "hello world\n");
```

你可能好奇過 C 語言`fprintf`函數中，第一個參數`1`代表什麼，你也許聽過在 Unix 家族中**Everything is a file**，可是他究竟代表什麼意思呢？這篇我們想談談究竟什麼是`file`，以及作業系統如何達到`i/o redirection`。

<!--more-->

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

從上面的例子中可以理解，一個 process 可以透過打開一個檔案、資料夾、裝置或是製造一個`pipe`獲取 fd。當然我們也可以複製 fd，透過 system call `dup`可以達到等等會說的兩個 process 交換訊息。到這裡你可能會好奇，fd 是如何對資源進行控制的呢？事實上是透過剛剛提到的`open file entry`去訪問系統層級的`open file table`，再去 inode 查找自己需要的資源，以下用一張圖說明兩者的關係：

{{< image src="/images/io-redirection/fd_table.png" title="relationship between fd tables, open file table and inode table">}}

由此可知，所有的 file 都指向一個底層的 inode。以 xv6 系統為例，在`kernel/stat.h`是這樣定義 inode 的資訊：

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

如果要獲取某個 fd 或是某個 file 所指向的 inode 資訊，我們可以使用以下兩個 system call：

```c
int fstat(int fd, struct stat *st)
int stat(char *file, struct stat *st)
```

`fstat`是用 fd 獲取 inode 資訊，`stat`則是獲取某個 path 的 inode 資訊，而 stat 資訊則會被放入\*st 中，根據執行後的 st.type，我們可以知道目前使用的資源是 file, directory 或是 device。到這裡可能會開始困惑，為什麼要那麼麻煩呢？這樣的設計有什麼好處？下一篇我們會談到檔案系統的設計。

## I/O redirection 和 stdin, stdout, stderr

```c
int fprintf(FILE *stream, const char *format, ...)
```

有人可能會問不同 process 對同一個 file 做`open`會得到一樣的 fd 嗎？如同前面所說 fd table 是每個 process 各自擁有的，所以理論上會不同(或碰巧相同)，但有些情況卻不是碰巧，要了解原因必須先回答開頭的問題，`fprintf`中第一個參數`1`是什麼呢？相信很多人已經猜到，答案便是 fd1，因為 POSIX <unistd.h> 的定義是 process 在啟動時會先開啟三個 stream: stdin, stdout, stderr，也就是我們剛剛說的 fd 0, 1, 2，預設是分別連接到用戶的終端設備，通常就是鍵盤和螢幕啦，如同下圖表示：

{{< image src="/images/io-redirection/io_redirection.jpg" title="stdin, stdout, stderr">}}

但也不是所有 process 都是這樣預設，因為如果經由一個 parent process`fork`得到的 child process，將會複製 parent 的 fd table，這也是達成 redirection 的重點，process 在安排 fd 時有一個原則：

> 以尚未被使用的最小 int 作為新的 fd

所以我們可以透過抽換 fd 0, 1, 2 的值去完成 redirection，舉我們常使用的`cat < input.txt`command 來說，簡單的 code 其實是：

```c
char *argv[2];
argv[0] = "cat";
argv[1] = 0;
if (fork()==0) { // child process
    close(0); // 把fd0關掉
    // 把指向input.txt資源的fd放在fd table 0的位置 (因為剛才0被關掉，所以最小是0)
    open("input.txt", O_RDONLY);
    exec("cat", argv);
}
```

上面的例子是在 shell 執行`cat < input.txt`的簡單範例，因為 shell 本身也是一個 process，所以透過`fork`去執行其他 process 時才有機會抽換 fd 0, 1, 2，當`fork`回傳 0 代表示 child process，這也是為什麼`fork`和`exec`要分成兩個 system call，如果沒辦法抽換 stdin, stdout 和 stderr，i/o 就沒有彈性，也就是 cat 只能吃到 shell 的輸入，無法吃一個 file 作為輸入。既然 child 會複製 parent，而我們又可以抽換他，因此偉大的`pipe`程式(`|`)也被人發明出來，以簡單的指令`echo "hello world" | wc`為例：

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

經由`pipe`串接不同的 process，原本用法單調的程式就可以有許多組合技！下一篇寫 file system 對不同層的設計，讓我們如此方便的使用**Everything is a file**的概念。

## 心得

因為很常使用 linux，但對 linux 卻一直是一知半解，所以才決定寫這篇。但寫的時候很怕出錯，許多細節都更深入的查資料，還是有許多地方自己也看不懂的，如果有任何錯誤也歡迎在底下留言～

## Reference

-   宅色夫老師筆記: https://hackmd.io/@sysprog/c-stream-io?type=view
-   Redirection-in-bash: https://blog.hexrabbit.io/2019/10/22/Redirection-in-bash/
-   Ubuntu manual page: http://manpages.ubuntu.com/manpages/bionic/zh_TW/man3/stdin.3.html
