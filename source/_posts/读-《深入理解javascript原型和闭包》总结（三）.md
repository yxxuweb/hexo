---
title: 读 《深入理解javascript原型和闭包》总结（三）
date: 2018-04-06 10:58:44
tags: javascript
categories: javascript
---

### 执行上下文栈

执行全局代码时，会产生一个执行上下文环境，每次调用函数都又会产生执行上下文环境。当函数调用完成时，这个上下文环境以及其中的数据都会被消除，再重新回到全局上下文环境。处于活动状态的执行上下文环境只有一个

***其实这是一个压栈出栈的过程——执行上下文栈***

![](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/232122300768665.png?raw=true)

有一种情况，而且是很常用的一种情况，无法做到这样干净利落的说销毁就销毁。这种情况就是伟大的——闭包。
要说闭包，咱们还得先从自由变量和作用域说起。

<!-- more -->

### 简介【作用域】

“javascript没有块级作用域”。所谓“块”，就是大括号“｛｝”中间的语句。

***javascript除了全局作用域之外，只有函数可以创建的作用域。***

***我们在声明变量时，全局代码要在代码前端声明，函数中要在函数体一开始就声明好。除了这两个地方，其他地方都不要出现变量声明。而且建议用“单var”形式***

![](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/241708372951952.png?raw=true)

全局代码和fn、bar两个函数都会形成一个作用域。而且，***作用域有上下级的关系，上下级关系的确定就看函数是在哪个作用域下创建的***。例如，fn作用域下创建了bar函数，那么“fn作用域”就是“bar作用域”的上级


***作用域最大的用处就是隔离变量，不同作用域下同名变量不会有冲突***

### 【作用域】和【上下文环境】

除了全局作用域之外，每个函数都会创建自己的作用域，***作用域在函数定义时就已经确定了。而不是在函数调用时确定。***

第一步，在加载程序时，已经确定了全局上下文环境，并随着程序的执行而对变量就行赋值。

![](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/250814158269779.png?raw=true)

第二步，程序执行到第27行，调用fn(10)，此时生成此次调用fn函数时的上下文环境，压栈，并将此上下文环境设置为活动状态

![](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/250814386853995.png?raw=true)

第三步，执行到第23行时，调用bar(100)，生成此次调用的上下文环境，压栈，并设置为活动状态。

![](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/250815006238997.png?raw=true)

第四步，执行完第23行，bar(100)调用完成。则bar(100)上下文环境被销毁。接着执行第24行，调用bar(200)，则又生成bar(200)的上下文环境，压栈，设置为活动状态

![](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/250815248579200.png?raw=true)

第五步，执行完第24行，则bar(200)调用结束，其上下文环境被销毁。此时会回到fn(10)上下文环境，变为活动状态。

![](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/250815435609914.png?raw=true)

第六步，执行完第27行代码，fn(10)执行完成之后，fn(10)上下文环境被销毁，全局上下文环境又回到活动状态。

![](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/250816112012394.png?raw=true)

最后我们可以把以上这几个图片连接起来看看。

![](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/250816269984619.png?raw=true)


***作用域只是一个“地盘”，一个抽象的概念，其中没有变量。要通过作用域对应的执行上下文环境来获取变量的值***。同一个作用域下，不同的调用会产生不同的执行上下文环境，继而产生不同的变量的值。所以，***作用域中变量的值是在执行过程中产生的确定的，而作用域却是在函数创建时就确定了***。

***如果要查找一个作用域下某个变量的值，就需要找到这个作用域对应的执行上下文环境，再在其中寻找变量的值。***

### 自由变量

自由变量： 在A作用域中使用的变量x，却没有在A作用域中声明（即在其他作用域中声明的），对于A作用域来说，x就是一个自由变量

```javascript
var x = 10;

function fn() {
  var b = 20;
  console.log(x + b); // 这里的 X 就是一个自由变量
}
```


```javascript
var x = 10;
function fn () {
  console.log(x);
}

function show(f) {
  var x = 20;
  (function () {
    fn();   // 10 , 而不是 20
  })()
}

show(fn);
```

***自由变量要到创建这个函数的那个作用域中取值——是“创建”，而不是“调用”，切记切记*** --- 这就是所谓的“静态作用域”

作用域的过程

第一步，现在当前作用域查找a，如果有则获取并结束。如果没有则继续；

第二步，如果当前作用域是全局作用域，则证明a未定义，结束；否则继续；

第三步，（不是全局作用域，那就是函数作用域）将创建该函数的作用域作为当前作用域；
第四步，跳转到第一步。

### 闭包

***闭包是由函数以及创建该函数的词法环境组合而成。这个环境包含了这个闭包创建时所能访问的所有局部变量。***

***你只需要知道闭包应用的两种情况即可——函数作为返回值，函数作为参数传递***

第一，函数作为返回值

```javascript
function fn() {
  var max = 10
  return function bar (x) {
    if (x > max) {
      console.log(x);
    }
  }
}

var f1 = fn()

f1(15)
```

第二，函数作为参数被传递

```javascript
var max = 10,
    fn = function (x) {
      if (x > max) {
        console.log(x);
      }
    };

(function (f) {
  var max = 100;
  f(15);
})(fn);
```

函数调用完成之后，其执行上下文环境不会接着被销毁。这就是需要理解闭包的核心内容。

演示例子：

第一步，代码执行前生成全局上下文环境，并在执行时对其中的变量进行赋值。此时全局上下文环境是活动状态。

![](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/260749349988764.png?raw=true)


第二步，执行第17行代码时，调用fn()，产生fn()执行上下文环境，压栈，并设置为活动状态

![](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/260750319351092.png?raw=true)

第三步，执行完第17行，fn()调用完成。按理说应该销毁掉fn()的执行上下文环境，但是这里不能这么做。注意，重点来了：因为执行fn()时，返回的是一个函数。函数的特别之处在于可以创建一个独立的作用域。而正巧合的是，返回的这个函数体中，还有一个自由变量max要引用fn作用域下的fn()上下文环境中的max。因此，这个max不能被销毁，销毁了之后bar函数中的max就找不到值了。

因此，这里的fn()上下文环境不能被销毁，还依然存在与执行上下文栈中。

——即，执行到第18行时，全局上下文环境将变为活动状态，但是fn()上下文环境依然会在执行上下文栈中。另外，执行完第18行，全局上下文环境中的max被赋值为100。如下图：

![](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/260957500455644.png?raw=true)

第四步，执行到第20行，执行f1(15)，即执行bar(15)，创建bar(15)上下文环境，并将其设置为活动状态。

![](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/260958057327369.png?raw=true)

执行bar(15)时，max是自由变量，需要向创建bar函数的作用域中查找，找到了max的值为10。这个过程在作用域链一节已经讲过。

这里的重点就在于，创建bar函数是在执行fn()时创建的。fn()早就执行结束了，但是fn()执行上下文环境还存在与栈中，因此bar(15)时，max可以查找到。如果fn()上下文环境销毁了，那么max就找不到了。


第五步，执行完20行就是上下文环境的销毁过程
