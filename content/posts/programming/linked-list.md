---
title: "[技術] 深入淺出 linked list"
date: 2022-01-21T06:30:15-04:00
draft: false
tag: ["Linked List", "Array"]
categories: ["Programming"]
thumbnailImagePosition: left
thumbnailImage: /images/linked-list/linked.jpg
---

假設我們有一串有順序性的數組，要以何種形式存入記憶體呢？
我們操作的虛擬記憶體就像是一格格的櫥窗，工程師在配置使用空間時有兩種常見的方式：
<!--more-->

1. 配置一段連續的格子 (Array)
2. 配置不連續的格子，並以額外的格子紀錄順序性 (Linked list)
    > With a linked list, we can store a list of values that can easily be grown by storing values in different parts of memory

{{< figure src="/images/linked-list/memory.png"  width="700">}}

因為連續的記憶體是可以隨機存取(random access)的，導致兩種配置方式在資料操作上(e.g., 新增、刪除)的時間及空間成本各有不同。

### Array vs. Linked list

兩者在記憶體的結構：

{{< figure src="/images/linked-list/array_data_schema.png"  width="700" attr="Data storage scheme of an array" >}}

{{< figure src="/images/linked-list/linkedlist_data_schema.png"  width="700" attr="Data storage scheme of a linked list" >}}

* Array：造訪第i個元素時因為是隨機存取所以時間複雜度是O(1)，但因為Array內的元素必須是連續的，移除第i個元素時必須將其後的元素通通往前移，因此為O(n)。

* Linked list：無法利用連續記憶體的隨機存取特性，造訪第i個元素時必須先造訪1至i-1，因此時間複雜度為O(n)，且因為必須以額外的指標做相連，使得存入相同的資料時較Array耗費空間。好處是空間配置較為彈性，刪除第i個節點時時間複雜度為O(1)，並且不需要在配置空間時固定長度。


比較兩者使用的空間及特性：
<table><tbody>
<tr>
<th style="width: 228px;" class=""></th>
<th>Linked List</th>
<th>Array</th>
</tr>
<tr>
<td style="width: 228px;">空間</td>
<td>較複雜(需要額外pointer)</td>
<td>較節省</td>
</tr>
<tr>
<td style="width: 228px;">查詢第i個元素</td>
<td>O(n)</td>
<td class="">O(1)</td>
</tr>
<tr>
<td>刪除第i個元素</td>
<td>O(1)</td>
<td>O(n)</td>
</tr>
<tr>
<td>優點</td>
<td>大小可不固定</td>
<td>隨機存取</td>
</tr>
</tbody></table>

### Dynamic Array

