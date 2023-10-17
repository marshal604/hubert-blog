+++
title = '整理報表數據時，利用數據回填 Backfilling 來解決程式碼的強耦合'
date = 2023-10-17T23:35:00+08:00
draft = false
featured_image = 'featured_image.png'
description = '從考慮使用 OLAP Table 到 Backfilling Data 的決策過程'
tags = ['Backend', 'Software Design', 'Problem Solving', 'Working Notes']
+++
> 這是關於整理報表時，遇到兩個模組間邏輯會打架的工作雜記
## 故事脈絡
---
最近遇到要把以前的報表新增**加權分數(weighted_score)**的欄位，而報表資料主要是從**使用者評分表 Table** 撈取出來整理的，**加權分數**以前是在其他模組計算但沒有被存放到 Table 中，**使用者評分表**的 Schema 簡化後大概如下

| 欄位名稱    | 資料類型 | 說明                          |
|:-----------|:-------|-------------------------------|
| id         | INT(PK) | 主鍵                          |
| user_id    | INT(FK) | 外鍵                          |
| value      | INT     | 新增/扣除的分數                 |
| params     | JSON    | 放置分數相關的任意資料           |
| confirmed  | BOOLEAN | 已經評分過的資料為 true         |

範例資料

| id | value | user_id | params                               |  confirmed   |
|----|:-----:|:-------:|:------------------------------------:|-------------:|
| 1  | 122   | 12      | { "type": 1, "weighted_score": 456 } | 0            |
| 2  | 19    | 11      | { "type": 2, "weighted_score": 314 } | 1            |

## 處理過程
---
### 建立共用的模組
由於以前的程式碼在計算**加權分數**時是直接計算，而**加權分數**其實會因為不同類型的**type**而有不同的加權計算方式，為了在報表也能使用這套邏輯，所以我們需要先抽出一個模組 Calculator ，慶幸的是這套邏輯已經有寫測試，所以我們可以省掉寫測試的時間直接重構即可，這邊我是使用[工廠模式](https://en.wikipedia.org/wiki/Factory_method_pattern)來處理，後續直接套用模組跑測試無誤就算是完成這個階段，可以到下一個階段進入資料的處理惹。
### 將計算完的資料放進 Table 中
這邊介紹一下會用到的兩個模組，模組 D 是負責進行使用者扣分的模組，而模組 E 是接收到扣分行為時會去**使用者評分表**把扣分的資料寫入進去，有個需要注意的地方是，扣分會先從確認過(confirmed = 1)的分數先扣不夠再從未確認(confirmed = 0)的分數扣，所以有可能會一次扣分行為但記錄兩筆資料到**使用者評分表**...

接著就來說說我的踩坑紀錄了，一開始我不知道模組 E 會有可能出現兩筆紀錄時，我就很傻很天真的使用了
```
# 模組 D
weight_score = Calculator(type, value)
params = { type, weight_score }
E.action(value, params)
```
結果就得到了兩筆相同 params 的資料

| id | value | user_id | params                                 |  confirmed   |
|----|:-----:|:-------:|:--------------------------------------:|-------------:|
| 1  | -88    | 11      | { "type": 1, "weighted_score": -222 } | 0            |
| 2  | -23    | 11      | { "type": 1, "weighted_score": -222 } | 1            |

想當然這個若是在報表出現的話，會造成 weighted_score 相加出來的結果有問題，所以趕緊把 Calculator 放到模組 E 裡面去判斷
```
# 模組 D
params = { type }
E.action(value, params)

# 模組 E
weight_score = Calculator(type, value)
```

這個寫法其實可以符合我的需求，但其實模組 E 是個通用的扣分模組，而 weight_score 其實是在使用模組 D 時才會有的資訊，所以寫在模組 E 其實是增加對外部的倚賴(coupling)，所以我靜下心來思考了幾個方案

1) Workaround 做法，先這樣就好
2) 只存 type ， weight_score 在產報表時再計算
3) 新增一個模組 F 去創建一個專屬報表用的 [OLAP](https://en.wikipedia.org/wiki/Online_analytical_processing) Table
4) 模組 E 回傳扣分後的 **使用者評分表** ids，使用 Backfilling 方式回填進 Table

TBD...

### Workaround 做法，先這樣就好

### 只存 type ， weight_score 在產報表時再計算

### 新增一個模組 F 去創建一個專屬報表用的 OLAP Table

### 模組 E 回傳扣分後的 **使用者評分表** ids，使用 Backfilling 方式回填進 Table

## 小結
---

