### Event Loop

宏任务和微任务切换执行，不断循环，形成了事件循环。

**浏览器执行script的过程**

![event-loop](/imgs/event-loop.jpg)

**宏任务 Macrotask** 有哪些？

HTML parsing、Network events、Mouse events、Keybord events、setTimeout、setInterval

**微任务 Microtask** 有哪些？

process.nextTick、Promise.then、Promise.catch、Promise.finally、DOM mutations

**判断以下代码，在node中的输出结果**

```
// 主线程直接执行
console.log(1);
// 放入宏事件队列中
setTimeout(function() {
    console.log(5);
    new Promise(function(resolve) {
        console.log(6);
        resolve();
    }).then(function() {
        console.log(8)
    })
    process.nextTick(function() {
      console.log(7);
    })
})
// 微任务
process.nextTick(function() {
    console.log(3);
})
// 主线程直接执行
new Promise(function(resolve) {
    console.log(2);
    resolve();
}).then(function() {
    // 微任务
    console.log(4)
})
```

输出顺序就是数字本身。注意：同为微任务，process.nextTick会在Promise.then之前哦





[浏览器事件循环](http://www.ruanyifeng.com/blog/2014/10/event-loop.html)