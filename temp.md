

Javascript实现并发的方式：

Javascript的执行默认情况下是串行的，一个NodeJS应用只会跑在一个进程中，NodeJS中也没有多线程的概念。

通过事件+回调机制，让浏览器或NodeJS去执行异步回调函数，缺点：容易引入回调地狱



promise ：https://nodejs.dev/learn/understanding-javascript-promises

https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/resolve

promise创建完以后，会是pending状态，最终会变成resolved或rejected状态，分别调用响应的回调函数(then catch)

await一个promise对象，是在等待promise的状态变为resolved，await的返回值是这个promise的resolve或reject出来的数据

async标记一个函数是异步的函数，它会返回一个promise,  该函数会异步执行（被放到Job Queue？？？）

Promise.resolve('test')是创建一个resolved状态的promise对象？resolve是将当前promise对象的状态变为resolve还是创建一个新的promise对象？？

resolve会得到一个promise

一个promise被创建出来后，它的executor就会被执行，执行完使用resolve会得到一个新的promise对象，调用这个对象的then可以得到resolve的信息

EventEmitter事件发出后，会同步执行所有监听的回调函数，按照它们注册的顺序。可以使用setImmediate() or process.nextTick()将回调函数转成异步的方式执行。