---
title: javascript类数组(ArrayLike)转化成数组
date: 2017-10-18T21:58:27.000Z
tags: javascript
categories: javascript 
---

# javascript类数组转化成数组

## 什么是类数组（ArrayLike）？

ArrayLike（类数组/伪数组）即拥有数组的一部分行为,例如 arguments, NodeList等！他们拥有**_length_**属性，但是却不能用一些数组的方法，如 push，pop等等

常见的ArrayLike有下面的这几个：

- Arguments
- NodeList
- StyleSheetList
- HTMLCollection
- HTMLFormControlsCollection (继承HTMLCollection)
- HTMLOptionsCollection(继承HTMLCollection)
- HTMLAllCollection
- DOMTokenList

## Array-Like to Array

在项目开发中，经常遇到需要把 Array-Like Objects 转为 Array 类型，使之能用数组的一些方法。下面列出几种方法：

* for循环

```javascript
  function fn() {
    var arr = [];
    for (var i = 0, len = arguments.length; i < len; i++)
    arr[i] = arguments[i];
  }

  fn(1, 2, 3);
```

* Array.prototype.slice.call(array-like object)

```javascript

function fn() {
  var arr = Array.prototype.slice.call(arguments);
}

fn(1, 2, 3);
```

或者可以用 **_[]_** 代替 Array.prototype

```javascript

function fn() {
  var arr = [].slice.call(arguments);
}

fn(1, 2, 3);

```

原理： slice的内部实现

```javascript
//slice的内部实现
Array.prototype.slice = function(start,end){  
      var result = new Array();  
      start = start || 0;  
      end = end || this.length; //this指向调用的对象，当用了call后，能够改变this的指向，也就是指向传进来的对象，这是关键  
      for(var i = start; i < end; i++){  
           result.push(this[i]);  
      }  
      return result;  
 }
```

兼容 IE8 以下的浏览器

```javascript
var toArray = function(s){  
    try{  
        return Array.prototype.slice.call(s);  
    } catch(e){  
            var arr = [];  
            for(var i = 0,len = s.length; i < len; i++){  
                //arr.push(s[i]);  
                 arr[i] = s[i];     //据说这样比push快
            }  
             return arr;  
    }
}
```

* Array.from()

```javascript
var str = "helloworld";
var arr = Array.from(str);
```

### Array-Like to Array 注意点

arguments 转换成数组的时候经常会把 ***Array.prototype.slice.call(arguments)*** 用更短的写法 *** [].slice.call(arguments)***

这个将导致Chrome和Node中使用的V8引擎跳过对其的优化，使其性能相当慢。