如果array大小固定，python中的list與go中的slice是如何實現？為何不須在一開始給定Array大小，也沒有剛剛提到的固定長度限制？
其實他們都是維護一個[Dynamic Array](https://en.wikipedia.org/wiki/Dynamic_array)，簡單來說就是一個包含Array的類別或結構體，空間不夠時再抽換Array。以go為例，slice的結構如下：

```go
type slice struct {
    array unsafe.Pointer
    len int
    cap int
}
```

`array`是一個指向底層Array的指標，如果執行`append`時發現`cap`等於`len`則會擴容，也就是更換pointer。

<div class="sl-block is-focused" data-block-type="image" data-name="image-09dfc7" style="width: 489.224px; height: 337.612px; left: 445.085px; top: 237.368px; min-width: 1px; min-height: 1px;" data-origin-id="e5ec20aa8b9d20867e8713bdf0a622b1"><div class="sl-block-content" style="z-index: 12;"><img style="" data-natural-width="626" data-natural-height="432" data-lazy-loaded="" src="https://s3.amazonaws.com/media-p.slid.es/uploads/1585165/images/9251604/WskKaXf.png"></div></div>

## Linked list 與 pointer

Linux開發者**Linus Torvalds**在[2016的TED演講](https://www.youtube.com/watch?v=o8NPllzkFhE)中提到了Linux的哲學，其中他舉了一個刪除Linked List節點的例子：

{{< youtube o8NPllzkFhE >}}
<br/>

<img src="/images/linked-list/remove_node.png"/>

以刪除節點為例，考慮到移除第一個節點`head`需要做例外處理，因為移除第一個節點時沒有辦法以上一個節點出發：
```c
void remove_list_node(List *list, Node *target)
{
    Node *prev = NULL;
    Node *current = list->head;
    // Walk the list
    while (current != target) {
        prev = current;
        current = current->next;
    }
    // Remove the target by updating the head or the previous node.
    if (!prev)
        list->head = target->next;
    else
        prev->next = target->next;
}
```

Linus認為有品味的作法是維護一個指向要更新的位址的indirect pointer，以「檢查某個位址是否需要被更新」的想法，程式碼將大大減少為4行：

```c
void remove_list_node(List *list, Node *target)
{
    Node **indirect = &list->head;
    while (*indirect != target)
        indirect = &(*indirect)->next;
    *indirect = target->next;
}
```

假設有`A`, `B`, `C` 三個節點相連，需要維護目前指標及上一個節點的指標，檢查目前指標是否為Target，如果是則更改上一個指標指向的值：
<img src="/images/linked-list/direct.png"/>

後者的想法則是以一個指標的指標順著指標走，尋找「需要被更新」的位址：
<img src="/images/linked-list/indirect.png"/>


## Linux kernel 中的 linked list
我們常見的Linked list結構是將資料存放於結構體內，但Linux kernel的`list_head`卻不是這麼做：
```c
// usually
struct list_node {
	int data;
	struct list_node *next;
}

// linux/scripts/kconfig/list.h
struct list_head {
	struct list_head *next, *prev;
};
```

使用方式如下：
```c
struct student
{
    char name[16];
    int id;
    struct list_head list;
};
```

如果資料不在Linked list的結構體內，不就只能一直循環取不到資料了嗎？假設Tom的下個節點是Jerry，該如何取得Jerry的id？

```c
struct list_node *Jerry_list = (Tom->list).next;
```

結構體連接示意圖：
<div class="sl-block is-focused" data-block-type="image" data-name="image-467510" style="width: 560px; height: 237.545px; left: 183.693px; top: 361.962px; min-width: 1px; min-height: 1px;" data-origin-id="d93128418da5740dfe459fa43a2706d6"><div class="sl-block-content" style="z-index: 11;"><img style="" data-natural-width="554" data-natural-height="235" data-lazy-loaded="" src="https://s3.amazonaws.com/media-p.slid.es/uploads/1585165/images/9254793/pasted-from-clipboard.png"></div></div>

Linux kernel提供了一個很巧妙的macro `container_of`，只要知道結構體型別，就能以member的位置回推出他所在的container位置。
因此剛剛上面的例子可以用這種方式取得Jerry：
```c
struct student
{
    char name[16];
    int id;
    struct list_head list;
};

struct list_node *Jerry_list = (Tom->list).next;
struct student *Jerry = container_of(jerry_list, struct student, list);
```

那`container_of`是如何實做的呢？我們先從另一個macro `offsetof`開始理解。

### offsetof

給定一個結構體student，要如何算出結構體與其成員的偏移呢？

<img src="/images/linked-list/struct.png" width="300px"/>

C89/C99 提供 offsetof 巨集，傳統上`offsetof`是這樣實做的：

```c
#define offsetof(TYPE, MEMBER) ((size_t) &((TYPE *)0)->MEMBER)
```
將數值 0 強制轉型成TYPE指標型別，0 會被當作該TYPE的起始地址，因為起始位址等於 0
，所以MEMBER的位址也就等於MEMBER與起始位址  0 的偏移(offset)。

```c
offsetof(struct student, id)); //16
```

### container_of

container_of的定義較為複雜：
```c
#define container_of(ptr, type, member) ({\
        const typeof( ((type *)0)->member ) *__mptr = (ptr);\
              (type *)( (char *)__mptr - offsetof(type,member));})
```

可以拆解為兩部份，第一部份為：
```c
const typeof( ((type *)0)->member ) *__mptr = (ptr);
```
利用ptr得到該member型態同值的__mptr，
以member是list為例就是x+20，
(但目前不知道offset為20)。

既然有了__mptr就輕鬆了，第二部份可以利用`offsetof`計算所在的結構體位置：
```c
(type *)( (char *)__mptr - offsetof(type,member));
```
	
回頭看看Tom如何取得Jerry，利用`struct student`中的成員 `list`以`offsetof`取得Jerry的指標。
```c
struct student *Jerry = container_of(Jerry_list, struct student, list);
```

一切便大功告成！這樣的方式使許多非連續記憶體的操作更廣泛使用，例如[network address table](https://github.com/torvalds/linux/blob/master/net/netlabel/netlabel_addrlist.h)及檔案系統的[namespace](https://github.com/torvalds/linux/blob/master/fs/mount.h)等等。

## 小結與後記

* Linked list 使用時機：
    * 無法預期資料數量或頻繁變動資料數量時。
    * 需要頻繁地新增/刪除資料時。
    * 不需要快速查詢資料。

* 程式可以用不同角度思考

>人們對「品味」的概念各有見解，談到程式碼是否有品味，開發者又是如何執行判斷？Linux之父 Linus Torvalds曾提到一個案例來說明，他認為，重點並不在於邊界情況（Edge case），而是能以不同的角度看問題、直覺地選擇出正確的方法

在[良葛格](https://www.ithome.com.tw/voice/139264)的部落格中也談到了，品味並非完全主觀的，其中涉及文化及時空，他以品酒為例，品酒的禮儀、使用的杯器、產地、品牌、文化等等，這些都能是品味的客觀因素。

>粗製濫造的程式碼固然是可以解決問題，然而，這可能也代表了一件事：手邊可以解決問題的方式，少得可憐。

## Reference

-   linked list vs. array: http://alrightchiu.github.io/SecondRound/linked-list-introjian-jie.html#comp
-   linked list vs. array: https://www.geeksforgeeks.org/linked-list-vs-array/
-   array and linked List: https://pjchender.dev/dsa/dsa-array-linked-list/
-   list_head 結構: http://adrianhuang.blogspot.com/2007/10/linux-kernel-listhead.html
-   良葛格blog: https://www.ithome.com.tw/voice/139264
