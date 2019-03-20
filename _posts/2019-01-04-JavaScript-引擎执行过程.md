---
layout:     post
title:      JavaScript 引擎执行过程
subtitle:   JavaScript 原理学习笔记
date:       2019-01-04
author:     Mokaffe
header-img: img/post-bg-ios9-web.jpg
catalog: true
tags:
    - JavaScript
    - Event Loop
---

> 学习&参考资料:
> - [这一次，彻底弄懂 JavaScript 执行机制](https://juejin.im/post/59e85eebf265da430d571f89)
> - [10分钟理解JS引擎的执行机制](https://segmentfault.com/a/1190000012806637)
> - [微任务、宏任务与Event-Loop](https://segmentfault.com/a/1190000016022069)
> - [JavaScript 异步、栈、事件循环、任务队列](https://segmentfault.com/a/1190000011198232)



## 前言
写了2个月的前端，照葫芦画瓢可以写页面了，但是JavaScript是个什么鬼，怎么执行的，同事说es5，es6的执行过程有可能不同，what！我看的这些资料又是以哪个为标准......内心很崩溃，写此文搞清楚JavaScript引擎的执行过程。

## JavaScript 的特点
* **JavaScript是单线程语言**
  在浏览器中一个页面永远只有一个线程在执行js脚本代码（在不主动开启新线程的情况下）
* **JavaScript 是单线程语言，但是代码解析却十分的快速，不会发生解析阻塞。**
JavaScript是异步执行，通过**事件循环(Event Loop)**的方式实现。

**为什么JavaScript是单线程的？**
js是为浏览器端设计的脚本语言，主要任务是操作Dom，如果设计成多线程，势必是需要牵扯多个线程争夺Dom资源，为了解决资源争夺问题必然要引入🔐，这样的实现会非常臃肿。

**为什么需要异步？**
主要是用户友好，不让用户感觉到等待，如果浏览器处理一个任务时间很长，你能忍！？！设计成单线程，加上异步执行，就可以减少用户等待时间，俗称用户体验。

 **单线程如何实现异步？**
通过event loop实现异步，理解了event loop机制,就理解了JS的执行机制


## JavaScript Event Loop（事件循环）
上面说了，JavaScript单线程异步操作是通过Event Loop实现的，那么我们就需要了解Event Loop到底是个撒。

JavaScript是单线程的，当遇到需要超长时间才能执行完成的任务怎么办？为了解决单个任务执行时间超长的问题，JS根据当前任务是否可以立即执行完毕，将任务分为：
* 同步任务 （可以立即执行）
* 异步任务（不等待，先去做其他任务）

```js
console.log(1);

setTimeout(function() {
  console.log(2)
}, 3000);

console.log(3)
```
按照同步任务、异步任务划分，以上代码的执行过程：1，3，2。

> - 代码加载进来，按照顺序执行，第一行代码 `console.log(1)`同步任务，立即执行，打印 `1` 
> - 然后遇到`setTimeout()`函数，是异步任务，不等待，继续往后执行下面的代码
> - 遇到  `console.log(3)`同步任务，立即执行，打印 `3`
> - 回到异步任务  `setTimeout()`函数， 打印 `2`

![image.png](https://upload-images.jianshu.io/upload_images/5013098-97056b990bea0bfb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


项目中的代码哪有这么简单，一般项目中的代码复杂度都是下面这样的
```js
setTimeout(function(){
    console.log('定时器开始啦')
});

new Promise(function(resolve){
    console.log('马上执行for循环啦');
    for(var i = 0; i < 10000; i++){
        i == 99 && resolve();
    }
}).then(function(){
    console.log('执行then函数啦')
});

console.log('代码执行结束');
```

那么按照之前的同步和异步任务划分，代码运行的结果应该是：
```js
懵逼了。。。。
... 怎么判断是同步任务还是异步任务？
... 同步任务执行完后，剩下的异步任务是什么顺序调用呢？
```
###### 怎么判断是同步任务还是异步任务？
一般来说满足一定条件后,才去执行的,这类代码,我们叫异步代码，被视作异步任务。有一些常见的异步操作有：`setTimeout()` ，`Promise的then, catch 等callback 函数` ...
当你写的代码越多，对编程理解的不断深入，就会很清晰的知道哪些是异步任务。

###### 那么，多个异步任务的调用顺序是怎么样的?
我们先假设异步任务的调用顺序是按照添加到任务队列的先后顺序执行的。
当我们知道 `setTimeout()` ，`Promise的 then 函数` 是异步任务的时候，我们按照现有的同步和异步任务处理过程，模拟出代码的执行结果是：
> - `setTimeout ` 是异步任务，先放一放，不执行
> - `Promise 的 then 函数` 是异步任务，先放一放，不执行，只执行`马上执行for循环啦`
此时，按照假设，`Promise 的 then 函数`任务放在`setTimeout`任务后面。
> - 打印`代码执行结束`, 此时，同步任务执行完毕，按顺序执行异步任务
> - `定时器开始啦`
> - `执行then函数啦`

但是，将代码放在chrome上运行的时候，真正的结果不是按照之前的假设执行的，而是先执行的 `Promise 的 then 函数`, 然后执行的`setTimeout`。
此时，我们可以认为异步任务按照添加顺序执行的假设不成立。
![image.png](https://upload-images.jianshu.io/upload_images/5013098-a2c0994960efba0b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
知识的匮乏，会让我们在接触到新问题时忘记问一下为什么。
- 为什么js的异步任务不是按照添加的顺序执行？
- 为什么js的Event Loop 会设计出两个新概念：**宏任务&微任务**来控制任务的执行过程？

理解完了宏任务和微任务的原理，也没有想明白，上面这两个为什么？需再思考思考。

### 宏任务&微任务
可以阅读 [微任务、宏任务与Event-Loop](https://segmentfault.com/a/1190000016022069)的这篇文章来理解宏任务和微任务的概念。

**自己的理解**
- 宏任务是<script> 标签代码里的所有任务已经按照哪些是宏任务已经排好队列了。
- 微任务是当前宏任务中产生的异步任务，等待异步任务执行完后进入微任务队列。

当JavaScript代码开始执行的时候，此时，只有一个宏任务，就是<script>代码里面的所有同步任务，这些同步任务里面会有一些带有callback函数的异步调用，产生了微任务，此时，会将已经返回的callback 函数放到微任务队列。

在当前宏任务中产生的微任务没有执行完成时，是不会执行下一个宏任务的。也就是说微任务队列只有一个，只有当微任务队列为空的时候，才会执行下一个宏任务。

这[微任务、宏任务与Event-Loop](https://segmentfault.com/a/1190000016022069)这篇文章中，给出了业界比较流行的宏任务和微任务的分类，并且和node进行了对比：
![宏任务和微任务.png](https://upload-images.jianshu.io/upload_images/5013098-49a2a18b32e5ffce.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


下面这张图来自 [JavaScript 异步、栈、事件循环、任务队列](https://segmentfault.com/a/1190000011198232)，这篇文章图文并茂，能够更好的理解JavaScript引擎执行过程。
![JavaScript异步，Event Loop，任务队列.png](https://upload-images.jianshu.io/upload_images/5013098-4d40f5db1e0b09ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从图上可以看出，任务队列分为宏任务队列和微任务队列。宏任务队列可以有多个，但是微任务队列只能有一个。

#### 代码解析
```js
setTimeout(function(){ 
    console.log('定时器开始啦')
});

new Promise(function(resolve){
    console.log('马上执行for循环啦');
    for(var i = 0; i < 10000; i++){
        i == 99 && resolve();
    }
}).then(function(){
    console.log('执行then函数啦')
});

console.log('代码执行结束');
```

这段代码解析后，会立即从头到尾执行，这是第一个宏任务，此时是没有微任务的。
- 遇到`setTimeout()`, 判断是宏任务，放到宏任务队列
- 继续执行，遇到 `Promise()`,那么会打印`马上执行for循环啦`, `then()`函数是异步执行，此时产生了微任务，等待当前的宏任务执行完毕后，才会执行微任务
- 打印`代码执行结束`
- 当前宏任务执行完毕，查看微任务是否存在，发现了 `then()` 函数这个微任务需要执行，打印`执行then函数啦`
- 当前宏任务和它产生的微任务已经执行完毕，可以执行下一个宏任务，发现了 `setTimeout()`，打印`定时器开始啦`


## 理解Event Loop
**2019年02月14日，此时的理解**
```js 
当JavaScript代码准备被执行的时候，只有宏任务，即代码里的所有能够立即执行的同步任务，
当同步任务中产生的一些异步任务准备完毕后，会将这些异步任务放到微任务队列，
同时，同步任务中会有其他的一些宏任务，那么这些任务不会立即执行，
而是会放到宏任务队列，等待当前宏任务执行完毕，
随后去执行产生的微任务，才会去执行下一个宏任务。
```
期待以后温故知新，理解更深入。

## Conclusion
现在信息越来越多，当你需要一些深入的理解时候，去找到一些文章的时候，发现很多文章都很相像，以至于不清楚的地方，翻看了很多相似的文章还是不清不楚。需要不断反思，不断查找，才能完善自己的知识体系。

## Punch List
- [JavaScript引擎、虚拟机、运行时环境是一回事儿吗？](https://www.zhihu.com/question/39499036) 
- [浏览器与Node的事件循环(Event Loop)有何区别?](https://juejin.im/post/5c337ae06fb9a049bc4cd218) 

> 学习过程中，阅读的相关资料
> - [js引擎的执行过程（一）](https://heyingye.github.io/2018/03/19/js%E5%BC%95%E6%93%8E%E7%9A%84%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B%EF%BC%88%E4%B8%80%EF%BC%89/)
> - [[译]理解 Javascript 执行上下文和执行栈](https://juejin.im/post/5bdfd3e151882516c6432c32)
> - [JavaScript运行原理解析](https://www.kancloud.cn/digest/liao-js/149467)
> - [Javascript 工作原理](https://cnodejs.org/topic/579e341885dba6b12ac58583)
