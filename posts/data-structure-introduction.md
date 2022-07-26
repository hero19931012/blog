---
title: 資料結構筆記01 什麼是資料結構
description: data structure notes
date: 2022-07-26
scheduled: 2022-07-26
tags: data structure
layout: layouts/post.njk
---
## 定義

> 整理資料的方式。
> 如何在電腦中存放資料。

## 基本特性

假設不同的資料結構就像是抽屣、衣櫥或冰箱等儲存空間，這個空間最主要的功能就是存放資料，所以一定都會有 `INSERT` (放資料進去) 與 `GET` (讀資料出來，不是刪除) 這 2 個方法。

### 其他的存取方法

| Method      | Description |
| ----------- |:-----------:|
| `CONSTRUCT` |  如何建構   |
| `REMOVE`    |  刪除資料   |
| `UPDATE`    |  更新資料   |

## 為什麼我們需要資料結構

> 何謂好的程式？以最少資源完成預期的任務。

當程式執行時，所需要的資源可大致分成 Space resource (空間), Computation resource (運算) 與其他類型的資源，其中資料結構主要影響空間上的資源，演算法主要影響運算上所需的資源 。

- `Space`: memory, disk, transmission bandwidth => cared by **Data structure**
- `Computation`: CPU, GPU, computation power => cared by **Algorithm** 
- `Other`: manpower, budget => cared by management

## 實作資料結構的 trade off

沒有完美的資料結構可以適用所有情況，只有當下適合用的結構，選擇不同結構的同時往往都有所謂的 trade off，可能有下面幾種：

- fast get <=> slow insert: 取出來較快，但寫入較慢
- fast get <=> more space: 取出來較快，但空間需求較大
- 較難的實作 <=> 較好的效能

在實作之前還是要思考**值不值得**，如果非常小的專案或是很少人用的服務，其實根本不需要花這麼大的力氣去提升效能。

## Reference

[NTU DSA 2022: Course Introduction / Algorithm](https://www.youtube.com/watch?v=upMnTq9JB2c)