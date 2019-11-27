---
title: 读 《深入理解javascript原型和闭包》总结（二）
date: 2018-04-06 09:43:11
tags: javascript
categories: javascript
---

### 继承

JavaScript 中的继承是通过原型链来体现的。

```javascript
function Foo() {}
var f1 = new Foo();

f1.a = 10;

Foo.prototype.a = 100;
Foo.prototype.b = 200;

console.log(f1.a); //10
console.log(f1.b); // 200
```

f1是Foo函数new出来的对象，f1.a是f1对象的基本属性，f1.b是怎么来的呢？——从Foo.prototype得来，因为`f1.__proto__`指向的是`Foo.prototype`

***访问一个对象的属性时，先在基本属性中查找，如果没有，再沿着`__proto__`这条链向上找，这就是原型链***

***如何区分一个属性到底是基本的还是从原型中找到的 ---- hasOwnProperty***

由于所有的对象的原型链都会找到Object.prototype，因此所有的对象都会有Object.prototype的方法。这就是所谓的“继承”。

每个函数都有call，apply方法，都有length，arguments，caller等属性。函数由Function函数创建，因此继承的Function.prototype中的方法。Function.prototype继承自Object.prototype的方法，`Function.prototype.__proto__`指向Object.prototype

JavaScript对象属性是可以随时改动的，如果你要添加内置方法的原型属性，最好做一步判断，如果该属性不存在，则添加。如果本来就存在，就没必要再添加了。

<!-- more -->

### 执行上下文（一）

在一段js代码拿过来真正一句一句运行之前，浏览器已经做了一些“准备工作”，其中就包括对变量的声明，而不是赋值。变量赋值是在赋值语句执行的时候进行的，

第一种情况
![验证](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/221744084828533.png?raw=true)

第二种情况
![](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/221744319354566.png?raw=true)

第三种情况，
![](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/221745066078791.png?raw=true)

第一种情况：只是对变量进行声明（并没有赋值）。

第二种情况：直接给this赋值。这也是“准备工作”情况要做的事情之一

第三种情况：需要注意代码注释中的两个名词——“函数表达式”和“函数声明”。虽然两者都很常用，但是这两者在“准备工作”时，却是两种待遇。“函数声明”时我们看到了第二种情况的影子，而“函数表达式”时我们看到了第一种情况的影子。

在准备工作中完成了哪些任务（***重点***）
***
* 变量、函数表达式——变量声明，默认赋值为undefined；
* this——赋值；
* 函数声明——赋值
***

***这三种数据的准备情况我们称之为“执行上下文”或者“执行上下文环境”。***

其实，javascript在执行一个代码段之前，都会进行这些“准备工作”来生成执行上下文。***这个“代码段”其实分三种情况——全局代码，函数体，eval代码。***

所谓“代码段”就是一段文本形式的代码

1. 全局代码是一种
2. eval代码接受的也是一段文本形式的代码
3. 函数体是代码段是因为函数在创建时，本质上是 new Function(…) 得来的，其中需要传入一个文本形式的参数作为函数体。

![](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/221746370927602.png?raw=true)

![](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/221747371078703.png?raw=true)

![](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/221746583578531.png?raw=true)

### 执行上下文（下）

执行上下文环境中有如何数据：

* 变量、函数表达式——变量声明，默认赋值为undefined；
* this——赋值；
* 函数声明——赋值；

如果在函数中，除了上面的数据之外，还会有其他数据。

```JavaScript
function fn(x) {
  console.log(arguments);
  console.log(x);
}
fn(10)
```

以上代码展示了在函数体的语句执行之前，arguments变量和函数的参数都已经被赋值。从这里可以看出，***函数每被调用一次，都会产生一个新的执行上下文环境。*** 因为不同的调用可能就会有不同的参数。

另外一点不同在于，***函数在定义的时候（不是调用的时候），就已经确定了函数体内部自由变量的作用域***

总结：全局代码的上下文环境数据内容为：

|||
|-|-|
|普通变量（包括函数表达式），如:var a=10;|声明(默认赋值为undefined)|
|函数声明,如:function fn(){}|赋值|
|this|赋值|

如果代码段是函数体，在此基础上需要附加：

|||
|-|-|
|参数|赋值|
|arguments|赋值|
|自由变量的取值作用域|赋值|


给执行上下文环境下一个通俗的定义——***在执行代码之前，把将要用到的所有的变量都事先拿出来，有的直接赋值了，有的先用undefined占个空。***

### this

***在函数中this到底取何值，是在函数真正被调用执行的时候确定的，函数定义的时候确定不了***
因为this的取值是执行上下文环境的一部分，每次调用函数，都会产生一个新的执行上下文环境。

###### 情况1： 构造函数

构造函数就是用new对象的函数。所有的函数都可以new一个对象，但是有些函数的定义是为了new一个对象，而有些函数则不是。另外注意，构造函数的函数名第一个字母大写（规则约定）。

```javascript
function Foo() {
  this.name = 'xxx';
  this.year = 1991;
  console.log(this);
}

var f1 = new Foo();
console.log(f1.name);
console.log(f1.year);
```
以上代码可以看出，***如果函数作为构造函数用，那么其中的this就代表它即将new出来的对象***

注意： 上面的情况仅限 new Foo() 的情况，即Foo函数作为构造函数的情况。

###### 情况2：函数作为对象的一个属性

如果函数作为对象的一个属性时，并且作为对象的一个属性被调用时，函数中的this指向该对象

```javascript
var obj = {
  x: 10,
  fn: function () {
    console.log(this);
    console.log(this.x);
  }
};

obj.fn();
```

以上代码中，fn不仅作为一个对象的一个属性，而且的确是作为对象的一个属性被调用。结果this就是obj对象

###### 情况3：函数用call或者apply调用

当一个函数被call和apply调用时，this的值就取传入的对象的值。

```javascript
var obj = {
  x: 10
}

var fn = function () {
  console.log(this);
  console.log(this.x);
}

fn.call(obj); // Object {x: 10}  10

```

###### 情况4：全局 & 调用普通函数

在全局环境下，this永远是window，这个应该没有非议。

普通函数在调用时，其中的this也都是window

下面的情况需要注意下：
```javascript
var obj = {
  x: 10,
  fn: function() {
    function f() {
      console.log(this);
      console.log(this.x);
    }
    f()
  }
}

obj.fn();
```

函数f虽然是在obj.fn内部定义的，但是它仍然是一个普通的函数，this仍然指向window

###### 补充 --- 在构造函数的prototype中，this代表着什么。


```javascript
function Fn() {
  this.name = 'xxx'
  this.year = 1991
}

Fn.prototype.getName = function () {
  console.log(this.name);
}

var f1 = new Fn()
f1.getName();
```

如上代码，在Fn.prototype.getName函数中，this指向的是f1对象。因此可以通过this.name获取f1.name的值。

***其实，不仅仅是构造函数的prototype，即便是在整个原型链中，this代表的也都是当前对象的值***
