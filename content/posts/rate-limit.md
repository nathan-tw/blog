---
title: "[技術] 各類Rate limit設計"
date: 2022-04-20T16:52:44+08:00
draft: true
categories: ["programming"]
---

## 來自實習的反省

因為工作的需求，需要設計一套rate limit的機制，讓我想起自己曾經在應徵Dcard後端實習時，根據他們的要求設計了一個類似要求Middleware。現在回頭看發現那時果然太年輕了，許多地方都考慮欠周，趁著這次多方研讀，紀錄一下不同的rate limit機制。

## Rate limit的目的？

限制過於頻繁的請求，保證服務的穩定性

## 何謂良好的Rate limit

1. 不會拖垮原服務的效能
2. 分散式架構下資料的一致性
3. 請求來源的識別能力

不難直覺的想到，如果用key-value類型的資料庫剛好滿足以上對資料庫的需求，如果剛好有一個這樣的單線程資料庫就太棒了！redis很自然的浮現在我們腦海中，但單線程真的就沒問題了嗎？


## Race condition

其實rate limit對資料庫來說是一個transaction，為了判斷是否超過限制，我們必須get，而此次的請求則是另一個對資料庫set操作，所以問題出在兩個請求接連對同一資源做get時會有其中一者在另一者之後set，但其實是一直判斷錯誤。

## Rate limit演算法

1. Fixed window
2. Sliding window



