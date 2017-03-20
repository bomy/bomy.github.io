---
layout     : post
title      : "使用IIS解決Google Analytics在大陸被擋的問題"
date       : 2017-03-19 00:00:00
author     : "Bomy"
tags       : Google-Analytics IIS Reverse-Proxy
comments   : true
signature  : true
---
[Google Analytics;GA](https://analytics.google.com/) 是分析流量的工具，透過GA所提供Javascript程式碼，嵌入所要分析網站，就可以將訪客的資料傳送至GA server。

 [在2014年Google服務被大陸封鎖](https://zh.wikipedia.org/wiki/2014%E5%B9%B4%E4%B8%AD%E5%9B%BD%E5%A4%A7%E9%99%86%E5%B1%8F%E8%94%BD%E8%B0%B7%E6%AD%8C%E6%9C%8D%E5%8A%A1%E4%BA%8B%E4%BB%B6)，所以GA在做傳送資料時，在大陸有可能發生網路不穩定的情況，導致有些數據無法收集到。在此透過[卡卡網](http://www.webkaka.com)去檢測GA網路回應情況，藉由淘寶網(https://world.taobao.com/)來做比較。根據下圖顯示，GA在某些省分有出現紅色，可能GA有連線不穩定的情況。

<div style="text-align:center"><img src="/public/image/CheckGAfromWebkaka.png" />GA在大陸連線狀況</div>

### 原理
如果你是ASP .NET架構的網站，可以藉由IIS的反代理(Reverse Proxy)來解決這個問題。如下圖，有兩個重要的地方。
* **需有個代理的Domain**。透過`IIS的Reverse Proxy的功能`，Request 至Google server 擷取analytics.js。
* **傳輸的資料必須做修改**。將`analytics.js`裡的內容，將`www.google-analytics.com`字串 取代為`代理的URL`。
<div style="text-align:center"><img src="/public/image/reverse_proxy.png" />reverse proxy運作方式</div>

### 作法
Step1. 下載 http://www.iis.net/extensions/ApplicationRequestRouting 安裝後，IIS管理工具，會出現這兩個功能，如下圖。
<div style="text-align:center"><img src="/public/image/IIS_Reverse_Proxy.png" /></div>

Step2. 加入 `HTTP_ACCEPT_ENCODING` 設定
<div style="text-align:center"><img src="/public/image/reverse_proxy1.png" /></div>

Step3. 在Web.config加入至<system.webServer>底下，裡面XML有個www.user.com，是隨著你的Domain決定的。

```xml
<rewrite>
  <rules>
    <rule name="Reverse Proxy to analytics" stopProcessing="true">
      <match url="(.*)" />
        <action type="Rewrite" url="https://www.google-analytics.com/analytics.js" />
          <serverVariables><set name="HTTP_ACCEPT_ENCODING" value="false" /></serverVariables>
     </rule>
  </rules>
  <outboundRules>
    <rule name="Outbound CDN" preCondition="IsTextContent">
      <match filterByTags="None" pattern="www.google-analytics.com" />
      <action type="Rewrite" value="www.user.com" />
    </rule>
    <preConditions>
      <preCondition name="IsTextContent">
        <add input="{RESPONSE_CONTENT_TYPE}" pattern="^text/(.+)$" />
      </preCondition>
     </preConditions>       
  </outboundRules>
</rewrite>
```


Step4. 在GA提供的Javascript程式碼，須將https://www.google-analytics.com/analytics.js 修改為http://www.user.com(依你Domain決定)。

``` javascript
<script>
(function(i,s,o,g,r,a,m){i['GoogleAnalyticsObject']=r;i[r]=i[r]||function(){
(i[r].q=i[r].q||[]).push(arguments)},i[r].l=1*new Date();a=s.createElement(o),
m=s.getElementsByTagName(o)[0];a.async=1;a.src=g;m.parentNode.insertBefore(a,m)
})(window,document,'script','http://www.user.com','ga');

ga('create', 'UA-xxxxx-1', 'auto');
ga('send', 'pageview');

</script>
```
### 相關連結
* [iis7.5做反向代理配置方法實例圖文教程](http://www.cnblogs.com/pengcc/p/4329207.html)
* [A reverse proxy with IIS and URL Rewrite](http://blog.cellenza.com/archi-patterns-bp/reverse-proxy-iis-url-rewrite/)
* [卡卡網](http://www.webkaka.com/)
