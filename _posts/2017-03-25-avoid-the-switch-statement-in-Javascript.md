---
layout     : post
title      : "避免使用Switch..Case語法"
date       : 2017-03-26 00:00:00
author     : "Bomy"
tags       : Javascript
comments   : true
signature  : true
---
之前某個前輩說`Switch..Case`不是一個很好的撰寫方法，例如底下的程式碼，有幾個缺點。

 * 在撰寫Switch..case時候，會不小心忽略`Break`語句，會導致很難找到BUG。
 * Switch..case有點像是麵條式的寫法，跟Goto語法很像，會將不相關的功能寫在一起，違反了High cohesion。
 * 無法在動態執行過程中，可隨意增加或減少或置換項目，例如在這範例Switch..case裡面的"Add"、"Subtract"..，在某種情況下不須要當中"Add"這個項目。
 * 缺乏組合與重複使用的特性。
``` javascript
function doCalculation(action,a,b) {
  switch (action) {
    case 'Add':
      return a + b;
      break;
    case 'Subtract':
      return a - b;
      break;
    case 'Divide':
      return a / b;
      break;
    default:
      throw new Error("Action not found.");
  }
}
console.log(doCalculation("Add",2,3)); //5
```
基於上述原因，我們需要比較有彈性的設計方法。那個如果去設計呢?可以使用Javascript中較優雅的`Object-literal`和`Function`語法來設計，例如底下的範例。

``` javascript
function doCalculation(action,a,b) {
  var actions = {
    Add: function(x, y) {
      return x + y;
    },
    Subtract: function(x, y) {
      return x - y;
    },
    Divide: function(x, y) {
      return x / y;
    }
  };

  if(typeof actions[action] !== 'function'){
    throw new Error("Action not found.");
  }

  return actions[action](a,b);
}
console.log(doCalculation("Add",2,3)); //5
```
假設在動態執行過程裡面，需要隨意的增加項目，我們可以這樣設計，例如底下範例增加`Multiply`，或者可移除`Add`項目，使得更有彈性。

``` javascript
function doCalculation(action,a,b) {
  var actions = {
    Add: function(x, y) {
      return x + y;
    },
    Subtract: function(x, y) {
      return x - y;
    },
    Divide: function(x, y) {
      return x / y;
    },
  };

  //增加項目
  var Multiply =  function(x, y) {
      return x * y;
  };
  actions['Multiply'] = Multiply;

  //移除項目
  delete actions['Add'];

  if(typeof actions[action] !== 'function'){
    throw new Error("Action not found.");
  }
  return actions[action](a,b);
}

console.log(doCalculation("Multiply",2,3)); //6
console.log(doCalculation("Add",2,3)); // Action not found.
```
如果遇到需要群組的使用案例，例如底下範例。
``` javascript
function doCalculation(action,a,b) {
  switch (action) {
    case 'Add':
    case '+':
      return a + b;
      break;
    case 'Subtract':
    case '-':  
      return a - b;
      break;
    default:
      throw new Error("Action not found.");
  }
}
```

可以改用較有彈性的設計方式，例如底下範例。
``` javascript
function doCalculation(action,a,b) {
  var Add =  function(x, y) {
      return x  + y;
  };
  var Subtract =  function(x, y) {
      return x  - y;
  };
  var actions = {
    'Add':Add,
    '+':Add,
    'Subtract':Subtract,
    '-':Subtract
  };

  if(typeof actions[action] !== 'function'){
    throw new Error("Action not found.");
  }
  return actions[action](a,b);
}

console.log(doCalculation("Add",2,3)); //5.
```
### 結論
雖然Object-literal設計的方式沒有Switch..Case來的直覺，至少不用在寫Break了(笑)。但它可以應用多種情況，例如Command的一些操作，隨時都可以根據情況置換(委派)出想要的功能性組合。

另外在效能方面，根據[這篇](https://toddmotto.com/deprecating-the-switch-statement-for-object-literals/)指出，Object-literal搜尋是透過Hash Table效率約為O(1)，而Switch...Case的效率跟排序有關係約為O(logN~N)，理論上Object效率會比Switch Case好，但是我看到[這篇](https://jsperf.com/switch-from-switches/)的測試結果，Switch...Case竟比Object-literal來的有效率，真是匪夷所思，不知道是不是在新的瀏覽器版本對Switch..Case有進行優化過，

### 相關連結
* [Replacing switch statements with Object literals](https://toddmotto.com/deprecating-the-switch-statement-for-object-literals/)
* [Jsperf](https://jsperf.com)
