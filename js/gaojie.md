## 1. 高阶函数

**高阶函数**就是那种输入参数里面有一个或者多个函数，输出也是函数的函数，这个在js里面主要是利用闭包实现的，最简单的就是经常看到的在一个函数内部输出另一个函数，比如

```
var test = function() {
    return function() {}
}
```

这个主要是利用闭包来保持着作用域：

```
var add = function() {
    var num = 0;
    return function(a) {
        return num = num + a;
    }
}
add()(1);      // 1
add()(2);      // 2
```

这里的两个add()(1)和add()(2)不会互相影响，可以理解为每次运行add函数后返回的都是不同的匿名函数，就是每次add运行后return的function其实都是不同的，所以运行结果也是不会影响的。

如果换一种写法，比如：

```
var add = function() {
    var num = 0;
    return function(a) {
        return num = num + a;
    }
}
var adder = add();
adder(1); // 1
adder(2); // 3
```

这样的话就会在之前运算结果基础上继续运算，意思就是这两个 adder 运行的时候都是调用的同一个 num



## 2. 高阶函数实现缓存(备忘模式)

比如有个函数：

```
var add = function(a) {
    return a + 1;
}
```

每次运行`add(1)`的时候都会输出`2`，但是输入`1`每次还是会计算一下`1+1`，如果是开销很大的操作的话就比较消耗性能了，这里其实可以对这个计算进行一次缓存。
所以这里可以利用高阶函数的思想来实现一个简单的缓存，我可以在函数内部用一个对象存储输入的参数，如果下次再输入相同的参数，那就比较一下对象的属性，把值从这个对象里面取出来。

```
const memorize = function(fn) {
  const cache = {}
  return function(...args) {
    const _args = JSON.stringify(args)
    return cache[_args] || (cache[_args] = fn.apply(fn, args))
  }
}
const add = function(a) {
  return a + 1
}
const adder = memorize(add)
adder(1)    // 2    cache: { '[1]': 2 }
adder(1)    // 2    cache: { '[1]': 2 }
adder(2)    // 3    cache: { '[1]': 2, '[2]': 3 }
```

用`JSON.stringify`把传给 adder 函数的参数变成了字符串，并且把它当做 cache 的 key，将 add 函数运行的结果当做 value 传到了 cache 里面，这样 memorize 的匿名函数运行的时候会返回`cache[_args]`，如果`cache[_args]`不存在的话就返回fn.apply(fn,args)，把`fn.apply(fn, arguments)`赋值给`cache[_args]`并返回。
**注意：**cache不可以是`Map`，因为Map的键是使用`===`比较的，`[1]！==[1]`，因此即使传入相同的对象或者数组，那么还是被存为不同的键。

```
const memorize = function(fn) {        //  X 错误示范
  const cache = new Map()
  return function(...args) {
    return cache.get(args) || cache.set(args, fn.apply(fn, args)).get(args)
  }
}
const add = function(a) {
  return a + 1
}
const adder = memorize(add)
adder(1)    // 2    cache: { [ 1 ] => 2 }
adder(1)    // 2    cache: { [ 1 ] => 2, [ 1 ] => 2 }
adder(2)    // 3    cache: { [ 1 ] => 2, [ 1 ] => 2, [ 2 ] => 3 }
```

