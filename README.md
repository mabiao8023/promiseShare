# promise分享

先来了解一下几个概念。

## 概念一：js线程

js从一诞生，执行环境被设计为单线程的。

HTML5提出Web Worker标准，允许创建多个线程，但是子线程完全受主线程控制，且不得操作DOM，只是用来操作逻辑复杂和执行时间较长的代码。所以，这个新标准并没有改变jst单线程的本质。


```
// main.js
 //实例化一个webWorker,创建了一个子线程
  w = new Worker("demo_workers.js");
  //监听
  w.onmessage = function (event) {
      document.getElementById("result").innerHTML=event.data;
  };
    
//demo_workers.js
var i=0;
function timedCount()
{
    i=i+1;
    postMessage(i);
    setTimeout("timedCount()",500);
}

timedCount();

```

## 概念二：回调队列与回调函数


单线程有个缺点，在执行时间长的代码块时，页面失去响应，造成页面假死，用户体验极差。

为了改进这种体验，js将执行的任务分成了两种：

1. 一种是同步任务（synchronous）

2. 一种是异步任务（asynchronous）

异步任务？异步任务有哪几种呢？



1. DOM事件任务(onclick)
2. 定时器事件任务(setTimeout/setInteval);
3. XMLHttpRequest事件任务（ajax）;

**特点：** 

1. 异步任务都是通过异步**回调函数**完成；
2. 异步任务不进入主线程而进入一个叫**回调队列**的东西；
3. 异步任务等主线程空闲时才去执行回调队列中的异步回调函数；

简单的看图来解释一下上面这些的相互关系:

