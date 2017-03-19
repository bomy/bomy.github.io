---
layout     : post
title      : "微服務架構-淺談溝通方式"
date       : 2017-02-28 00:00:00
author     : "Bomy"
tags       : Microservices API
comments   : true
signature  : true
---
當系統日益狀大，是否有遇過為了升級一個Feature很多系統要一起變更的問題，或者做Request回應過慢的問題，在這裡會先簡單介紹同步和非同步溝通模式。之後探討系統架構，這會關於到系統間的耦合度、維護性和擴充性。再來簡單介紹選擇API技術的考量點。

### 同步(Synchronous)和非同步(Asynchronous)
溝通模式有分為同步(Synchronous)和非同步(Asynchronous)，在這裡先簡單介紹這兩種方式。
* **同步模式**。Client端將訊息送到Server端後，會有阻塞(block)等待Server端處理後的回應，即可以知道事情是否處理完全。
* **非同步模式**。Client端將訊息送到Server端後，不必等待Server端處理後的回應，另可Callback 於Clinet端。當要處理低延遲回應時，是種很好的運作模式。

### 系統架構風格(Orchestration vs Choreography)
介紹同步(Synchronous)和非同步(asynchronous)後，接下來討論系統架構，當建立一個微服務時候，需要思考如何與其他服務做互動，有兩種系統風格Orchestration是仰賴集中管理，依總指揮方式每個流程都一步一步來處理。另一種方格是Choreography，每一個服務自己去做彼此協調處理，不透過總指揮方式。假設有個情境，有個會員註冊個功能，裡面的流程需要建立新會員的紅利、計算吸引會員的行銷費用和發送E-mail。
<div style="text-align:center"><img src="/public/image/orchestration.png" />Orchestration架構</div>
在Orchestration風格中，使用同步的`Request/Response`方式，當Request紅利服務，需要進行等待處理是否完成，才能進行後面的行銷服務和E-mail服務的流程。此優點是可以方便追蹤目前處理到那個階段，缺點是紅利服務系統故障，導致後面的行銷服務和E-mail服務都會無法進行處理。

<div style="text-align:center"><img src="/public/image/choreography.png" />Choreography架構</div>
在Choreography，若改採用`發布/訂閱(Publish/Subscribe)`的非同步設計方式，像是RabbitMQ技術。在這裡當會員註冊Publish事件，若紅利服務、行銷服務和E-mail服務有Subscribe，則會有對應處理。此優點是，如果有其他服務想加入跟會員註冊有關的，只要對仲介者(broker)做Subscribe動作，此服務就可以被觸發，可以降低系統間藕合度。缺點是，較難輕易的了解整個業務流程、有一定的實作難度和不易被測試，因此必須要有良好的系統監控機制。

### 如何選擇和設計適合API
選擇API方法有很多，例如[REST](https://en.wikipedia.org/wiki/Representational_state_transfer)、[SOAP](https://en.wikipedia.org/wiki/SOAP)和[RPC](https://en.wikipedia.org/wiki/Remote_procedure_call)，在選擇技術之前，但有幾項因素必須去考慮的。

*  **易用性**。當Client端要串接你的服務時，是否很容易的被使用，例如設定過於繁雜，這些都需要有一定的維護成本。

*  **避免技術耦合**。例如RPC的Java RMI技術，所產生的Stub就要綁定到Client端，這在做版本升級的時候，Client端也需要同步更新，會造成一些麻煩。

*  **盡量避免變更成本**。如果Server端依需求要增加一個欄位或做其他變更，則Client端應該要盡量去避免相關的變動。

* **傳輸速度**。如果服務間的溝通需要即時性的，那麼通訊協定須要選擇低底層的，例如UDP會比HTTP來的好。

### 結論
如果要考慮降低系統耦合和系統擴充，Choreography架構會優於Orchestration。在API設計考量點，我覺得比較重要的是，在版本升級時候，如何的做到與Client的低耦合。

### 相關連結
* [Orchestration vs Choreography](http://www2.sys-con.com/itsg/virtualcd/webservices/archives/0307/peltz/index.html)
* [RabbitMQ](http://blog.csdn.net/doc_sgl/article/details/50615496)
