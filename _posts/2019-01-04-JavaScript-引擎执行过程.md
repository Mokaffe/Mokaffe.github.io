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
---

>参考资料:
- [ js引擎的执行过程（一）](https://heyingye.github.io/2018/03/19/js%E5%BC%95%E6%93%8E%E7%9A%84%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B%EF%BC%88%E4%B8%80%EF%BC%89/)
- [[译]理解 Javascript 执行上下文和执行栈](https://juejin.im/post/5bdfd3e151882516c6432c32)
- [这一次，彻底弄懂 JavaScript 执行机制](https://juejin.im/post/59e85eebf265da430d571f89)

# JavaScript是什么鬼
写了2个月的前端，照葫芦画瓢可以写页面了，但是JavaScript是个什么鬼，怎么执行的，同事说es5，es6的执行过程有可能不同，what！我看的这些资料又是以哪个为标准......内心很崩溃，写此文搞清楚JavaScript引擎的执行过程。

# JavaScript 的特点
* **JavaScript是单线程语言**
  在浏览器中一个页面永远只有一个线程在执行js脚本代码（在不主动开启新线程的情况下）
* **JavaScript 是单线程语言，但是代码解析却十分的快速，不会发生解析阻塞。**
JavaScript是异步执行，通过**事件循环(Event Loop)**的方式实现。

## 为什么JavaScript是单线程的？
js是为浏览器端设计的脚本语言，主要任务是操作Dom，如果设计成多线程，势必是需要牵扯多个线程争夺Dom资源，为了解决资源争夺问题必然要引入🔐，这样的实现会非常臃肿。

## 为什么需要异步？
主要是用户友好，不让用户感觉到等待，如果浏览器处理一个任务时间很长，你能忍！？！设计成单线程，加上异步执行，就可以减少用户等待时间，俗称用户体验。

## 单线程如何实现异步？
通过event loop实现异步，理解了event loop机制,就理解了JS的执行机制

# 人脑运行代码检验理解js执行过程
这两个例子贴在这里，因为才看到这些例子的时候，懵逼，等待理解完后写出解答过程，看到此文的可以留言交流，因为我也不知道正确的分析过程是什么样的。
##题目（1）
```
console.log(person);
console.log(fun);
var person="Eric"; 
console.log(person);

function fun() { 
  console.log(person); 
  var person="Tom"; 
  console.log(person);
} 

fun();     
console.log(person)
```


##题目（2）
```
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



##分析过程
### 题目（1）分析过程
![题目（1）运行结果.png](https://upload-images.jianshu.io/upload_images/5013098-f9d4cee12e8bc6e4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 题目（2）分析过程
![题目（2）运行结果.png](https://upload-images.jianshu.io/upload_images/5013098-07584fb92bb8ca25.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


# JavaScript Event Loop（事件循环）
上面说了，JavaScript单线程异步操作是通过Event Loop实现的，那么我们就需要了解Event Loop到底是个撒。

JavaScript是单线程的，那么当遇到需要超长时间才能执行完成的任务怎么办？那么根据任务是否可以立即执行完毕，将任务分为：
* 同步任务 （可以立即执行）
* 异步任务（不等待，知道任务执行时间长，先去做其他任务）



# 原理解析JavaScript引擎执行过程
JavaScript引擎执行过程分为三个阶段：
1. 语法分析
2. 预编译阶段
3. 执行阶段

在HTML中写JavaScript脚本的时候，需要添加<script>标签，浏览器首先按顺序加载由<script>标签分割的js代码块，加载js代码块完毕后，立刻进入以上三个阶段，然后再按顺序查找下一个代码块，再继续执行以上三个阶段，无论是外部脚本文件（不异步加载）还是内部脚本代码块，都是一样的原理，并且都在同一个全局作用域中。

## 语法分析

js脚本加载完毕后，会首先进行语法分析阶段，该阶段主要作用是：
> 分析js脚本代码块的语法是否正确，如果出现不正确，则向外抛出一个语法错误（），停止该js代码块的执行，然后继续查找下一个并加载下一个代码块，如果语法正确，则会进行预编译阶段。
（疑问：出现语法错误，）
