# webpack核心模块解析

核心模块比较抽象，距离我们比较远，让我带你一步步到达

## （一）webpack中的钩子

webpack的运行分很多阶段，每个阶段都有一个钩子，我们可以在钩子上注册我们自定义的插件。
注册后，我们自定义的插件会在钩子运行阶段执行。例如以下钩子：

![](/imgs/webpackhooks.png)

webpack的钩子 [webpack.docschina.org/api/compile…](https://webpack.docschina.org/api/compiler-hooks/)

## （二）webpack中的插件定位

插件目的：在于解决 loader 无法实现的其他事。  

插件的重要性：是 webpack 的 **支柱** 功能，webpack 自身也是构建于插件系统之上！

## （三）webpack中自定义插件应用

（1）定义插件

```
pulgins/TestPlugin1.js(自定义文件地址)

/* eslint-disable no-console */
class TestPlugin1 {
  constructor(options) {
    console.log('options1:' + options)
  }
  apply(compiler) {
    compiler.hooks.entryOption.tap('TestPlugin1', (...args) => {
      console.log(args)
    })
  }
}

module.exports = TestPlugin1
复制代码
```

（2）使用插件

```
webpack.config.js（可以配置webpack拓展的地方）

const TestPlugin1 = require('plugins/TestPlugin1.js')
plugins: [
  new TestPlugin1('have a test'),
]
复制代码
```

## （四）插件类型

插件类型分为 tap、tapAsync、tapPromise
其中，tap是同步；tapAsync、tapPromise是异步
[webpack.docschina.org/api/plugins…](https://webpack.docschina.org/api/plugins/)

```
compiler.hooks.compile.tap('MyPlugin', params => {
  console.log('以同步方式触及 compile 钩子。');
})
compiler.hooks.run.tapAsync('MyPlugin', (source, target, routesList, callback) => {
  console.log('以异步方式触及 run 钩子。');
  callback();
});

compiler.hooks.run.tapPromise('MyPlugin', (source, target, routesList) => {
  return new Promise(resolve => setTimeout(resolve, 1000)).then(() => {
    console.log('以具有延迟的异步方式触及 run 钩子。');
  });
});
复制代码
```

## （五）核心模块tapable

话说，webpack的Compiler和Compilation都继承自tapable！  
tapable的基本用法如下，和webpack的钩子及插件基本一致。  

[github.com/webpack/tap…](https://github.com/webpack/tapable)

```
(1)取出钩子类  

const {
	SyncHook,
	SyncBailHook,
	SyncWaterfallHook,
	SyncLoopHook,
	AsyncParallelHook,
	AsyncParallelBailHook,
	AsyncSeriesHook,
	AsyncSeriesBailHook,
	AsyncSeriesWaterfallHook
 } = require("tapable");  
 
 (2)为插件创建钩子  
 const hook = new SyncHook(["arg1", "arg2", "arg3"]);  
 
 class Car {
	constructor() {
		this.hooks = {
			accelerate: new SyncHook(["newSpeed"]),
			brake: new SyncHook(),
			calculateRoutes: new AsyncParallelHook(["source", "target", "routesList"])
		};
	}

	/* ... */
}  

（3）给钩子创建一个消费者  

const myCar = new Car();  

// Use the tap method to add a consument
myCar.hooks.brake.tap("WarningLampPlugin", () => warningLamp.on());

(4)同步消费者、异步消费者  

myCar.hooks.calculateRoutes.tap("CachedRoutesPlugin", (source, target, routesList) => {
	const cachedRoute = cache.get(source, target);
	if(cachedRoute)
		routesList.add(cachedRoute);
})  

myCar.hooks.calculateRoutes.tapPromise("GoogleMapsPlugin", (source, target, routesList) => {
	// return a promise
	return google.maps.findRoute(source, target).then(route => {
		routesList.add(route);
	});
});
myCar.hooks.calculateRoutes.tapAsync("BingMapsPlugin", (source, target, routesList, callback) => {
	bing.findRoute(source, target, (err, route) => {
		if(err) return callback(err);
		routesList.add(route);
		// call the callback
		callback();
	});
});  

（5）声明这些钩子的类需要调用它们  

class Car {
	/**
	  * You won't get returned value from SyncHook or AsyncParallelHook,
	  * to do that, use SyncWaterfallHook and AsyncSeriesWaterfallHook respectively
	 **/

	setSpeed(newSpeed) {
		// following call returns undefined even when you returned values
		this.hooks.accelerate.call(newSpeed);
	}

	useNavigationSystemPromise(source, target) {
		const routesList = new List();
		return this.hooks.calculateRoutes.promise(source, target, routesList).then((res) => {
			// res is undefined for AsyncParallelHook
			return routesList.getRoutes();
		});
	}

	useNavigationSystemAsync(source, target, callback) {
		const routesList = new List();
		this.hooks.calculateRoutes.callAsync(source, target, routesList, err => {
			if(err) return callback(err);
			callback(null, routesList.getRoutes());
		});
	}
}  

```



## （六）tapable的源码解读

读完此篇，就解开了webpack的架构基石。

tap究竟是怎么运行的, 怎么处理,控制 tap 进去的钩子函数,拦截器又是怎么运行的.

先从同步函数开始分析,异步也就是回调而已;

## tap

这里有一个例子

```
let SyncHook = require('./lib/SyncHook.js')

let h1 = new SyncHook(['options']);

h1.tap('A', function (arg) {
  console.log('A',arg);
  return 'b'; // 除非你在拦截器上的 register 上调用这个函数,不然这个返回值你拿不到.
})

h1.tap('B', function () {
  console.log('b')
})
h1.tap('C', function () {
  console.log('c')
})
h1.tap('D', function () {
  console.log('d')
})

h1.intercept({
  call: (...args) => {
    console.log(...args, '-------------intercept call');
  },
  //
  register: (tap) => {
  console.log(tap, '------------------intercept register');

    return tap;
  },
  loop: (...args) => {
    console.log(...args, '-------------intercept loop')
  },
  tap: (tap) => {
    console.log(tap, '-------------------intercept tap')

  }
})
h1.call(6);
```

## `new SyncHook(['synchook'])`

首先先创建一个同步钩子对象,那这一步会干什么呢?

这一步会先执行超类Hook的初始化工作

```
// 初始化
constructor(args) {
  // 参数必须是数组
  if (!Array.isArray(args)) args = [];
  // 把数组参数赋值给 _args 内部属性, new 的时候传进来的一系列参数.
  this._args = args;
  // 绑定taps,应该是事件
  this.taps = [];
  // 拦截器数组
  this.interceptors = [];
  // 暴露出去用于调用同步钩子的函数
  this.call = this._call;
  // 暴露出去的用于调用异步promise函数
  this.promise = this._promise;
  // 暴露出去的用于调用异步钩子函数
  this.callAsync = this._callAsync;
  // 用于生成调用函数的时候,保存钩子数组的变量,现在暂时先不管.
  this._x = undefined;
}
```

## 第二部 `.tap()`

现在我们来看看调用了tap() 方法后发生了什么

```
tap(options, fn) {
  // 下面是一些参数的限制,第一个参数必须是字符串或者是带name属性的对象,
  // 用于标明钩子,并把钩子和名字都整合到 options 对象里面
  if (typeof options === "string") options = { name: options };
  if (typeof options !== "object" || options === null)
    throw new Error(
      "Invalid arguments to tap(options: Object, fn: function)"
    );
  options = Object.assign({ type: "sync", fn: fn }, options);
  if (typeof options.name !== "string" || options.name === "")
    throw new Error("Missing name for tap");
  // 注册拦截器
  options = this._runRegisterInterceptors(options);
  // 插入钩子
  this._insert(options);
}
```

- 现在我们来看看如何注册拦截器

```
_runRegisterInterceptors(options) {
  // 现在这个参数应该是这个样子的{fn: function..., type: sync,name: 'A' }
// 遍历拦截器,有就应用,没有就把配置返还回去
for (const interceptor of this.interceptors) {
  if (interceptor.register) {
    // 把选项传入拦截器注册,从这里可以看出,拦截器的register 可以返回一个新的options选项,并且替换掉原来的options选项,也就是说可以在执行了一次register之后 改变你当初 tap 进去的方法
    const newOptions = interceptor.register(options);
    if (newOptions !== undefined) options = newOptions;
  }
}
return options;
}
```

**注意**: 这里执行的register拦截器是有顺序问题的, 这个执行在tap()里面,也就是说,你这个拦截器要在调用tap(),之前就调用 intercept()添加的.

那拦截器是怎么添加进去的呢,来看下intercept()

```
intercept(interceptor) {
  // 重置所有的 调用 方法,在教程中我们提到了 编译出来的调用方法依赖的其中一点就是 拦截器. 所有每添加一个拦截器都要重置一次调用方法,在下一次编译的时候,重新生成.
  this._resetCompilation();
  // 保存拦截器 而且是复制一份,保留原本的引用
  this.interceptors.push(Object.assign({}, interceptor));
  // 运行所有的拦截器的register函数并且把 taps[i],(tap对象) 传进去.
  // 在intercept 的时候也会遍历执行一次当前所有的taps,把他们作为参数调用拦截器的register,并且把返回的 tap对象(tap对象就是指 tap函数里面把fn和name这些信息整合起来的那个对象) 替换了原来的 tap对象,所以register最好返回一个tap, 在例子中我返回了原来的tap, 但是其实最好返回一个全新的tap
  if (interceptor.register) {
    for (let i = 0; i < this.taps.length; i++)
      this.taps[i] = interceptor.register(this.taps[i]);
  }
}
```

**注意**: 也就是在调用tap() 之后再传入的拦截器,会在传入的时候就为每一个tap 调用register方法

- 现在我们来看看_insert

```
_insert(item) {
  // 重置资源,因为每一个插件都会有一个新的Compilation
  this._resetCompilation();
  // 顺序标记, 这里联合 __test__ 包里的Hook.js一起使用
  // 看源码不懂,可以看他的测试代码,就知道他写的是什么目的.
  // 从测试代码可以看到,这个 {before}是插件的名字.
  let before;
  // before 可以是单个字符串插件名称,也可以是一个字符串数组插件.
  if (typeof item.before === "string") {
    before = new Set([item.before]);
  }
  else if (Array.isArray(item.before)) {
    before = new Set(item.before);
  }
  // 阶段
  // 从测试代码可以知道这个也是一个控制顺序的属性,值越小,执行得就越在前面
  // 而且优先级低于 before
  let stage = 0;
  if (typeof item.stage === "number") stage = item.stage;
  let i = this.taps.length;
  // 遍历所有`tap`了的函数,然后根据 stage 和 before 进行重新排序.
  // 假设现在tap了 两个钩子  A B  `B` 的配置是  {name: 'B', before: 'A'}
  while (i > 0) {// i = 1, taps = [A]
    i--;// i = 0 首先-- 是因为要从最后一个开始
    const x = this.taps[i];// x = A
    this.taps[i + 1] = x;// i = 0, taps[1] = A  i+1 把当前元素往后移位,把位置让出来
    const xStage = x.stage || 0;// xStage = 0
    if (before) {// 如果有这个属性就会进入这个判断
      if (before.has(x.name)) {// 如果before 有x.name 就会把这个插件名称从before这个列表里删除,代表这个钩子位置已经在当前的钩子之前
        before.delete(x.name);
        continue;// 如果before还有元素,继续循环,执行上面的操作
      }
      if (before.size > 0) {
        continue;// 如果before还有元素,那就一直循环,直到第一位.
      }
    }
    if (xStage > stage) {// 如果stage比当前钩子的stage大,继续往前挪
      continue;
    }
    i++;
    break;
  }
  this.taps[i] = item;// 把挪出来的位置插入传进来的钩子
}
```

这其实就是一个排序算法, 根据before, stage 的值来排序,也就是说你可以这样tap进来一个插件

```
h1.tap({
  name: 'B',
  before: 'A'
  }, () => {
    console.log('i am B')
  })
```

## 发布订阅模式

发布订阅模式是一个在前后端都盛行的一个模式,前端的promise,事件,等等都基于发布订阅模式,其实tapable 也是一种发布订阅模式,上面的tap 只是订阅了钩子函数,我们还需要发布他,接下来我们谈谈`h1.call()`,跟紧了,这里面才是重点.

我们可以在初始化中看到`this.call = this._call`,那我们来看一下 this._call() 是个啥

```
Object.defineProperties(Hook.prototype, {
  _call: {
    value: createCompileDelegate("call", "sync"),
    configurable: true,
    writable: true
  },
  _promise: {
    value: createCompileDelegate("promise", "promise"),
    configurable: true,
    writable: true
  },
  _callAsync: {
    value: createCompileDelegate("callAsync", "async"),
    configurable: true,
    writable: true
  }
});
```

结果很明显,这个函数是由createCompileDelegate(),这个函数返回的,依赖于,函数的名字以及钩子的类型.

## `createCompileDelegate(name, type)`

```
function createCompileDelegate(name, type) {
  return function lazyCompileHook(...args) {
    // 子类调用时,this默认绑定到子类
    // (不明白的可以了解js this指向,一个函数的this指向调用他的对象,没有就是全局,除非使用call apply bind 等改变指向)
    // 在我们的例子中,这个 this 是 SyncHook
    this[name] = this._createCall(type);
    // 用args 去调用Call
    return this[name](...args);
  };
}
```

在上面的注释上可以加到,他通过闭包保存了`name`跟`type`的值,在我们这个例子中,这里就是`this.call = this._createCall('sync');`然后把我们外部调用call(666) 时 传入的参数给到他编译生成的方法中.

**注意**,在我们这个例子当中我在call的时候并没有传入参数.

这时候这个`call`方法的重点就在`_createCall`方法里面了.

## _createCall()

```
_createCall(type) {

  // 传递一个整合了各个依赖条件的对象给子类的compile方法
  return this.compile({
    taps: this.taps,
    interceptors: this.interceptors,
    args: this._args,
    type: type
  });
}
```

从一开始,我们就在Hook.js上分析,我们来看看Hook上的compile

```
compile(options) {
  throw new Error("Abstract: should be overriden");
}
```

清晰明了,这个方法一定要子类复写,不然报错,上面的`_createCompileDelegate`的注释也写得很清楚,在当前的上下文中,this指向的是,子类,在我们这个例子中就是`SyncHook`

## 来看看`SyncHook` 的compile

```
compile(options) {
  // 现在options 是由Hook里面 传到这里的
  // options
  // {
  //  taps: this.taps, tap对象数组
  //  interceptors: this.interceptors, 拦截器数组
  //  args: this._args,
  //  type: type
  // }
  // 对应回教程中的编译出来的调用函数依赖于的那几项看看,是不是这些,钩子的个数,new SyncHook(['arg'])的参数个数,拦截器的个数,钩子的类型.
  factory.setup(this, options);

  return factory.create(options);
}
```

好吧 现在来看看setup, 咦? factory 怎么来的,原来

```
const factory = new SyncHookCodeFactory();
```

是new 出来的

## 现在来看看SyncHookCodeFactory 的父类 HookCodeFactory

```
constructor(config) {

  // 这个config作用暂定.因为我看了这个文件,没看到有引用的地方,
  // 应该是其他子类有引用到
  this.config = config;
  // 这两个不难懂, 往下看就知道了
  this.options = undefined;
  this._args = undefined;
}
```

## 现在可以来看一下setup了

```
setup(instance, options) {
  // 这里的instance 是syncHook 实例, 其实就是把tap进来的钩子数组给到钩子的_x属性里.
  instance._x = options.taps.map(t => t.fn);
}
```

OK, 到create了

## 这个create有点长, 看仔细了,我们现在分析同步的部分.

```
create(options) {
  // 初始化参数,保存options到本对象this.options,保存new Hook(["options"]) 传入的参数到 this._args
  this.init(options);
  let fn;
  // 动态构建钩子,这里是抽象层,分同步, 异步, promise
  switch (this.options.type) {
    // 先看同步
    case "sync":
      // 动态返回一个钩子函数
      fn = new Function(
        // 生成函数的参数,no before no after 返回参数字符串 xxx,xxx 在
        // 注意这里this.args返回的是一个字符串,
        // 在这个例子中是options
        this.args(),
        '"use strict";\n' +
          this.header() +
          this.content({
            onError: err => `throw ${err};\n`,
            onResult: result => `return ${result};\n`,
            onDone: () => "",
            rethrowIfPossible: true
          })
      );
      break;
    case "async":
      fn = new Function(
        this.args({
          after: "_callback"
        }),
        '"use strict";\n' +
          this.header() +
          // 这个 content 调用的是子类类的 content 函数,
          // 参数由子类传,实际返回的是 this.callTapsSeries() 返回的类容
          this.content({
            onError: err => `_callback(${err});\n`,
            onResult: result => `_callback(null, ${result});\n`,
            onDone: () => "_callback();\n"
          })
      );
      break;
    case "promise":
      let code = "";
      code += '"use strict";\n';
      code += "return new Promise((_resolve, _reject) => {\n";
      code += "var _sync = true;\n";
      code += this.header();
      code += this.content({
        onError: err => {
          let code = "";
          code += "if(_sync)\n";
          code += `_resolve(Promise.resolve().then(() => { throw ${err}; }));\n`;
          code += "else\n";
          code += `_reject(${err});\n`;
          return code;
        },
        onResult: result => `_resolve(${result});\n`,
        onDone: () => "_resolve();\n"
      });
      code += "_sync = false;\n";
      code += "});\n";
      fn = new Function(this.args(), code);
      break;
  }
  // 把刚才init赋的值初始化为undefined
  // this.options = undefined;
  // this._args = undefined;
  this.deinit();

  return fn;
}
```

到了这个方法,一切我们都一目了然了(看content的参数), 在我们的例子中他是通过动态的生成一个call方法,根据的条件有,钩子是否有context 属性(这个是根据header的代码才能知道), 钩子的个数, 钩子的类型,钩子的参数,钩子的拦截器个数.

**注意**,这上面有关于 fn这个变量的函数,返回的都是字符串,不是函数不是方法,是返回可以转化成代码执行的字符串,思维要转变过来.

现在我们来看看header()

```
header() {
  let code = "";
  // this.needContext() 判断taps[i] 是否 有context 属性, 任意一个tap有 都会返回 true
  if (this.needContext()) {
    // 如果有context 属性, 那_context这个变量就是一个空的对象.
    code += "var _context = {};\n";
  } else {
    // 否则 就是undefined
    code += "var _context;\n";
  }
  // 在setup()中 把所有tap对象的钩子 都给到了 instance ,这里的this 就是setup 中的instance _x 就是钩子对象数组
  code += "var _x = this._x;\n";
  // 如果有拦截器,在我们的例子中,就有一个拦截器
  if (this.options.interceptors.length > 0) {
    // 保存taps 数组到_taps变量, 保存拦截器数组 到变量_interceptors
    code += "var _taps = this.taps;\n";
    code += "var _interceptors = this.interceptors;\n";
  }
  // 如果没有拦截器, 这里也不会执行.一个拦截器只会生成一次call
  // 在我们的例子中,就有一个拦截器,就有call
  for (let i = 0; i < this.options.interceptors.length; i++) {
    const interceptor = this.options.interceptors[i];
    if (interceptor.call) {
      // getInterceptor 返回的 是字符串 是 `_interceptors[i]`
      // 后面的before 因为我们的拦截器没有context 所以返回的是undefined 所以后面没有跟一个空对象
      code += `${this.getInterceptor(i)}.call(${this.args({
        before: interceptor.context ? "_context" : undefined
      })});\n`;
    }
  }
  return code;
  // 注意 header 返回的不是代码,是可以转化成代码的字符串(这个时候并没有执行).
  /**
    * 此时call函数应该为:
    * "use strict";
    * function (options) {
    *   var _context;
    *   var _x = this._x;
    *   var _taps = this.taps;
    *   var _interterceptors = this.interceptors;
    * // 我们只有一个拦截器所以下面的只会生成一个
    *   _interceptors[0].call(options);
    *}
    */
}
```

现在到我们的`this.content()`了,仔细一看,`this.content()`方法并不在`HookCodeFactory`上,很明显这个content是由子类来实现的,往回看看这个create是由谁调用的?没错,是SuncHookCodeFactory的石料理,我们来看看`SyncHook.js`上的`SyncHookCodeFactory`实现的`content`

在看这个content实现之前,先来回顾一下父类的`create()`给他传了什么参数.

```
this.content({
  onError: err => `throw ${err};\n`,
  onResult: result => `return ${result};\n`,
  onDone: () => "",
  rethrowIfPossible: true
})
```

**注意了,这上面不是抛出错误,不是返回值. 这里面的回调执行了以后返回的是一个字符串,不要搞混了代码与可以转化成代码的字符串.**

```
content({ onError, onResult, onDone, rethrowIfPossible }) {
  return this.callTapsSeries({
    // 可以在这改变onError 但是这里的 i 并没有用到,这是什么操作...
    // 注意这里并没有传入onResult
    onError: (i, err) => onError(err),
    onDone,
    // 这个默认为true
    rethrowIfPossible
  });
}
```

这个函数返回什么取决于this.callTapSeries(), 那接下来我们来看看这个函数(这层层嵌套,其实也是有可斟酌的地方.看源码不仅要看实现,代码的组织也是很重要的编码能力)

刚才函数的头部已经出来了,头部做了初始化的操作,与生成执行拦截器代码.content很明显,要开始生成执行我们的tap对象的代码了(如果不然,我们的tap进来的函数在哪里执行呢? 滑稽:).

```
callTapsSeries({ onError, onResult, onDone, rethrowIfPossible }) {
  // 如果 taps 钩子处理完毕,执行onDone,或者一个tap都没有 onDone() 返回的是一个字符串.看上面的回顾就知道了.
  if (this.options.taps.length === 0) return onDone();
  // 如果由异步钩子,把第一个异步钩子的下标,如果没有这个返回的是-1
  const firstAsync = this.options.taps.findIndex(t => t.type !== "sync");
  // 定义一个函数 接受一个 number 类型的参数, i 应该是taps的index
  // 从这个函数的命名来看,这个函数应该会递归的执行
  // 我们先开最后的return语句,发现第一个传进来的参数是0
  const next = i => {
    // 如果 大于等于钩子函数数组长度, 返回并执行onDone回调,就是tap对象都处理完了
    // 跳出递归的条件
    if (i >= this.options.taps.length) {
      return onDone();
    }
    // 这个方法就是递归的关键,看见没,逐渐往上遍历
    // 注意这里只是定义了方法,并没有执行
    const done = () => next(i + 1);
    // 传入一个值 如果是false 就执行onDone true 返回一个 ""
    // 字面意思,是否跳过done 应该是增加一个跳出递归的条件
    const doneBreak = skipDone => {
      if (skipDone) return "";
      return onDone();
    };
    // 这里就是处理单个taps对象的关键,传入一个下标,和一系列回调.
    return this.callTap(i, {
      // 调用的onError 是 (i, err) => onError(err) , 后面这个onError(err)是 () => `throw ${err}`
      // 目前 i done doneBreak 都没有用到
      onError: error => onError(i, error, done, doneBreak),
      // 这里onResult 同步钩子的情况下在外部是没有传进来的,刚才也提到了
      // 这里onResult是 undefined
      onResult:
        onResult &&
        (result => {
          return onResult(i, result, done, doneBreak);
        }),
      // 没有onResult 一定要有一个onDone 所以这里就是一个默认的完成回调
      // 这里的done 执行的是next(i+1), 也就是迭代的处理完所有的taps
      onDone:
        !onResult &&
        (() => {return done();}),
      // rethrowIfPossible 默认是 true 也就是返回后面的
      // 因为没有异步函数 firstAsync = -1.
      // 所以返回的是 -1 < 0,也就是true, 这个可以判断当前的是否是异步的tap对象
      //  这里挺妙的 如果是 false 那么当前的钩子类型就不是sync,可能是promise或者是async
      // 具体作用要看callTaps()如何使用这个.
      rethrowIfPossible:
        rethrowIfPossible && (firstAsync < 0 || i < firstAsync)
    });
  };
  
  return next(0);
}
```

## 参数搞明白了,现在,我们可以进入`callTap()` 了.

`callTap`挺长的,因为他也分了3种类型分别处理,像create()一样.

```
/** tapIndex 下标
  * onError:() => onError(i,err,done,skipdone) ,
  * onReslt: undefined
  * onDone: () => {return: done()} //开启递归的钥匙
  * rethrowIfPossible: false 说明当前的钩子不是sync的.
  */
callTap(tapIndex, { onError, onResult, onDone, rethrowIfPossible }) {
  let code = "";
  // hasTapCached 是否有tap的缓存, 这个要看看他是怎么做的缓存了
  let hasTapCached = false;
  // 这里还是拦截器的用法,如果有就执行拦截器的tap函数
  for (let i = 0; i < this.options.interceptors.length; i++) {
    const interceptor = this.options.interceptors[i];
    if (interceptor.tap) {
      if (!hasTapCached) {
        // 这里getTap返回的是 _taps[0] _taps[1]... 的字符串
        // 这里生成的代码就是 `var _tap0 = _taps[0]`
        // 注意: _taps 变量我们在 header 那里已经生成了
        code += `var _tap${tapIndex} = ${this.getTap(tapIndex)};\n`;
        // 可以看到这个变量的作用就是,如果有多个拦截器.这里也只会执行一次.
        // 注意这句获取_taps 对象的下标用的是tapIndex,在一次循环中,这个tapIndex不会变
        // 就是说如果这里执行多次,就会生成多个重复代码,不稳定,也影响性能.
        // 但是你又要判断拦截器有没有tap才可以执行,或许有更好的写法
        // 如果你能想到,那么你就是webpack的贡献者了.不过这样写,似乎也没什么不好.
        hasTapCached = true;
      }
      // 这里很明显跟上面的getTap 一样 返回的都是字符串
      // 我就直接把这里的code 分析出来了,注意 这里还是在循坏中.
      // code += _interceptor[0].tap(_tap0);
      // 由于我们的拦截器没有context,所以没传_context进来.
      // 可以看到这里是调用拦截器的tap方法然后传入tap0对象的地方
      code += `${this.getInterceptor(i)}.tap(${
        interceptor.context ? "_context, " : ""
      }_tap${tapIndex});\n`;
    }
  }
  // 跑出了循坏
  // 这里的getTapFn 返回的也是字符串 `_x[0]`
  // callTap用到的这些全部在header() 那里生成了,忘记的回头看一下.
  // 这里的code就是: var _fn0 = _x[0]
  code += `var _fn${tapIndex} = ${this.getTapFn(tapIndex)};\n`;
  const tap = this.options.taps[tapIndex];
  // 开始处理tap 对象
  switch (tap.type) {
    case "sync":
      // 全是同步的时候, 这里不执行, 如果有异步函数,那么恭喜,有可能会报错.所以他加了个 try...catch
      if (!rethrowIfPossible) {
        code += `var _hasError${tapIndex} = false;\n`;
        code += "try {\n";
      }
      // 前面分析了 同步的时候 onResult 是 undefined
      // 我们也分析一下如果走这里会怎样
      // var _result0 = _fn0(option)
      // 可以看到是调用tap 进来的钩子并且接收参数
      if (onResult) {
        code += `var _result${tapIndex} = _fn${tapIndex}(${this.args({
          before: tap.context ? "_context" : undefined
        })});\n`;
      } else {
        // 所以会走这里
        // _fn0(options) 额... 我日 有就接受一下结果
        code += `_fn${tapIndex}(${this.args({
          before: tap.context ? "_context" : undefined
        })});\n`;
      }
      // 把 catch 补上,在这个例子中没有
      if (!rethrowIfPossible) {
        code += "} catch(_err) {\n";
        code += `_hasError${tapIndex} = true;\n`;
        code += onError("_err");
        code += "}\n";
        code += `if(!_hasError${tapIndex}) {\n`;
      }
      // 有onResult 就把结果给传递出去. 目前没有
      if (onResult) {
        code += onResult(`_result${tapIndex}`);
      }
      // 有onDone() 就调用他开始递归,还记得上面的next(i+1) 吗?
      if (onDone) {
        code += onDone();
      }
      // 这里是不上上面的if的大括号,在这个例子中没有,所以这里也不执行
      if (!rethrowIfPossible) {
        code += "}\n";
      }
      // 同步情况下, 这里最终的代码就是
      // var _tap0 = _taps[0];
      // _interceptors[0].tap(_tap0);
      // var _fn0 = _x[0];
      // _fn0(options);
      // 可以看到,这里会递归下去
      // 因为我们tap了4个钩子
      // 所以这里会从复4次
      // 最终长这样
      // var _tap0 = _taps[0];
      // _interceptors[0].tap(_tap0);
      // var _fn0 = _x[0];
      // _fn0(options);
      // var _tap1 = _taps[1];
      // _interceptors[1].tap(_tap1);
      // var _fn1 = _x[1];
      // _fn1(options);
      // ......
      break;
    case "async":
      let cbCode = "";
      if (onResult) cbCode += `(_err${tapIndex}, _result${tapIndex}) => {\n`;
      else cbCode += `_err${tapIndex} => {\n`;
      cbCode += `if(_err${tapIndex}) {\n`;
      cbCode += onError(`_err${tapIndex}`);
      cbCode += "} else {\n";
      if (onResult) {
        cbCode += onResult(`_result${tapIndex}`);
      }
      if (onDone) {
        cbCode += onDone();
      }
      cbCode += "}\n";
      cbCode += "}";
      code += `_fn${tapIndex}(${this.args({
        before: tap.context ? "_context" : undefined,
        after: cbCode
      })});\n`;
      break;
    case "promise":
      code += `var _hasResult${tapIndex} = false;\n`;
      code += `var _promise${tapIndex} = _fn${tapIndex}(${this.args({
        before: tap.context ? "_context" : undefined
      })});\n`;
      code += `if (!_promise${tapIndex} || !_promise${tapIndex}.then)\n`;
      code += `  throw new Error('Tap function (tapPromise) did not return promise (returned ' + _promise${tapIndex} + ')');\n`;
      code += `_promise${tapIndex}.then(_result${tapIndex} => {\n`;
      code += `_hasResult${tapIndex} = true;\n`;
      if (onResult) {
        code += onResult(`_result${tapIndex}`);
      }
      if (onDone) {
        code += onDone();
      }
      code += `}, _err${tapIndex} => {\n`;
      code += `if(_hasResult${tapIndex}) throw _err${tapIndex};\n`;
      code += onError(`_err${tapIndex}`);
      code += "});\n";
      break;
  }
  return code;
}
```

**好了, 到了这里 我们可以把compile 出来的call 方法输出出来了**

```
"use strict";
function (options) {
  var _context;
  var _x = this._x;
  var _taps = this.taps;
  var _interterceptors = this.interceptors;
// 我们只有一个拦截器所以下面的只会生成一个
  _interceptors[0].call(options);

  var _tap0 = _taps[0];
  _interceptors[0].tap(_tap0);
  var _fn0 = _x[0];
  _fn0(options);
  var _tap1 = _taps[1];
  _interceptors[1].tap(_tap1);
  var _fn1 = _x[1];
  _fn1(options);
  var _tap2 = _taps[2];
  _interceptors[2].tap(_tap2);
  var _fn2 = _x[2];
  _fn2(options);
  var _tap3 = _taps[3];
  _interceptors[3].tap(_tap3);
  var _fn3 = _x[3];
  _fn3(options);
}
```

到了这里可以知道,我们的例子中`h1.call()`其实调用的就是这个方法.到此我们可以说是知道了这个库的百分之80了.

不知道大家有没有发现,这个生成的函数的参数列表是从哪里来的呢?往回翻到create()方法里面调用的`this.args()`你就会看见,没错就是this._args. 这个东西在哪里初始化呢? 翻一下就知道,这是在`Hook.js`这个类里面初始化的,也就是说你`h1 = new xxxHook(['options'])` 的时候传入的数组有几个值,那么你`h1.call({name: 'haha'})` 就能传几个值.看教程的时候他说,这里传入的是一个参数名字的字符串列表,那时候我就纳闷,什么鬼,我传入的不是值吗,怎么就变成了参数名称,现在完全掌握....

好了,最简单的`SyncHook` 已经搞掂,但是一看`tapable`内部核心使用的钩子却不是他,而是`SyncBailHook`,在教程中我们已经知道,`bail`是只要有一个钩子执行完了,并且返回一个值,那么其他的钩子就不执行.我们来看看他是怎么实现的.

**从刚才我们弄明白的synchook,我们知道了他的套路,其实生成的函数的header()都是一样的,这次我们直接来看看bailhook实现的content()方法**

```
content({ onError, onResult, onDone, rethrowIfPossible }) {
  return this.callTapsSeries({
    onError: (i, err) => onError(err),
  // 看回callTapsSeries 就知道这里传入的next 是 done
    onResult: (i, result, next) =>
      `if(${result} !== undefined) {\n${onResult(
        result
      )};\n} else {\n${next()}}\n`,
    onDone,
    rethrowIfPossible
  });
}
```

看出来了哪里不一样吗? 是的`bailhook`的 `callTapsSeries`传了`onResult`属性,我们来看看他这个onResult是啥黑科技

父类传的`onResult`默认是 `(result) => 'return ${result}'`,那么他这里返回的就是:

```
// 下面返回的是字符串,
if (xxx !== undefined) {
  // 这里说明,只要有返回值(因为不返回默认是undefined),就会立即return;
  return result;
} else {
  // next(); 这里返回的是一个字符串(因为要生成字符串代码)
  // 我在上面的注释中提到了 next 是 done 就是那个开启递归的门
  // 所以如果tap 一直没返回值, 这里就会一直 if...else.. 的嵌套下去
  
}
```

回头想想,我们刚刚是不是分析了`callTap()`,如果我们传了`onResult` 会怎样? 如果你还记得就知道,如果有传了`onResult`这个回调,他就会接收这个返回值.并且会调用这个回调把`result`传出去.

而且还要注意的是,`onDone`在`callTap()`的时候是处理过的,我在贴出来一次.

```
onDone:!onResult && (() => {return done();})
```

也就是说如果我传了`onResult` 那么这个`onDone`就是一个`false`.

所以递归的门现在从`sync`的`onDone`,变到`syncBail`的`onResult`了

好,现在带着这些变化去看`this.callTap()`,你就能推出现在这个 call 函数会变成这样.

```
"use strict";
function (options) {
  var _context;
  var _x = this._x;
  var _taps = this.taps;
  var _interterceptors = this.interceptors;
// 我们只有一个拦截器所以下面的只会生成一个
  _interceptors[0].call(options);

  var _tap0 = _taps[0];
  _interceptors[0].tap(_tap0);
  var _fn0 = _x[0];
  var _result0 = _fn0(options);

  if (_result0 !== undefined) {
    // 这里说明,只要有返回值(因为不返回默认是undefined),就会立即return;
    return _result0
  } else {
    var _tap1 = _taps[1];
    _interceptors[1].tap(_tap1);
    var _fn1 = _x[1];
    var _result1 = _fn1(options);
    if (_result1 !== undefined) {
      return _result1
    } else {
      var _tap2 = _taps[2];
      _interceptors[2].tap(_tap2);
      var _fn2 = _x[2];
      var _result2 = _fn2(options);
      if (_result2 !== undefined) {
        return _result2
      } else {
        var _tap3 = _taps[3];
        _interceptors[3].tap(_tap3);
        var _fn3 = _x[3];
        _fn3(options);
      }
    }
  }
```

到如今,tapable库 已经删除了 tapable.js文件(可能做了一些整合,更细分了),只留下了钩子文件.但不影响功能,webpack 里的`compile` `compilation` 等一众重要插件,都是基于 tapable库中的这些钩子.

现在我们require('tapable')得到的对象是这样的:

```
{
    SyncHook: function(...){},
    SyncBailHook: function(...){},
    ...
}
```