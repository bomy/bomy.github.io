---
layout     : post
title      : "微服務架構-拆分單體式系統"
date       : 2017-07-07 00:00:00
author     : "Bomy"
tags       : Microservices
comments   : true
signature  : true
---
單體式系統會隨著增加需求，使得程式碼變得更肥大，在之前文章有提到如何劃分微服務來達到`鬆綁耦合`和`高度內聚`目的，這裡說明如何對單體式系統進行瘦身，在不同服務間過於耦合的問題，因架構的改變會造成大量的破壞，如何以最小衝擊風險前提下進行解偶，演變成微服務。

<div style="text-align:center"><img src="/public/image/Microservices_from_a_monolith.png" /></div>

## 如何看到高耦合問題
假設我們識別好[每個Service邊界上下文](http://bomy.github.io/2017/02/11/Microservices-Bounded-Context/)，那如何進行系統重構呢？首先可以使用`[SchnaSpy](http://schemaspy.sourceforge.net/sample/relationships.html/)`工具，可觀察資料庫的資料表之間的關係，最大的好處是以圖形介面呈現，可以清楚使我們看到資料間的耦合關係，在這裡你可能會看到幾種資料表間耦合的情況。


#### **資料表間耦合關係**
在這個範例中Sale程序使用Order資料表來儲存客戶交易，Finance程序使用Ledger資料表追蹤財務狀況。假設在Finance程序為了產生報表可能需要一些Order資料表的交易資料，依據下圖顯示，圖中可以看到有兩個需要解偶的地方，第一個是Finance程序不能夠直接擷取Order資料表的資料，只能由Sale本身服務去擷取Order的資料，在這裡最好的做法是，在Sale服務建立一個開放API的窗口，供Finance服務去呼叫Sale服務的API來擷取資料。

![Alt text](/public/image/Foreign_Key_Examples.png)

第二個解偶點是，就是要移除外來鍵，移除外來鍵的缺點可能會造成找不到資料的參考點，例如這個ID的資料被刪除了，會造成找不到資料的問題。如果要避免這情況的發生，最好方式是實作一個跨服務的一致性檢查機制，或者去執行一些可能措施的動作，完全取決於功能需求。

![Alt text](/public/image/Remove-foreign_Key_Examples.png)
據上圖是移除此兩個耦合的結果。然而跨服務的存取，效能的議題也會跟這發生，例如網路的Latency，有時候必須去評估說到底什麼是我們要的東西，對於速度的容忍是否能夠去接受，有時候犧牲一點效能換取一些效益有時候也是很好的

#### **共用的動態資料**
假設Sale程序需要對Inventory資料表進行變更庫存品項的數量，或者查看目前可賣的數量，另外Finance程需要庫存盤點會一樣會做存取，意味著會共用到動態的`Inventory資料表`，如下圖右邊說明。可以看到有些耦合的關系，我們必須把Inventory獨立出來，換句話說，少了一個Inventory Domian邊界上下文，假設有了Inventory Service，就可以利用API供其他服務進行存取。
![Alt text](/public/image/shardrecord.png)

####  **共用的資料表**
另外一種情況是，不同的關注點都放在同一張資料表裡面，例如Sale程序會使用到商品的價格和資訊，Finance程序會使用到商品的成本。而這兩個是不一樣的一個概念，可以將原始的資料表切割成兩個資料表，例如Item sale info 和Item cost info資料表，讓各個服務能夠獨立自主。

![Alt text](/public/image/split_table.png)

## 資料庫如何拆分
在系統持續運行下，資料庫的重構是很有風險的變革，但可以參考[Refactoring Databases]( https://martinfowler.com/books/refactoringDatabases.html)書籍，裡面有詳細的說明。

在定義好每個Service邊界上下文後，可能須將從Monolithic Schema拆分出各個微服務，的雍有自已獨立的資料庫，而為了確保系統的運作，依照下圖，會有兩個階段，第一個階段是先拆分資料庫，第二個階段是拆分程式碼(微服務)，拆分的過程會遇到一個很重要的問題，就是會失去`資料一致性`，在分散式
交易，如果`交易資料(Transaction Data)`失敗了，需要去思考是否需要Roll back或者有個補救失敗的機制。
![Alt text](/public/image/Microservices_splite_database.png)

## 結論
分解系統大多時候是困難的，且充滿風險例如銀行的系統。為了減輕重構的失敗風險，作法盡可能的逐漸調整，以每次最小成本進行變動，風險會比較小。另外還需要上面組織的高度支持，因為這需要勇氣去執行和別互相指責錯誤。

### 相關連結
[建構微服務：設計細微化的系統](http://www.books.com.tw/products/0010719805)