![image](https://mabiao8023.github.io/web-flex/htdl.png)

**event loop(事件循环)**：

js会创建一个类似于while(ture)的循环，每次循环就是查看是否有待处理的事件，有则取出相关事件及回调函数放入到执行中由主线程执行，待处理的事件会存储在一个回调队列中。

**callback quene(回调队列)**：

异步任务会将相关回调函数添加到回调队列中。而不同的异步任务添加到回调队列的时机也不同，这些异步任务是由浏览器内核的 webcore 来执行的，webcore 包含3种 webAPI，分别是 DOM Binding、network、timer模块。

1. DOM事件(onclick)由DOM Binding 模块来处理，当事件触发的时候，回调函数会立即添加到回调队列中；
2. 定时器事件(setTimeout) 由timer 模块来进行延时处理，当时间到达的时候，才会将回调函数添加到回调队列中；
3. XHR(ajax)由network 模块来处理，在网络请求完成返回之后，才将回调函数添加到回调队列中；



## 小结：

一切的异步任务都是要用 **回调函数** 实现的。

如今越来越注重页面体验的时代，多层的异步回调会写的越来越难以掌控，而promise就是来拯救js的多级回调。下面让我们来看看什么是promise。

# Promise(“承诺”)

承诺在将来一定发生的事情。

[promise/A+规范：](http://malcolmyu.github.io/malnote/2015/06/12/Promises-A-Plus/)
一个开放、健全且通用的 JavaScript Promise 标准。由开发者制定，供开发者参考。

promise是一个拥有 then 方法的对象或函数，其行为符合规范；

规范摘要：

Promise对象有三种状态：

- 等待态（Pending）
- 执行态（Fulfilled）
- 拒绝态（Rejected）


promise状态可转变，一旦转变状态不再变化，状态转变只有两种：

- “Pending”状态转到“Fulfilled”
- “Pending”状态转到“rejected”

then 方法：一个promise对象必须提供一个then方法，状态发生改变后调用该方法。

```
promise.then(onFulfilled, onRejected)

```
参数：onFulfilled（执行状态的回调），onRejected（拒绝状态的回调）

返回值：promise对象

上面理论有点抽象，来个图看看：
![image](https://mabiao8023.github.io/web-flex/promise2.png)

jQuery等流行的js库参考规范都已经实现了这个对象,es6也实现了这个对象。下面来看看
## 1.1 JQuery中的deferred

jQuery1.5.0版本开始引入deferred对象，实现promise。
$.Deferred()返回一个deferred对象，拉出来溜溜


#### 1.1.1 什么是deferred?

简单说，**deferred对象就是jQuery的异步回调函数解决方案**。defer的意思是"延迟"，所以deferred对象的含义就是"延迟"到未来某个点再执行。

#### 1.1.2 ajax的解决方案

jQuery中的ajax的传统写法：


```
　　$.ajax({
　　　　url: "index.html",
　　　　data:data,
　　　　success: function(){
　　　　　　alert("哈哈，成功了！");
　　　　},
　　　　error:function(){
　　　　　　alert("噢，出错啦！");
　　　　},
　　　　complete:function(){
　　　　    alert("哈哈，完成了");
　　　　}
　　});
```
上面的就不用解释了，但是有没有想过$.ajax()执行完后返回的是什么呢？？

$.ajax()操作完成后，如果使用的是低于1.5.0版本的jQuery，返回的是XHR对象；如果高于1.5.0版本，返回的是deferred对象，可以进行对异步回调链式操作。

1.5.0版本上支持的写法：

```
$.ajax("test.html")
　　.done(function(){ alert("哈哈，成功了！"); })
　　.fail(function(){ alert("噢，出错啦！"); });
　　.always(function(){
　　    alert("哈哈，完成了");
　　});
```
可以看到，done()相当于success方法，fail()相当于error方法，always()相当于complete方法,采用链式写法后，代码的可读性大大提高，更加文艺了。


**一个操作多个回调函数**

deferred对象的一大好处，就是它允许你自由添加任意多个回调函数,按顺序执行。

```
$.ajax("test.html")
　　.done(function(){ alert("哈哈，成功了！");} )
　　.fail(function(){ alert("出错啦！"); } )
　　.done(function(){ alert("第二个回调函数！");} );
```

**多个操作一个回调函数**


```
$.when($.ajax("test1.html"), $.ajax("test2.html"))
　　.done(function(){ alert("哈哈，成功了！"); })
　　.fail(function(){ alert("出错啦！"); });
```
$.when()的参数只能是deferred对象。
执行$.ajax("test1.html")和$.ajax("test2.html")，如果都成功了，就运行done()指定的回调函数；如果有一个失败或都失败了，就执行fail()指定的回调函数。

#### 1.1.3 普通任务的解决方案

上面讲的都是ajax操作的回调处理，但是deferred对象的回调函数的接口还可以用于其他普通接口，同步和异步就可以使用该对象的各种方法。

我们来看一个具体的例子。假定有一个很耗时的操作wait：

```
　　var wait = function(){
　　　　var tasks = function(){
　　　　　　alert("执行完毕！");
　　　　};
　　　　setTimeout(tasks,5000);
　　};
```

我们为其指定完成后的回调函数，参考上面应该怎么做？

很自然的，你会想到，可以使用$.when()：

```
$.when(wait())
　　.done(function(){ alert("哈哈，成功了！"); })
　　.fail(function(){ alert("出错啦！"); });
```




但是，这样写的话，done()方法会立即执行，起不到回调函数的作用。
前面讲到$.when()的参数只能是deferred对象，而wait()普通函数不是deferred对象。
原因在于jQuery为ajax单独封装了deferred对象，所以必须对wait()进行改写：


```
　　var dtd = $.Deferred(); // 新建一个deferred对象
　　var wait = function(dtd){
　　　　var tasks = function(){
　　　　　　alert("执行完毕！");
　　　　　　dtd.resolve(); // 改变deferred对象的执行状态
　　　　};
　　　　setTimeout(tasks,5000);
　　　　return dtd;
　　};
　　$.when(wait())
　　.done(function(){ alert("哈哈，成功了！"); })
　　.fail(function(){ alert("出错啦！"); });
```
现在，wait()函数返回的是deferred对象，这就可以加上链式操作了，使用deferred的方法和属性了。

#### 1.1.4 deferred.resolve() 和reject()

**deferred参考promise/A+规范规定任务的三种执行状态**：
1. 未完成（pending）；
2. 已完成（fulfilled/resolved）；
3. 已失败（rejected）；

**deferred执行状态转变**：

1. 未完成 -> 已完成(由deferred.resolve()触发)：调用done()方法指定的回调函数；
2. 未完成 -> 已失败(由deferred.reject()触发) ：调用fail()方法指定的回调函数；

**注：**jq中$.ajax()的deferre对象的状态是内部机制控制转变的，而其余的状态改变就需要自己调用这两个方法自动触发状态的改变。

#### 1.1.5 deferred.promise()

上面这种写法，还是有问题。那就是dtd是一个全局对象，所以它的执行状态可以从外部改变。

```
　　var dtd = $.Deferred(); // 新建一个Deferred对象
　　var wait = function(dtd){
　　　　var tasks = function(){
　　　　　　alert("执行完毕！");
　　　　　　dtd.resolve(); // 改变Deferred对象的执行状态
　　　　};
　　　　setTimeout(tasks,5000);
　　　　return dtd;
　　};
　　$.when(wait(dtd))
　　.done(function(){ alert("哈哈，成功了！"); })
　　.fail(function(){ alert("出错啦！"); });
　　dtd.resolve();
```

尾部的dtd.resolve()，这就改变了dtd对象的执行状态，因此导致done()方法立刻执行，跳出"哈哈，成功了！"的提示框，等5秒之后再跳出"执行完毕！"的提示框。

为改善上述问题，jQuery提供了deferred.promise()方法。

**作用**：在原来的deferred对象上返回另一个deferred对象，只开放与改变执行状态无关的方法（比如done()方法和fail()方法），屏蔽与改变执行状态有关的方法（比如resolve()方法和reject()方法），从而使得执行状态不能被改变。

改进代码：

```
    var dtd = $.Deferred(); // 新建一个Deferred对象
　　var wait = function(dtd){
　　　　var tasks = function(){
　　　　　　alert("执行完毕！");
　　　　　　dtd.resolve(); // 改变Deferred对象的执行状态
　　　　};

　　　　setTimeout(tasks,5000);
　　　　return dtd.promise(); // 返回promise对象
　　};
　　var d = wait(dtd); // 新建一个d对象，改为对这个对象进行操作
　　$.when(d)
　　.done(function(){ alert("哈哈，成功了！"); })
　　.fail(function(){ alert("出错啦！"); });
　　d.resolve(); // 此时，这个语句是报错的
```
更好的方法，将dtd对象变成wait(）函数的内部对象：

```
　var wait = function(dtd){
　　　　var dtd = $.Deferred(); //在函数内部，新建一个Deferred对象
　　　　var tasks = function(){
　　　　　　alert("执行完毕！");
　　　　　　dtd.resolve(); // 改变Deferred对象的执行状态
　　　　};

　　　　setTimeout(tasks,5000);
　　　　return dtd.promise(); // 返回promise对象
　　};
　　$.when(wait())
　　.done(function(){ alert("哈哈，成功了！"); })
　　.fail(function(){ alert("出错啦！"); });
```
#### 1.1.5  $.Deferred()
$.Deferred()是deferred对象的建构函数。可以防止执行状态被外部改变的方法。

```
  var wait = function(dtd){
　　　　var tasks = function(){
　　　　　　alert("执行完毕！");
　　　　　　dtd.resolve(); // 改变Deferred对象的执行状态
　　　　};

　　　　setTimeout(tasks,5000);
　　};  
　$.Deferred(wait)
　　.done(function(){ alert("哈哈，成功了！"); })
　　.fail(function(){ alert("出错啦！"); });
```
jQuery规定，$.Deferred()可以接受一个函数名（注意，是函数名）作为参数，$.Deferred()所生成的deferred对象将作为这个函数的默认参数。
#### 1.1.6 deferred.then()
为了省事，可以把done()和fail()合在一起写，这就是then()方法。

```
$.when($.ajax( "/main.php"))
　　.then(successFunc, failureFunc );
```
如果then()有两个参数，那么第一个参数是done()方法的回调函数，第二个参数是fail()方法的回调方法。如果then()只有一个参数，那么等同于done()。

#### 1.1.7 deferred.always()

这个方法也是用来指定回调函数的，它的作用是，不管调用的是deferred.resolve()还是deferred.reject()，最后总是执行。



## 1.2 es6中的promise

ES6正式提出把Promise作为原生对象，原理实现其实都差不多。

ES6 Promise 先拉出来遛遛 ：

```
console.dir(Promise);
```
![image](https://mabiao8023.github.io/web-flex/promise.png)


#### 1.2.1 基本用法(new , then , resolve , reject)

ES6规定，Promise对象是一个构造函数，用来生成Promise实例。

下面代码创造了一个Promise实例。

```
function getNumber(){
    //实例化一个promise实例，构造函数接受一个函数作为参数
    var p = new Promise(function(resolve, reject){
        //做一些异步操作
        setTimeout(function(){
            var num = Math.ceil(Math.random()*10); //生成1-10的随机数
            if(num<=5){
                resolve(num);
            }
            else{
                reject('数字太大了');
            }
        }, 2000);
    });
    
    //返回promise对象
    return p;            
}
 
getNumber()
.then(
    //已完成后的回调
    function(data){
        console.log('resolved');
        console.log(data);
    }, 
    //已失败后的回调
    function(reason, data){
        console.log('rejected');
        console.log(reason);
    }
);
```
getNumber函数用来异步获取一个数字，2秒后执行完成，如果数字小于等于5，我们认为是“成功”了，调用resolve修改Promise的状态。否则我们认为是“失败”了，调用reject并传递一个参数，作为失败的原因。

then方法可以接受两个参数，第一个对应resolve的回调，第二个对应reject的回调，如果只有一个则是resolve的回调。

运行结果：resolved 2 或者 rejected 数字太大了

#### 1.2.1 catch用法
它和then的第二个参数一样，用来指定reject的回调，用法是这样：
```
getNumber()
.then(function(data){
    console.log('resolved');
    console.log(data);
})
.catch(function(reason){
    console.log('rejected');
    console.log(reason);
});
```
不过它还有另外一个作用：在执行resolve的回调（也就是上面then中的第一个参数）时，如果抛出异常了（代码出错了），那么并不会报错卡死js，而是会进到这个catch方法中。请看下面的代码：

```
getNumber()
.then(function(data){
    console.log('resolved');
    console.log(data);
    console.log(somedata); //此处的somedata未定义
})
.catch(function(reason){
    console.log('rejected');
    console.log(reason);
});

//运行结果
//resolved
//4 
//rejected
//ReferenceError: somedata is no defind(....)
```
#### 1.2.2 all用法
Promise的all方法提供了并行执行异步操作的能力，并且在所有异步操作执行完后才执行回调。

```
Promise
.all([runAsync1(), runAsync2(), runAsync3()])
.then(function(results){
    console.log(results);
});
```
all接收一个promise对象的数组参数。三个异步操作是并行执行的，等到它们都执行完后才会进到then里面。那么，三个异步操作返回的数据哪里去了呢？都在then里面呢，all会把所有异步操作的结果放进一个数组中传给then，就是上面的results。

#### 1.2.3 race的用法

all方法实际上是「谁跑的慢，以谁为准执行回调」
race方法「谁跑的快，以谁为准执行回调」

```
Promise
.race([runAsync1(), runAsync2(), runAsync3()])
.then(function(results){
    console.log(results);
});
```

#### 1.2.4 多级链式调用

then方法返回一个promise对象，另外，在 then 的函数当中的返回值，可以作为后续操作的参数，因此上面的例子也可以写成,可以多级链式调用，按照回调的先后顺序依次执行
```

Promise.then(function (message) {
    return message;
}).then(function (message) {
    return message  + ' World';
}).then(function (message) {
    return message + '!';
}).then(function (message) {
    alert(message);
});
```
### 1.3 promise的其他应用案例

AngularJS大量的使用了promise；
    
```
var app = angular.module('myApp', []);
app.controller('siteCtrl', function($scope, $http) {
  $http.get("http://www.runoob.com/try/angularjs/data/sites.php")
  .success(function (response) {$scope.names = response.sites;});
});
```

这一部分我了解的也不多，但是对于promise,理解核心思想后，用起来都是大同小异。


    
