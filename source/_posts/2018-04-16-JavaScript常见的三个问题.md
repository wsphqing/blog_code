---
title: JavaScript常见的三个问题
date: 2018-04-16 10:09:47
categories: 计算机
tags: [Javascript]
---

## 事件委托、事件代理 

将一个事件侦听器绑定到整个容器上，然后在实际点击时可以访问每个确切的元素，这样的方式称为事件委托。

用户希望点击列表中某一项时执行某些操作：
```html
<ul id="todo-app">
     <li class="item">Walk the dog</li>
     <li class="item">Pay bills</li>
     <li class="item">Make dinner</li>
      <li class="item">Code for one hour</li>
</ul>
```

如果采用单独将侦听器绑定到每个列表项方法，列表元素数量少时可行，但随着列表项数量的增加，代码的执行效率将会非常低下
```js
document.addEventListener('DOMContentLoaded', function(){
     let app = document.getElementById('todo-app');
     let items = app.getElementByClassName('item');
     for (let item of items) {
          item.addEventListener('click', function(){
               console.log('you clicked on item:' + item.innerHTML);
          });
     };
});
```

<!-- more -->

更加高效的方式是采用事件委托，将事件侦听绑定到整个容器上，然后点击每个确切元素时再出发每个元素委托的事件
```js
document.addEventListener('DOMContentLoaded', function(){
     let app = document.getElementById('todo-app');
     app.addEventListener('click', function(e){
          if (e.target && e.target.nodeName === 'LI') {
               let item = e.target;
               console.log('you clicked on item: ' + item.innerHTML);
          };
     });
});
```

## 在循环内使用闭包
闭包是指有权访问另一个函数作用域中的变量的函数。闭包可以实现诸如
```js
function closure(){
     var test = 'test';
     return function(){
         return test; 
     };
};
```

编写一个函数，将它循环遍历整数列表，并在3秒延迟后打印每个元素的索引。
如果按照下边实现是错误的，因为setTimeout是异步执行的，3秒后for循环已经执行完毕，i已经为4，所以实际每次打印的是全局的i变量，输出的是4。
```js
const arr = [10, 12, 15, 21];
for (var i = 0; i < arr.length; i++) {
    setTimeout(function() {
        console.log('The index of this number is: ' + i);
    }, 3000);
};
```

**正确方法1**，使用闭包的方式，输出内部函数，内部函数打印外部函数作用域中的变量i_local；
```js
const arr = [10, 12, 15, 21];
for (var i = 0; i < arr.length; i++) {
    setTimeout(function(i_local) {
        return function() {
            console.log('The index of this number is: ' + i_local);
        };
    }(i), 3000);
};

```
变量i_local就是闭包所保存在函数作用域中的变量，for循环传入的参数为0,1,2,3 数值保存在i_local变量中，不会因为全局作用域中的i值的变化而受到影响。

**正确方法2**，使用let创建for循环，保证作用域的独立；
```js
const arr = [10, 12, 15, 21];
for (let i = 0; i< arr.length; i++) {
    setTimeout(function() {
        console.log('The index of this number is: ' + i);
    }, 3000);
};
```
ES6新增let命令，用于声明变量，但只在let命令所在的代码快内有效，所以每次循环的i只在当前循环有效。

## 函数防抖动
有些浏览器事件可以在很短的时内快速启动多次，例如调整窗口大小或向下滚动页面，用户非常快速地向下滚动页面，事件有可能在3秒的范围内被触发数千次，这可能会导致一些严重的性能问题。

在讨论构建应用程序和事件，如滚动、窗口调整大小，或键盘事件时，务必注意  函数防抖动（Debouncing） 或 函数节流（Throttling）来提升页面速度和性能。

**方法一**
函数防抖动是解决问题的一种方式，通过限制需要经过的时间，直到再次调用函数。一个正确实现函数防抖的方法是：把多个函数放在一个函数里调用，隔一定时间执行一次。

一个使用原生JavaScript实现的例子，用到了作用域、闭包、this和定时事件：
```js
// 包裹事件函数
function debounce(fn, delay) {
    // 持久化一个定时器
    let timer = null;
    // 闭包函数可以访问timer
    return function() {
        // 通过this和arguments 获得函数的作用域和参数
        let context = this;
        let args = arguments;
        // 如果事件触发，清除timer并重新开始计时
        clearTimeout(timer);
        timer = setTimeout(function() {
            fn.apply(context, args);
        }, delay);
    };
};
```

当这个函数绑定在一个事件上，只有时间调用超过了事件间隔才会调用，否则直接清除掉定时器。
```js
// 当用户滚动是函数会被调用
function foo() {
     console.log('You are scrolling!');
};

// 事件触发的两秒后，触发函数
let elem = document.getElementById('container');
elem.addEventListener('scroll', debounce(foo, 2000));
```

**方法二**
函数节流是另一个类似函数防抖的技巧，除了使用等待一段时间在调用函数的方法，函数节流还限制固定事件内只能调用一次。所以一个事件如果在100毫秒内发生10次，函数节流会每2秒调用一次函数，而不是100毫秒内全部调用。

原文：https://blog.csdn.net/aerchi/article/details/69525526