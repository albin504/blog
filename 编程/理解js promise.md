假如我们要实现了一个行程规划的功能，分两步：
1. 获取当地天气
2. 天气获取成功之后，进行行程规划
这两步都要调用api，比较慢，因此我们想异步实现“行程规划”的功能。

# 过去jquery ajax用的比较多，涉及到异步回调的代码，一般是这么写的：

```
// 获取最新的天气信息
function refreshWhether(resolve, reject) {
    $.ajax({
        "url" : 'https://fiddle.jshell.net',
        'success' : function (result,status,xhr)	{
        	resolve("成功");
        },
        'error' : function (result,status,xhr)	{
        	reject("失败");
          console.log(result);
        }
    })
}


function whetherResolve(msg)
{
    alert("resolve:" + msg)
    // 天气刷新成功，执行“行程规划”异步任务.
    // 在回调中调用另一个回调会出现多层嵌套，这种“回调地狱”使我们的代码难以理解
    planTrip(msg => {
    	// 行程规划成功回调
      alert("行程规划成功")
    }, msg => {
    	// 行程规划失败回调
      alert("行程规划失败")
    });
}

function whetherReject(msg)
{
    alert("reject:" + msg)
}

// 根据天气信息，规划出行计划
function planTrip(resolve, reject) {
    $.ajax({
        "url" : 'https://fiddle.jshell.net',
        'success' : function (result,status,xhr)	{
        	resolve("成功");
        },
        'error' : function (result,status,xhr)	{
        	reject("失败");
        }
    })
}

// 获取最新的天气情况，并重新规划行程。
refreshWhether(whetherResolve, whetherReject);
```

在whetherResolve()函数中，出现了多层嵌套，怎么解决这个问题呢。

#  下面改用promise实现同样的功能：
```
// 获取最新的天气信息
function refreshWhether(resolve, reject) {
    $.ajax({
        "url" : 'https://fiddle.jshell.net',
        'success' : function (result,status,xhr)	{
        	resolve("成功");
        },
        'error' : function (result,status,xhr)	{
        	reject("失败");
          console.log(result);
        }
    })
}


function whetherResolve(msg)
{
    alert("resolve:" + msg)
    // 天气刷新成功，执行“行程规划”异步任务.
    // 在回调中调用另一个回调会出现多层嵌套，这种“回调地狱”使我们的代码难以理解
    return planTrip();
}

function whetherReject(msg)
{
    alert("reject:" + msg)
}

// 根据天气信息，规划出行计划
function planTrip() {
    return $.ajax({
        "url" : 'https://fiddle.jshell.net'
    })
}


// 串行之行多个任务
var promise = new Promise(refreshWhether);
promise.then(whetherResolve).then((data) => {
  // planTrip中ajax请求的返回的内容（html）
	console.log(data);
}).catch(whetherReject);
```

通过promise.then().then()，我们规避了前面循环嵌套的问题，代码可读性更好。

# **这里我们来理一下，promise到底是什么？**

> Promise 是现代 JavaScript 中异步编程的基础，是一个由异步函数返回的可以向我们指示当前操作所处的状态的对象。

1. 先有一个异步函数。如setTimeout、ajax，都是异步函数
2. 通过 **new Promise(asyncFunc)** 的方式，封装异步函数，返回一个“承诺”
3. 承诺，当异步任务执行完成后，执行特定的回调函数。

Promise 有三种状态：
- 待定（pending）：初始状态，既没有被兑现，也没有被拒绝。这是调用 fetch() 返回 Promise 时的状态，此时请求还在进行中。
- 已兑现（fulfilled）：意味着操作成功完成。当 Promise 完成时，它的 then() 处理函数被调用。
- 已拒绝（rejected）：意味着操作失败。当一个 Promise 失败时，它的 catch() 处理函数被调用。


下面用一个例子，来说明如何构造一个Promise对象，以及Promise对象状态流转过程。
> https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/Promise

```
const myFirstPromise = new Promise((resolve, reject) => {
  //do something asynchronous which eventually calls either:
     resolve(someValue)        // fulfilled
  or
     reject("failure reason")  // rejected
});
```
即在异步函数中：
- 调用reslove()方法，会使得“承诺”的状态变为"fulfilled",然后它的 then() 处理函数被调用
- 调用reject()方法，会使得“承诺”的状态变为"rejected",然后它的 catch() 处理函数被调用

# promise 的一种特殊语法async/await 
> https://zh.javascript.info/async-await

在函数前面的 “async” 这个单词表达了一个简单的事情：即这个函数总是返回一个 promise。其他值将自动被包装在一个 resolved 的 promise 中。
啥意思呢？
```
async function f() {
  return 1;
}
f().then(alert); // 1
```
这段代码，就类似于“new了一个Promise对象，并调用了resolve()方法”后的效果，promise会自动进入fulfilled状态，then()方法会被执行。

个人理解：async关键字，从功能上讲：是把同步函数，改为异步函数。看下面的例子：

```
async function f() {
	// 用法
  return 1;
}

f().then(() => {
	console.log("then");
}); // 1

console.log("开始执行");
```
函数f() 现在就是异步执行了，先输出“开始执行”，再输出"then“.


## await关键字就是等待promise状态变为“fulfilled”,把异步改为同步。
看例子：
```
async function f() {

  let promise = new Promise((resolve, reject) => {
    setTimeout(() => resolve("done!"), 1000)
  });

  let result = await promise; // 等待，直到 promise resolve (*)
	console.log(result);
  alert(result); // "done!"
}

f();

console.log("开始执行")
```

# js fetch api替代jquery

刚工作那会儿写js，发http请求用的都是jquery。
今天发现了js fetch api，使用起来要更加便捷。
看一个例子：
```
fetch('https://api.github.com/users/ruanyf')
  .then(response => response.json())
  .then(json => console.log(json))
  .catch(err => console.log('Request Failed', err)); 
```

fetch()返回的的是Promise对象，fetch()接收到的response是一个 Stream 对象，response.json()是一个异步操作，返回的也是Promise对象。
因此就可以进行链式调用了then().then()



