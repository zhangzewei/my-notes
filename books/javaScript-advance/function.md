## 摘要
一般来说，一个函数是可以通过外部代码调用的一个“子程序”（或在递归的情况下由内部函数调用）。像程序本身一样，一个函数由称为函数体的一系列语句组成。值可以传递给一个函数，函数将返回一个值。在 JavaScript 中，函数是 **头等 (first-class)** 对象，因为它们可以像任何其他 **对象** 一样具有属性和方法。它们与其他对象的区别在于函数可以被调用本文主要收录一些关于JavaScript中函数的应用技巧，希望能够帮助到大家在开发过程中提升技巧。

### IIFE 立即执行函数
```js
(function () {
  statements;
})();
```
在函数体外面使用括号包裹起来，然后在使用括号执行，就能够达到立即执行的效果，但是这个函数内部的东西是无法被外部访问的，除非挂载在某个已经存在的对象上。

![image.png](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b7819fa0cf0447b29d9731f31466264e~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=926&h=410&s=61282&e=png&a=1&b=1c1d20)

在项目中没有使用框架写纯js的时候会使用这种方式进行变量的隔离，然后使用公用对象进行数据传递。

### arguments
在函数内部可以拿到获取到的参数的数据，arguments 对象只能在函数内使用。
```js
function A() {
  console.log(arguments)
}

A(1,2,3, [1,2,3]);
/*
{
  "0": 1,
  "1": 2,
  "2": 3,
  "3": [
    1,
    2,
    3
  ]
}
*/
```
arguments 是对象，类数组，需要用from对其进行转换才能够使用数组的操作。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4135eb4820db4884ae4a3b0a7e6d3335~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=709&h=214&s=28409&e=png&a=1&b=1b1c1f)
### Function.length
获取到函数括号内部的参数个数，如果有预设参数，那么获取的个数是预设参数之前的个数。

![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c37adb1dfb1415bb815aac97acf9313~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=548&h=218&s=26207&e=png&a=1&b=1b1c1f)

### 闭包
闭包（closure）是一个函数以及其捆绑的周边环境状态（lexical environment，词法环境）的引用的组合。换而言之，闭包让开发者可以从内部函数访问外部函数的作用域。在 JavaScript 中，闭包会随着函数的创建而被同时创建。

那么我们可以利用闭包的特性来进行一些timer的操作：

#### 1. 防抖
```js
const debounce = (fn, during) => {
    const timer = null;
    return (...args) => {
        clearTimeout(timer); // 直接clear，有的话直接销毁，没有的话也没关系
        timer = setTimeout(() => { fn(...args) }, during);
    }
}
```
#### 2. 节流
```js
const throttle = (fn, during) => {
    const timer = null;
    return (...args) => {
        if (!timer) {
            timer = setTimeout(() => {
                fn(...args);
                clearTimeout(timer);
                timer = null;
            }, during);
        }
    }
}
```
### 递归
调用自身的函数我们称之为递归函数。在某种意义上说，递归近似于循环。两者都重复执行相同的代码，并且两者都需要一个终止条件（避免无限循环，或者在这种情况下更确切地说是无限递归）。

```js
// 将这个循环改编为递归
let x = 0;
// “x < 10”是循环条件
while (x < 10) {
  // 做些什么
  x++;
}

function loop(x) {
  // “x >= 10”是退出条件（等同于“!(x < 10)”）
  if (x >= 10) {
    return;
  }
  // 做些什么
  loop(x + 1); // 递归调用
}
loop(0);
```

斐波那契数列

```js
function fib(n) {
    if (n === 1 || n === 2) return n;
    return fib(n - 1) + fib(n - 2)
}
```
### 可理化
使用递归，也可以实现可理化，柯里化是一种函数的转换，它是指将一个函数从可调用的 f(a, b, c) 转换为可调用的 f(a)(b)(c)。

柯里化不会调用函数。它只是对函数进行转换。

```js
const curry = fn => 
    curried = (...args) => 
        (...arg) => {
            // 这个函数是递归
            if (arg.length === 0) {
                return fn(...args, ...arg)
            } else {
                return curried(...args, ...arg)
            }
        }
    const sum = (...args) => {
        return args.reduce((a,b) => a+b, 0);
    }

const curriedSum = curry(sum);
console.log('curry test',curriedSum(1)(1,3)(3,3,3,3,3,3)())

// "curry test" 23
```
## 参考文献
- [柯理化](https://zh.javascript.info/currying-partials)
- [MDN 函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Functions)