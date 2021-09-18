---
title: "[心得] 2021面試心得 Dcard/Garmin/Amazon/IBM"
date: 2021-05-06T01:28:04-04:00
draft: false
tags: ["Career"]
cover:
    image: "/images/intern/contribution.png"
---



## 背景

投遞時間落在2021/3-5月，大四即將畢業，實在對讀書沒什麼熱忱，雖然讀碩對職涯來說體質比較好，試著花了大約一個半月讀研究所(看github 的 contribution就知道有一個多月的空白qq)，但最後只有備取就認命開始找工作吧。想過將兵役提前，但今年因為疫情許多本來要出國的人或在國外的學生回台，造成等待兵役的人數增加，提前未必能在畢業後馬上進入軍營，於是有了先找工作慢慢等兵單的念頭，因此主要找的是可以給preoffer或以轉正為目標的實習為主。

## Dcard (backend intern)

{{< figure src="/images/intern/dcard.png" attr="dcard logo">}}

從大二就注意到Dcard的實習了，第一印象是年輕有活力的公司讓我很嚮往，但那時我的web知識匱乏連題目都看不懂(題目google就有，每年都一樣)，只好花時間多充實自己。大三時看到一位[<span style="color:blue">厲害的學長</span>](https://oldmo860617.medium.com/)分享Dcard實習，覺得裡面的人對技術都相當有熱忱，也在gopher conf聽了Dcard backend精彩的演講，於是這次完成了[<span style="color:blue">作業</span>](https://github.com/nathan-tw/gin-rate-limiter)決定嘗試看看。

### 作業要求

Dcard 每天午夜都有大量使用者湧入抽卡，請設計一個 middleware：

-   限制每小時來自同一個 IP 的請求數量不得超過 1000
-   在 response headers 中加入剩餘的請求數量 (X-RateLimit-Remaining) 以及 rate limit 歸零的時間 (X-RateLimit-Reset)
-   如果超過限制的話就回傳 429 (Too Many Requests)
-   可以使用各種資料庫達成

### 實做方式

限定只能用golang或nodejs，資料庫並沒有限制。我是以golang寫，資料庫的部份以單線程為主的redis，使得單個transaction不用考慮race condition，加上存取快速高併發的特性，以及方便的ttl對這次的作業十分友善，因此選擇redis作為這次作業的資料庫。至於來自不同client的get和set可能造成race condition，我在[<span style="color:blue">github repo</span>](https://github.com/nathan-tw/gin-rate-limiter)中有詳細說明我如何實做lock。

### 一面

可以提早到hr會帶你參觀辦公室，由兩位backend面試後換hr面，先自我介紹約5-10分鐘，backend會針對以前的專案經驗追根究底的問，一定要對自己做過的專案非常熟悉，不然會和我一樣被問倒XD，加上以前在底層的計算機基礎知識不夠扎實，例如被問到為何mips要分成五個stage的pipeline來設計，當場腦袋空白只好說不知道，結果離開後馬上想到zzz。所以基礎也非常重要，面完就知道不會上了，期待蠻久的面試以失望落幕，果然自己還有許多基礎需要補足。和hr的面試就是一般的behavior問題，這個網路上有許多厲害的技巧分享，我就不多花篇幅了。

### 結果
感謝信

## IBM (Associate developer)

官網投遞校園招募後一星期收到線上趣味測驗，趣味測驗結束就收到感謝信了。

### 結果
感謝信

## Amazon Ring (SWE)

{{< figure src="/images/intern/ring.jpg"  attr="amazon ring">}}


官網投遞校園招募後一星期收到線上測驗。

### Onsite Assignment

總共有三題，分別是easy, hard, medium。只能用四種語言: C, C++, Java, Python，職位的語言偏好也是前面的順序，第一次看到大科技公司對語言有偏好的，由於C-like的語言都不太熟，Java寫起來也不心安，雖然是最低順位的語言但也只能用Python了。

### 答題過程

總共有90分鐘，第一題花了15-20分鐘，以easy來說蠻久的，讓我想花更多時間學好英文，不然閱讀速度太慢蠻虧的。第二題是比較複雜的背包問題，因為太久沒寫DP，花了很多時間還是沒寫出來，剩下20分鐘時放棄轉看第三題。結果第三題也是DP，比較基本的二維DP，看完還蠻有信心做出來，但後來和時間賽跑太緊張，index一直算錯還是沒有完成。雖然還沒收到信，但應該是沒什麼機會。

### 更新後續

後來收到信說，這個職位比較偏向硬體的部份，OA僅能用C/C++完成，由於我是用python，於是HR問我要不要再用C like語言寫一次，但我實在對這樣的職務內容沒興趣，且對這兩個語言相當不熟，因此拒絕了機會。

### 結果
reject

## Garmin (SRE intern)

{{< figure src="/images/intern/garmin.webp" attr="garmin logo">}}

在臉書社團看到的職位，實習期間為7-8月，算了一下和實習結束兵單應該差不多到，如果沒到可以去面更多不同的公司，看了許多關於garmin的薪資調查，以軟體來說應該算還不錯的，所以這個實習的目的除了接觸更多SRE實務經驗，有很大一部份是為了轉正而實習。1111投遞履歷後收到面試通知。

### 面試

總部在汐止，但實習地點在林口，剛好離家不遠，不過當天騎到汐止的路上天色昏暗加上毛毛細雨，一路上看起來都相當荒涼，心中暗自慶幸自己不是在這上班。面試會先請你自我介紹，不會有白板題(其實還蠻期待的)。由於目前的實習也是SRE相關職務，所以講完自己的學經歷後就分享了自己對這職位的想法。我認為SRE對任何東西都需要了解一些，因為你不知道與自己對接的是怎樣的開發團隊，其實講到一點很有趣的是我大學跟風學了ML/AI的皮毛，但越是深入越是感覺無力以及渺小，之後便果斷放棄資料科學家一途。面試時我向面試官提到我有一個優勢是對ML有基礎的了解，剛好他們也有Data的團隊在嘗試DevOps，總覺得是命運使然，面試結束我蠻有信心會上的。
>我相信努力不會騙人，也相信沒有路是白走的

### 結果
Offer Get
## 小節

最後去了Garmin，但因為只是實習而已，所以實習尾聲如果有更多心得應該會更新，雖然是以轉正為目標，但還是會憧憬FANG的生活，無論是作夢或是野心，都會不斷訓練自己朝這個方向努力，尤其會更加強自己的英文><如果文章有幫助到你，歡迎email和我聊聊，或是任何社交軟體私訊我我都會看到～
