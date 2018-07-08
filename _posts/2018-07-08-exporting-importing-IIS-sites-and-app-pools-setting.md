---
layout     : post
title      : "如何將IIS站台設定匯出至新機器"
date       : 2018-07-08 00:00:00
author     : "Bomy"
tags       : IIS 8007000d
comments   : true
signature  : true
---

在新機器要將原有的設定值移植過去，如果要用人工手動方式一個一個設定上去會很耗時且容易出錯，因此可以用指令自動化方式去取代工人的作業。

### 實作方式
Step1. 在原本機器將設定值匯出
```
%windir%\system32\inetsrv\appcmd list apppool /config /xml > c:\apppool.xml
%windir%\system32\inetsrv\appcmd list site /config /xml > c:\website.xml
```
Step2. 把此兩檔案內容Copy出來至新機器

Step3. 進行匯入作業
```
%windir%\system32\inetsrv\appcmd add apppool /in <c:\apppool.xml
%windir%\system32\inetsrv\appcmd add site /in < c:\website.xml
```
### 遇到的問題
在進行`Step3`匯入動作時候，可能會遇到`8007000d`錯誤代碼，主要原因是匯入的XML格式未完全符合這台新機器所要的格式，主要可能是不同IIS版本造成的差異，但這怎辦呢?

*`ERROR ( hresult:8007000d, message:Command execution failed.
The data is invalid.
 )`*

![Alt text](/public/image/IISP1.png)

我的方法是在新機器建立一個Sample站台和apppool後，在透過`Step1`方式把它產出XML檔案。之後開啟比對軟體(beyond compare)，比對新機器跟原本機器所匯出XML檔(apppool.xml, website.xml)的格式差異，觀察出格式差異後，再將原本設定XML檔案進行格式微調，通常是Remove幾個不影響的設定值，微調好之後，重新進行`Step3`動作後，此時就可以解決此問題。



### 相關連結
[Exporting and Importing Sites and App Pools from IIS 7 and 7.5](http://www.microsoftpro.nl/2011/01/27/exporting-and-importing-sites-and-app-pools-from-iis-7-and-7-5/)
