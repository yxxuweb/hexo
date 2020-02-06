---
title: 读 《深入理解javascript原型和闭包》总结（一）
date: 2018-01-10 17:52:32
tags: JavaScript
categories: JavaScript
---

### 一切都是对象

```javascript
function show(x) {
    console.log(typeof x);    // undefined
    console.log(typeof 10);   // number
    console.log(typeof 'abc'); // string
    console.log(typeof true);  // boolean
    console.log(typeof function () {});  //function

    console.log(typeof [1, 'a', true]);  //object
    console.log(typeof { a: 10, b: 20 });  //object
    console.log(typeof null);  //object
    console.log(typeof new Number(10));  //object
}
show();
```

（undefined, number, string, boolean）属于简单的值类型，不是对象。（函数、数组、对象、null、new Number(10)、）属于引用类型，都是对象。（引用类型都是对象）

*** 判断一个变量是不是对象非常简单。值类型的类型判断用`typeof`，引用类型的类型判断用`instanceof`。  ***



java中的对象是通过`new`一个`class`创建的。对于属性、方法、字段都是非常严格的。但是`javascript`的对象比较随意--数组是对象，函数式对象，对象还是对象。对象里面的一切都是属性（ *** 方法也是一种属性 *** ），属性表示为键值对的形式。*** 对象是属性的集合 ***

JavaScript中的对象可以任意的扩展属性，没有`class`的约束。

<!-- more -->

---

### 函数和对象的关系

```javascript
var fn = function() {};
console.log(fn instanceof Object);
```

函数是一种对象，你可以说数组是对象的一种，因为数组就像是对象的一个子集一样。函数和对象关系比较复杂。

通过下面的内容缕一缕

```javascript
function Fn() {
    this.name = '王福朋';
    this.year = 1988;
}
var fn1 = new Fn();

console.log(typeof (Object)); // function
console.log(typeof (Array)); // function
```

通过上述代码，可以得出：`对象都是通过函数来创建`。

### prototype 原型

*** 每个函数都有一个属性叫做prototype *** 。这个prototype的属性值是一个对象（***属性的集合，再次强调***），默认的只有一个叫做`constructor`的属性，指向这个函数本身。

![prototype](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/172121182841896.png?raw=true)

Object

![Object](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/172130097842386.png?raw=true)

```javascript
 function Fn() { }
    Fn.prototype.name = '王福朋';
    Fn.prototype.getYear = function () {
        return 1988;
    };

    var fn = new Fn();
    console.log(fn.name);
    console.log(fn.getYear());
```

Fn是一个函数，fn对象是从Fn函数new出来的，这样fn对象就可以调用Fn.prototype中的属性。

### 隐式原型


每个对象都有一个隐藏的属性——`__proto__`，这个属性引用了创建这个对象的函数的prototype。即：`fn.__proto__ === Fn.prototype`。这里的`__proto__`成为“隐式原型”

![__proto__](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/181509180812624.png?raw=true)

> 总结：每个函数function都有一个prototype，即原型。每个对象都有一个`__proto__`，可成为隐式原型。特殊：Object.prototype是一个特例——它的__proto__指向的是null，切记切记

![Object.prototype](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/181510403153733.png?raw=true)

谁创建了函数？看下面的内容

函数也是对象，它也有`__proto__`。函数是被 Function 创建出来的。 ---- 这里的 “F” 是大写

![Function](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/181511124714709.png?raw=true)

以上代码中，第一种方式是比较传统的函数创建方式，第二种是用new Functoin创建。

总结：自定义函数`Foo.__proto__`指向`Function.prototype`，`Object.__proto__`指向`Function.prototype`

![总结](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/181512068463597.png?raw=true)

> 注意点：`Object.__proto__`指向`Function.prototype`，`Function.prototype`指向的对象，它的`__proto__`指向 `Object.prototype`。因为`Function.prototype`指向的对象也是一个普通的被`Object`创建的对象，所以也遵循基本的规则。


###  instanceof

*** instanceof判断规则 ***

Instanceof运算符的第一个变量是一个对象，暂时称为A；第二个变量一般是一个函数，暂时称为B。

Instanceof的判断队则是：***沿着A的__proto__这条线来找，同时沿着B的prototype这条线来找，如果两条线能找到同一个引用，即同一个对象，那么就返回true。如果找到终点还未重合，则返回false***。

```javascript
console.log(Object instanceof Function); // true

console.log(Function instanceof Object); // true

console.console.log(Function instanceof Function);
```

![instanceof](https://github.com/yxxuweb/markdownPhoto/blob/master/markdown/181637013624694.png?raw=true)

instanceof表示的就是一种继承关系，或者原型链的结构。
