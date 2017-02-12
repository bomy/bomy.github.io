---
layout     : post
title      : "微服務架構-如何劃分微服務"
date       : 2017-02-11 00:00:00
author     : "Bomy"
tags       : Microservices
comments   : true
signature  : true
---

### 怎樣才是好的微服務
剛認識Microservice時，會誤認為Microservice就是一個輕量化的服務，可以依程式碼的Size去定義服務的顆粒大小。然而，正確的方式應掌握在`鬆綁耦合(loose coupling)`和`高度內聚(high cohesion)`的原則。

 * **鬆綁耦合:** 一個服務與其他服務之間關係應該越小越好，即變更一個服務不會去影響另一個服務。如果服務間呼叫方式過於複雜，可能造成緊密耦合(tight coupling)。

 * **高度內聚:** 相同的Business應該聚集再一起，不相關的須分處在不同的地方。例如要改變某功能的需求，最佳狀況是變更範圍在同一個地方，而不是在多個服務做修改，且在多個地方一起做Deploy時，風險也會提高。

### 如何劃分微服務的顆粒大小?
在領域驅動設計([Domain-Driven Design](https://en.wikipedia.org/wiki/Domain-driven_design)；DDD)，Bounded Context對於微服務劃分是很重要的概念，藉由封裝此Context的範圍例如Know how、DevOps流程、應用服務和資料模型等等，有自己的`Ubiquitous Language`。目的是確保不會被其他Context所影響，且擁有領域的自主性。然而，每一個Context都具有`Explicit interface`，決定了與其他Context的交互作用。

<div style="text-align:center"><img src="/public/image/Bounded-Context.png" /></div>

在圖中，我們看到將存貨部門和銷售部門畫分兩個獨立`Bounded Context`，各自封裝了自己領域細節流程，彼此不需要了解各自內部細節，像是進貨處理，銷售部門就不需要了解。此外，彼此也具有開放給外面使用Interface，例如在銷售部門訂單是否成立，就需要存貨部門的庫存數量資訊。

因此，清楚那些是該開放、那些是隱藏的模型，可以鬆綁彼此系統間的耦合，減少不必要的連動關係。相同業務聚劃分在一起，則提高高內聚力所帶來的好處。

### 結論
透過DDD的Bounded Context方式來劃分Microservice適合的邊界，可帶來鬆綁耦合和高度內聚的好處，避免了整體系統的混沌不明。

### 相關連結

* [Small Bounded Contexts Over One Comprehensive Model](http://blog.xebia.com/microservices-architecture-principle-3-small-bounded-contexts-over-one-comprehensive-model)
* [BoundedContext](https://martinfowler.com/bliki/BoundedContext.html)
* [Using Domain-Driven Design When Creating Microservices](https://www.infoq.com/news/2016/02/ddd-microservices)
