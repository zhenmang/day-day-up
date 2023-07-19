### jQuery 常用方法及其实现原理

#### 链式调用

```
// 传统写法
var elem = document.getElementById("foobar");
elem.style.background = "red";
elem.style.color = "green";
elem.addEventListener('click', function(event) {
  alert("hello world!");
}, true);
 
// jQuery 写法
$('xxx')
    .css("background", "red")
    .css("color", "green")
    .on("click", function(event) {
    　　alert("hello worl")
    })
```

#### 样式

```
// 传入键值对
jQuery("#some-selector")
  .css("background", "red")
  .css("color", "white")
  .css("font-weight", "bold")
  .css("padding", 10);
 
// 传入 JSON 对象
jQuery("#some-selector").css({
  "background" : "red",
  "color" : "white",
  "font-weight" : "bold",
  "padding" : 10
});
```

#### 属性

```
// 获取 title 属性的值

$('#id').attr('title');

// 设置 title 属性的值

$('#id').attr('title','jQuery');

 

// 获取 css 某个属性的值

$('#id').css('title');

// 设置 css 某个属性的值

$('#id').css('width','200px');
```



#### 事件

```
// binding events by passing a map
jQuery("#some-selector").on({
  "click" : myClickHandler,
  "keyup" : myKeyupHandler,
  "change" : myChangeHandler
});
 
// binding a handler to multiple events:
jQuery("#some-selector").on("click keyup change", myEventHandler);
```

### jQuery中的部分经典实现

#### 轻污染闭包
```
IIFE
(function(){}())
(function(){})()
```
```
(function(global, factory){
    ...
})(typeof window !== "undefined" ? window : this, function( window, noGlobal(){...}){});
```

#### $返回实例

用例

```
// 无 new 构造

$('#test').text('Test');


// 当然也可以使用 new

var test = new $('#test');

test.text('Test');
```

实现

```
(function(window, undefined) {
    var
    // ...
    jQuery = function(selector, context) {
        // The jQuery object is actually just the init constructor 'enhanced'
        // 看这里，实例化方法 jQuery() 实际上是调用了其拓展的原型方法 jQuery.fn.init
        return new jQuery.fn.init(selector, context, rootjQuery);
    },
 
    // jQuery.prototype 即是 jQuery 的原型，挂载在上面的方法，即可让所有生成的 jQuery 对象使用
    jQuery.fn = jQuery.prototype = {
        // 实例化化方法，这个方法可以称作 jQuery 对象构造器
        init: function(selector, context, rootjQuery) {
            // ...
        }
    }
    // 这一句很关键，也很绕
    // jQuery 没有使用 new 运算符将 jQuery 实例化，而是直接调用其函数
    // 要实现这样,那么 jQuery 就要看成一个类，且返回一个正确的实例
    // 且实例还要能正确访问 jQuery 类原型上的属性与方法
    // jQuery 的方式是通过原型传递解决问题，把 jQuery 的原型传递给jQuery.prototype.init.prototype
    // 所以通过这个方法生成的实例 this 所指向的仍然是 jQuery.fn，所以能正确访问 jQuery 类原型上的属性与方法
    jQuery.fn.init.prototype = jQuery.fn;
 
})(window);
```

- jQuery.fn.init.prototype = jQuery.fn = jQuery.prototype ;
- new jQuery.fn.init() 相当于 new jQuery() ;
- jQuery() 返回的是 new jQuery.fn.init()，而 var obj = new jQuery()，所以这 2 者是相当的，所以我们可以无 new 实例化 jQuery 对象。

#### ready实现

```
jQuery.extend({
    ready: function(wait) {
        // 如果需要等待，holdReady()的时候，把hold住的次数减1，如果还没到达0，说明还需要继续hold住，return掉
        // 如果不需要等待，判断是否已经Ready过了，如果已经ready过了，就不需要处理了。异步队列里边的done的回调都会执行了
        if (wait === true ? --jQuery.readyWait : jQuery.isReady) {
            return;
        }
 
        // 确定 body 存在
        if (!document.body) {
            // 如果 body 还不存在 ，DOMContentLoaded 未完成，此时
            // 将 jQuery.ready 放入定时器 setTimeout 中
            // 不带时间参数的 setTimeout(a) 相当于 setTimeout(a,0)
            // 但是这里并不是立即触发 jQuery.ready
            // 由于 javascript 的单线程的异步模式
            // setTimeout(jQuery.ready) 会等到重绘完成才执行代码，也就是 DOMContentLoaded 之后才执行 jQuery.ready
            // 所以这里有个小技巧：在 setTimeout 中触发的函数, 一定会在 DOM 准备完毕后触发
            return setTimeout(jQuery.ready);
        }
 
        // Remember that the DOM is ready
        // 记录 DOM ready 已经完成
        jQuery.isReady = true;
 
        // If a normal DOM Ready event fired, decrement, and wait if need be
        // wait 为 false 表示ready事情未触发过，否则 return
        if (wait !== true && --jQuery.readyWait > 0) {
            return;
        }
 
        // If there are functions bound, to execute
        // 调用异步队列，然后派发成功事件出去（最后使用done接收，把上下文切换成document，默认第一个参数是jQuery。
        readyList.resolveWith(document, [jQuery]);
 
        // Trigger any bound ready events
        // 最后jQuery还可以触发自己的ready事件
        // 例如：
        //    $(document).on('ready', fn2);
        //    $(document).ready(fn1);
        // 这里的fn1会先执行，自己的ready事件绑定的fn2回调后执行
        if (jQuery.fn.trigger) {
            jQuery(document).trigger("ready").off("ready");
        }
    }
})
```