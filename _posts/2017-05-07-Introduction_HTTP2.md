---
layout     : post
title      : "淺談HTTP2"
date       : 2017-05-07 00:00:00
author     : "Bomy"
tags       : HTTP2
comments   : true
signature  : true
---
### 目前網站(HTTP1.1)效能瓶頸
據[HTTP Archive](http://httparchive.org/trends.php)統計前1000TOP網站裡，如下圖說明，從2012年的網站平均Request次數從87次，到2017年來到143次，可以看到Request次數逐年在增加，所以每建立一次TCP Connection，都會經過DNS lookup、TCP handshake和TLS協議(如果需要)處理，這些動作對於HTTP1.1協定的網頁資料傳輸是很大的開銷。PS:雖然HTTP1.1有Keep-alive來改善交握，但多條Connection 對於Server來說負擔不小。

<div style="text-align:center"><img src="/public/image/TotalTransfer_Requests.png" />從2012/1/1到2017/4/15的統計</div>

##### 那麼加大網路頻寬有沒有效?
在[Mike Belshe's blog](https://www.belshe.com/2010/05/24/more-bandwidth-doesnt-matter-much/)報告指出，如下圖參考自[hpbn.co](https://hpbn.co/primer-on-web-performance/#latency-as-a-performance-bottleneck)，如果Client端網路頻寬由1 Mbps增加至2 Mpbs，網頁的載入時間(Page Load Time)會縮減一半，可是如果提升至頻寬5 Mpbs開始到10 Mpbs對於網頁的載入時間並沒有很有效的改善。我看到這研究很滿訝異的，顛覆我過去的印像，原來努力的頻寬是沒有用的，不過頻寬大一點對於較大的檔案或著影音，在User體驗還是比較好的。

另外作者還有個實驗，如果網路的延遲(Latency)的減少，對於網頁反應時間就會越快，是線性成反比的效果。
<div style="text-align:center"><img src="/public/image/page-load-bandwidth.svg" />Page load time vs. bandwidth and latency</div>

那麼為什麼對於網路頻寬增加沒有顯著的效果? 前一段有提到，主要問題是大多HTTP的Request次數不少，回來的資料都是微小的，Header資料重複傳送。有時為了讓回應速度變快，Browser會去建立數條connection數目(最多六條)，Connection的初始建立去做 DNS lookup、TCP handshake和TLS協議(如果需要)，同樣會重複的去處理。特別在Web Mobile的使用情況下，可能因位置移動，它們的重新建立Connection機率就會變高。


以目前網站主要優化方式，像是將JS、CSS和ICON壓縮合併成一個單個檔案，或者使用Cache方式，無非就是減少Networking往返次數，來增加網頁回應速度。

### HTTP2.0的出現
HTTP2.0是建立在SPDY基礎上，解決一些HTTP1.1存在的問題，當然包含上述的問題，在這裡簡單介紹HTTP2.0的特色。

* 載入時間變快。採用了multiplexing技術，因共享同條TCP，不必在多開Connection，進而增加載入速度，據[chromium.org](https://blog.chromium.org/2009/11/2x-faster-web.html)可提升55％的Page Load Time。但如果把一些靜態檔放在不同Domain，那麼就要多開Connect去請求，所以如果要優化，就要盡可能放在同個Domain。

* Server Push特點。如下圖，例如Client端發出Index.html，那麼Server端會連其他的CSS和Image一起預先Push回給Client端，先放進Cache裡面，那麼Client不用再次發出Request，就可以取用，這個有點像是預先載入的概念。
<div style="text-align:center"><img src="/public/image/HTTP2_Server_Push.png" /></div>

* 可控制Reponse回來優先權。例如依序Request送出A, B, C, D封包，在HTTP1.1 的架構下會依序A, B, C, D接收回來，然而在HTTP2.0的版本裡，可控制D封包比A, B, C早優先接收回來。

* Header 壓縮(HPACK)。在HTTP1.1版本是沒有做Header壓縮，在HTTP2.0中是使用HPACK規範作為 header壓縮，因此更能提高性能和安全性。

### 比較HTTP1.X和HTTP2.0

| HTTP1.x | HTTP2 |
| ---------- | --- |
| Client可能發個多個TCP connection至Server 端 | Client和Server端共用一個TCP connection至Server|
| 沒有採用Header壓縮| 使用HPACK壓縮方式，提高效能跟安全 |
|較慢的壓縮方式| 較快的壓縮方式  |
|沒有支援回應優先權| 可控制回應優先權  |

### ASP.NET和Broser支援性
以目前微軟生態圈，需要ASP.NET 4.6、IIS10和Windows Server 2016或者 windows 10以上作業系統的才能夠支援HTTP2，老實說這規格版本太新了，對於要升級上去的企業要不少$$。

在Broser方面，IE Broser至少要到11版本才有部分支援(嘆氣)，如果要全面支援，就要使用Edge Broser
<div style="text-align:center"><img src="/public/image/http2Support.png" /></div>

### 結論
在這裡大約介紹HTTP2的特點，有很多技術沒有去提到(如果有機會，我在寫上來)，不過HTTP2對於HTTP1.X有很大的變革，為了帶來更好的使用者體驗，在未來一定是要升級上去的。另外，或許有些HTTP1.X架構網站的優化技巧，像是合併壓縮方式，到了HTTP2.0架構，就可能效果就沒有那麼顯著了(時代的眼淚)，

###相關連結
* [hpbn.co](https://hpbn.co/primer-on-web-performance/#latency-as-a-performance-bottleneck)
* [Creating HTTP/2 Supported Website With ASP.NET Core and Hosting In Windows 10 (IIS 10.0.x)](http://www.c-sharpcorner.com/UploadFile/efa3cf/creating-http2-supported-website-with-Asp-Net-core-hostin/)
* [WHAT IS HTTP/2: AN INTRODUCTION](https://kinsta.com/learn/what-is-http2/)
* [caniuse.com](http://caniuse.com/#search=HTTP2)
