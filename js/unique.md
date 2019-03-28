### 单例模式

**通过单例模式可以保证开发中，一个类只有一个实例。**

网上的实现方式很多，我也写了一个，感觉更容易理解

```
const Unique = (function () {
  var instance
  return function () {
    if (!instance) {
      instance = new (function () {})()
    }
    return instance
  }
})()

let u1 = new Unique()
let u2 = new Unique()
console.log(u1===u2) // true
u1.do1 = function () {}
u2.do2 = function () {} // 实质上u1和u2都是同一个对象，是在同一个对象上进行的操作
```

实现的关键：

1.闭包，缓存实例，外部碰不到，只有暴漏出的函数可以访问到

2.如果已经被实例化，则返回；未实例化，则进行唯一一次实例化过程