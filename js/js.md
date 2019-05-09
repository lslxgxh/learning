# js

## 1.作用域

**作用域，是指变量的生命周期（一个变量在哪些范围内保持一定值）**

### 1.1 全局作用域

全局变量：生命周期将存在于整个程序之内。能被程序中任何函数或者方法访问。在 JavaScript 内默认是可以被修改的。

* 显式声明：

  使用var（关键字）+变量名(标识符)的方式在function外部声明，即为全局变量，否则在function声明的是局部变量。

* 隐式声明：

  没有使用var，直接复制例如a = 0,就会把a隐性定义为全局变量（无论a是否在function内）

### 1.2 函数作用域

函数作用域内，对外是封闭的，从外层的作用域无法直接访问函数内部的作用域

~~~js

function bar() {
  var testValue = 'inner';
}

console.log(testValue);		// 报错：ReferenceError: testValue is not defined

~~~



* 通过 return 访问函数内部变量

~~~js
function bar(value) {
  var testValue = 'inner';
  
  return testValue + value;
}

console.log(bar('fun'));		// "innerfun"

~~~

* 通过 闭包 访问函数内部变量

  ~~~js
  function bar(value) {
    var testValue = 'inner';
    
    var rusult = testValue + value;
    
    function innser() {
       return rusult;
    };
    
    return innser();
  }
  
  console.log(bar('fun'));		// "innerfun"
  
  ~~~

  

* 立即执行函数:它能够自动执行 `(function() { //... })()` 里面包裹的内容，能够很好地消除全局变量的影响；

~~~js
<script type="text/javascript">

    (function() {

      var testValue = 123;

      var testFunc = function () { console.log('just test'); };

    })();

    console.log(window.testValue);		// undefined
    
    console.log(window.testFunc);		// undefined
    
</script>

~~~

### 1.3 块级作用域

很明显，用 var 关键字声明的变量，在 `for` 循环之后仍然被保存这个作用域里；

这可以说明： `for() { }`仍然在，全局作用域里，并没有产生像函数作用域一样的封闭效果；

如果想要实现 **块级作用域** 那么我们需要用 `let` 关键字声明！！！

 ~~~js
for(let i = 0; i < 5; i++) {
  // ...
}

console.log(i)	
 ~~~

**在 for 循环执行完毕之后 i 变量就被释放了，它已经消失了！！！**

同样能形成块级作用域的还有 `const` 关键字：

~~~js
if (true) {
  const a = 'inner';
}

console.log(a);				// 报错：ReferenceError: a is not defined

~~~

`let` 和 `const` 关键字，创建块级作用域的条件是必须有一个 `{ }` 包裹：

~~~js
{
  let a = 'inner';
}
  
if (true) {
   let b = 'inner'; 
}

var i = 0;

// ......
~~~

举一个面试中常见的例子：

```
for(var i = 0; i < 5; i++) {
  setTimeout(function() {
     console.log(i);			// 5 5 5 5 5
  }, 200);
};
复制代码
```

这几乎是作用域的必考题目，你会觉得这种结果很奇怪，但是事实就是这么发生了；

这里的 i 是在全局作用域里面的，只存在 1 个值，等到回调函数执行时，用词法作用域捕获的 i 就只能是 5；

因为这个循环计算的 i 值在回调函数结束之前就已经执行到 5 了；我们应该如何让它恢复正常呢？？？

**解法1：调用函数，创建函数作用域：**

```js
for(var i = 0; i < 5; i++) {
  abc(i);
};

function abc(i) {
  setTimeout(function() {
     console.log(i);			// 0 1 2 3 4 
  }, 200); 
}

复制代码
```

这里相当于创建了5个函数作用域来保存，我们的 i 值；

**解法2：采用立即执行函数，创建函数作用域；**

```js
for(var i = 0; i < 5; i++) {
  (function(j) {
    setTimeout(function() {
      console.log(j);
    }, 200);
  })(i);
};
复制代码
```

原理同上，只不过换成了自动执行这个函数罢了，这里保存了 5 次 i 的值；

**解法3：let  创建块级作用域**

```js
for(let i = 0; i < 5; i++) {
    setTimeout(function() {
      console.log(i);
    }, 200);
};
```

### 1.4 词法作用域

词法作用域是指一个变量的可见性，及其文本表述的模拟值（《JavaScript函数式编程》）;

听起来，十分地晦涩，不过将代码拿来分析就非常浅显易懂了；

```js
testValue = 'outer';

function afun() {
  var testValue = 'middle';
  
  console.log(testValue);		// "middle"
  
  function innerFun() {
    var testValue = 'inner';
    
    console.log(testValue);		// "inner"
  }
  
  return innerFun();
}

afun();

console.log(testValue);			// "outer"

```

**当我们要使用声明的变量时：JS引擎总会从最近的一个域，向外层域查找；**



![img](https://user-gold-cdn.xitu.io/2018/3/28/1626ce92a304bc47?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



再举一个一个实际的例子：

```js
var testValue = 'outer';

function foo() {
  console.log(testValue);		// "outer"
}

function bar() {
  var testValue = 'inner';
  
  foo();
}

bar();
```

**显然，当 JS 引擎查找这个变量时，发现全局的 testValue 离得更近一些，这恰好和 动态作用域 相反；**



![img](https://user-gold-cdn.xitu.io/2018/3/28/1626ce99fad0f0ea?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



如上图所示，下面将讲述与 词法作用域相反的动态作用域；

### 1.5 动态作用域

在编程中，最容易被低估和滥用的概念就是动态作用域（《JavaScript函数式编程》）。

在 JavaScript 中的仅存的应用动态作用域的地方：`this` 引用，所以这是一个大坑！！！！！

> 动态作用域，作用域是基于调用栈的，而不是代码中的作用域嵌套；
>
> 作用域嵌套，有词法作用域一样的特性，查找变量时，总是寻找最近的作用域；

同样是，词法作用域，例子2，同一份代码，**如果** 是动态作用域：

```
var testValue = 'outer';

function foo() {
  console.log(testValue);		// "inner"
}

function bar() {
  var testValue = 'inner';
  
  foo();
}

bar();
复制代码
```

**当然，JavaScript 除了this之外，其他，都是根据词法作用域查找！！！**

[![img](https://user-gold-cdn.xitu.io/2018/3/28/1626ce78d69108cb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)](https://user-gold-cdn.xitu.io/2018/3/28/1626ce78d69108cb?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

为什么要理解动态作用域呢？因为，这能让你更好地学习 `this` 引用！！！

要清楚，JavaScript 实际上没有动态作用域。它拥有词法作用域。就这么简单。但是 `this` 机制有些像动态作用域。

关键的差异：词法作用域是编写时的，而动态作用域（和 `this`）是运行时的。词法作用域关心的是 *函数在何处被声明*，但是动态作用域关心的是函数 *从何处* 被调用。

## 2 原型链
http://www.ruanyifeng.com/blog/2011/06/designing_ideas_of_inheritance_mechanism_in_javascript.html
https://juejin.im/post/58f94c9bb123db411953691b

## 3 闭包
http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html

## 4 模块化
http://www.ruanyifeng.com/blog/2012/10/javascript_module.html
http://www.ruanyifeng.com/blog/2012/10/asynchronous_module_definition.html
http://www.ruanyifeng.com/blog/2012/11/require_js.html

## 5 ES6
https://es6.ruanyifeng.com/
30分钟搞定es6重点
https://juejin.im/entry/58f21df95c497d006c87469e
