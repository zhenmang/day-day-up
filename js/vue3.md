vue2.x

```
npm install vue-cli -g
vue init webpack my_project
npm i
npm run dev
```

vue3.0 Beta

春

2020年4月19日凌晨4点左右，vue-next v3.0.0-beta.1发布

```
npm install -g @vue/cli
vue -V
vue create my_project 或者 vue create -r https://registry.npm.taobao.org my_project
// 升级为Vue 3
cd my_project
vue add vue-next
npm run serve
```

秋

2020 年 9 月 18 日晚 11 点半， Vue 3.0 版本发布

Scaffold via [Vite](https://github.com/vitejs/vite):

```
npm init vite-app hello-vue3 # OR yarn create vite-app hello-vue3
```

Scaffold via [vue-cli](https://cli.vuejs.org/):

```
npm install -g @vue/cli # OR yarn global add @vue/cli
vue create hello-vue3
# select vue 3 preset
```



### **性能提升**

Vue 3 与 Vue 2 相比，在 bundle 包大小方面（tree-shaking 减少了 41% 的体积），初始渲染速度方面（快了 55%），更新速度方面（快了 133%）以及内存占用方面（减少了 54%）都有着**显著的性能提升**。



## vue全家桶

### 1、实例化

2.x使用构造函数`new Vue(...)`创建实例，3.x使用`createApp`函数创建实例；

2.x所有属性方法和设置都绑定到全局`Vue`对象上，3.x改为绑定到`vue`实例下，收紧了scope；

3.x移除了 `Vue.config.productionTip` 和 `Vue.config.keyCodes` 配置属性；

```
// vue 2.x
import Vue from 'vue'
import App from './App'
import router from './router'
import store from './store'

Vue.config.ignoredElements = [/^app-/]
Vue.use(/* ... */)
Vue.mixin(/* ... */)
Vue.component(/* ... */)
Vue.directive(/* ... */)
Vue.prototype.customProperty = () => {}

new Vue({
  el: '#app',
  router,
  store,
  render: h => h(App)
})
复制代码
```

\--

```
// vue 3.x
import { createApp } from 'vue'
import App from './App.vue'
import router from './router'
import store from './store'

const app = createApp(App)

app.config.isCustomElement = tag => tag.startsWith('app-')
app.use(/* ... */)
app.mixin(/* ... */)
app.component(/* ... */)
app.directive(/* ... */)
app.config.globalProperties.customProperty = () => {}

app.use(router).use(store).mount('#app')
复制代码
```

### 2、创建页面

在/src/views目录中新建Test.vue

```
<template>
  <div class="page-wrapper">
    <span>这是一个新页面</span>
  </div>
</template>

<script>
export default {
  name: 'Test',
  setup () {
    return {}
  }
}
</script>
复制代码
```

在 /src/router/index.js中创建路由

```
import { createRouter, createWebHistory } from 'vue-router'
import Home from '../views/Home.vue'

const routes = [
  {
    path: '/',
    name: 'Home',
    component: Home
  },
  {
    path: '/about',
    name: 'About',
    // route level code-splitting
    // this generates a separate chunk (about.[hash].js) for this route
    // which is lazy-loaded when the route is visited.
    component: () => import(/* webpackChunkName: "about" */ '../views/About.vue')
  },
  {
    path: '/test',
    name: 'Test',
    component: () => import(/* webpackChunkName: "test" */ '../views/Test.vue')
  }
]

const router = createRouter({
  history: createWebHistory(process.env.BASE_URL),
  routes
})

export default router
复制代码
```

### 3、Composition API

vue2.x中，所有的数据都在`data`方法中定义返回，方法定义在`methods`下面，并通过`this`调用vue3.x中，所有的代码逻辑将在`setup`方法中实现，包括`data`、`watch`、`computed`、`methods`、`hooks`，并且不再有`this`

vue3.x `setup`方法在组件生命周期内只执行一次，不会重复执行

相比vue2.x中基于`OPTIONS`配置的方式，vue3.x基于组合式API的方式语义没有2.x清晰，2.x中`data`、`methods`、`computed`、`watch`等都通过不同的scope区分开，看起来很清晰，3.x都放在`setup`方法中，对代码组织能力会有更高的要求。

> vue2.x使用Composition API可以安装@vue/composition-api，使用基本跟Composition API一样，这里不再赘述

#### ① 状态和事件绑定 reactive & ref

`reactive` 几乎等价于 2.x 中的 `Vue.observable()` API，只是为了避免与 `RxJS` 中的 `observable` 混淆而做了重命名

vue3.x的`reactive`和`ref`取代了vue2.x中的`data`数据定义

从下面的代码中可以看到，`reactive`处理的是对象的双向绑定，而`ref`则可以处理js基础类型的双向绑定，其实`ref`的实现原理也只是对基础类型进行对象化封装，把数据放在{ value: 基础值 }里，再添加一个ref标识属性用来区分。

```
// vue2.x
export default {
  name: 'Test',
  data () {
    return {
      count: 0,
      num: 0
    }
  },
  methods: {
    addCount () {
      this.count++
    }
    addNum() {
      this.num++
    }
  }
}
复制代码
```

**----**

```
// vue3.x
<template>
  <div class="page-wrapper">
    <div>
      <span>count 点击次数: </span>
      <span>{{ count }}</span>
      <button @click="addCount">点击增加</button>
    </div>
    <div>
      <span>num 点击次数: </span>
      <span>{{ num }}</span>
      <button @click="addNum">点击增加</button>
    </div>
  </div>
</template>

<script>
import { reactive, ref, toRefs } from 'vue'

export default {
  name: 'Test',
  setup () {
    const state = reactive({
      count: 0
    })
    const num = ref(0)

    const addCount = function () {
      state.count++
    }
    const addNum = function () {
      num.value++
    }

    return {
      // 这样展开后state property会失去响应式，因为是取值返回，不是引用
      // ...state,
      ...toRefs(state),
      num,
      addCount,
      addNum
    }
  }
}
</script>
复制代码
```

**解开 Ref**

我们可以将一个 `ref` 值暴露给渲染上下文，在渲染过程中，Vue 会直接使用其内部的值，也就是说在模板中你可以把 `{{ num.value }}` 直接写为 `{{ num }}` ，但是在js中还是需要通过 `num.value`取值和赋值。

**使用 Reactive**

使用 `reactive` 组合函数时必须始终保持对这个所返回对象的引用以保持响应性。这个对象不能被解构或展开，一旦被解构或者展开，返回的值将失去响应式。

`toRefs` API 用来提供解决此约束的办法——它将响应式对象的每个 property 都转成了相应的 `ref`。

#### ② 只读数据 readonly

对于不允许写的对象，不管是普通`object`对象、`reactive`对象、`ref`对象，都可以通过`readonly`方法返回一个只读对象

直接修改`readonly`对象，控制台会打印告警信息，不会报错

```
const state = reactive({
  count: 0
})
const readonlyState = readonly(state)
// 监听只读属性，state.count修改后依然会触发readonlyState.count更新
watch(() => readonlyState.count, (newVal, oldVal) => {
  console.log('readonly state is changed!')
  setTimeout(() => {
    // 修改只读属性会打印告警信息，但是不会报错
    readonlyState.count = 666
  }, 1000)
})
复制代码
```

#### ③ 计算属性 computed

2.x和3.x中的`computed`都支持getter和setter，写法一样，只是3.x中是组合函数式

```
// vue2.x
export default {
  ...
  computed: {
    totalCount() {
      return this.count + this.num
    },
    doubleCount: {
      get() {
        return this.count * 2
      },
      set(newVal) {
        this.count = newVal / 2
      }
    }
  }
}
复制代码
```

\--

```
// vue3.x
import { reactive, ref, toRefs, computed } from 'vue'

export default {
  name: 'Test',
  setup () {
    const state = reactive({
      count: 0,
      double: computed(() => {
        return state.count * 2
      })
    })
    const num = ref(0)

    const addCount = function () {
      state.count++
    }
    const addNum = function () {
      num.value++
    }

    // only getter
    const totalCount = computed(() => state.count + num.value)
    // getter & setter
    const doubleCount = computed({
      get () {
        return state.count * 2
      },
      set (newVal) {
        state.count = newVal / 2
      }
    })

    return {
      ...toRefs(state),
      num,
      totalCount,
      doubleCount,
      addCount,
      addNum
    }
  }
}
复制代码
```

#### ④ 监听属性 watch & watchEffect

3.x和2.x的`watch`一样，支持`immediate`和`deep`选项，但3.x不再支持`'obj.key1.key2'`的"点分隔"写法；

3.x中`watch`支持监听单个属性，也支持监听多个属性，相比2.x的`watch`更灵活；

3.x中`watchEffect`方法会返回一个方法，用于停止监听；

`watch`跟`watchEffect`不同的地方在于，`watchEffect`注册后会立即调用，而`watch`默认不会，除非显示指定`immediate=true`，并且`watchEffect`可以停止监听

> 在 DOM 当中渲染内容会被视为一种“副作用”：程序会在外部修改其本身 (也就是这个 DOM) 的状态。我们可以使用 `watchEffect` API 应用基于响应式状态的副作用，并自动进行重应用。

```
// vue2.x
export default {
  ...
  data () {
    return {
      ...
      midObj: {
        innerObj: {
          size: 0
        }
      }
    }
  },
  computed: {
    totalCount() {
      return this.count + this.num
    }
  },
  watch: {
    totalCount(newVal, oldVal) {
      console.log(`count + num = ${newVal}`)
    },
    'midObj.innerObj.size': {
      // deep: true,
      immediate: true,
      handler(newVal, oldVal) {
        console.log(`this.midObj.innerObj.size = ${newVal}`)
      }
    }
  }
}
复制代码
```

\--

```
// vue3.x
import { reactive, ref, toRefs, computed, watch } from 'vue'

export default {
  name: 'Test',
  setup () {
    const state = reactive({
      count: 0,
      double: computed(() => {
        return state.count * 2
      }),
      midObj: {
        innerObj: {
          size: 0
        }
      }
    })
    const num = ref(0)

    const addCount = function () {
      state.count++
    }
    const addNum = function () {
      num.value++
    }

    // 监听单个属性
    watch(() => totalCount.value, (newVal, oldVal) => {
      console.log(`count + num = ${newVal}`)
    })
    // 监听单个属性, immediate
    watch(() => totalCount.value, (newVal, oldVal) => {
      console.log(`count + num = ${newVal}, immediate=true`)
    }, {
      immediate: true
    })
    // 监听单个属性, deep
    watch(() => state.midObj, (newVal, oldVal) => {
      console.log(`state.midObj = ${JSON.stringify(newVal)}, deep=true`)
    }, {
      deep: true
    })
    setTimeout(() => {
      state.midObj.innerObj.size = 1
    }, 2000)
    // 监听多个属性
    watch([num, () => totalCount.value], ([numVal, totalVal], [oldNumVal, OldTotalVal]) => {
      console.log(`num is ${numVal}, count + num = ${totalVal}`)
    })
    // 副作用，会立即执行
    let callTimes = 0
    const stopEffect = watchEffect(() => {
      console.log('watchEffect is called!')
      const div = document.createElement('div')
      div.textContent = `totalCount is ${totalCount.value}`
      document.body.appendChild(div)
      // 调用 5 次后，取消effect监听
      callTimes++
      if (callTimes >= 5) stopEffect()
    })

    return {
      ...toRefs(state),
      num,
      totalCount,
      addCount,
      addNum
    }
  }
}
复制代码
```

### 4、生命周期钩子

2.x中生命周期钩子放在跟`methods`同级属性下

3.x中需要先导入钩子，然后在`setup`方法中注册钩子回调，并且钩子命名也跟React保持一样了

3.x移除了2.x中的`beforeCreate`和`created`钩子，通过`setup`方法代替

**与 React Hooks 相比**

基于函数的组合式 API 提供了与 React Hooks 同等级别的逻辑组合能力，但是它们还是有很大不同：组合式 API 的 `setup`() 函数只会被调用一次，这意味着使用 Vue 组合式 API 的代码会是：

一般来说更符合惯用的 JavaScript 代码的直觉；

> 不需要顾虑调用顺序，也可以用在条件语句中；
>  不会在每次渲染时重复执行，以降低垃圾回收的压力；
>  不存在内联处理函数导致子组件永远更新的问题，也不需要 useCallback；
>  不存在忘记记录依赖的问题，也不需要“useEffect”和“useMemo”并传入依赖数组以捕获过时的变量。Vue 的自动依赖跟踪可以确保侦听器和计算值总是准确无误。
>  我们感谢 React Hooks 的创造性，它也是本提案的主要灵感来源，然而上面提到的一些问题存在于其设计之中，且我们发现 Vue 的响应式模型恰好为解决这些问题提供了一种思路。

```
// vue2.x
export default {
  data () {
    return {}
  },
  methods: {
    ...
  },
  beforeCreate() {},
  created() {},
  beforeMount() {},
  mounted() {},
  beforeUpdate() {},
  updated() {},
  beforeDestroy() {},
  destroyed() {}
}
复制代码
```

\--

```
// vue3.x
import {
  onBeforeMount,
  onMounted,
  onBeforeUpdate,
  onUpdated,
  onBeforeUnmount,
  onUnmounted
} from 'vue'

export default {
  setup() {
    onBeforeMount(() => {
      console.log('component is onBeforeMount')
    })
    onMounted(() => {
      console.log('component is onMounted')
    })
    onBeforeUpdate(() => {
      console.log('component is onBeforeUpdate')
    })
    onUpdated(() => {
      console.log('component is onUpdated')
    })
    onBeforeUnmount(() => {
      console.log('component is onBeforeUnmount')
    })
    onUnmounted(() => {
      console.log('component is onUnmounted')
    })
  }
}
复制代码
```

**2.x钩子对比3.x**



![img](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ff94cb96d5684b62aac54b1438785788~tplv-k3u1fbpfcp-zoom-1.image)



### 5、Fragment

2.x中，vue template只允许有一个根节点

3.x中，vue template支持多个根节点，用过React的人应该知道`<React.Fragment>`和`<></>`

```
// vue2.x
<template>
  <div>
    <span>hello</span>
    <span>world</span>
  </div>
</template>
复制代码
```

\--

```
// vue3.x
<template>
  <span>hello</span>
  <span>world</span>
</template>
复制代码
```

### 6、Teleport

`teleport`参照React中的`portal`，可以将元素渲染在父节点以外的其他地方，比如`<body>`下面的某个子元素

在vue3中， `<teleport>`是一个内置标签，我们通常将弹窗、tooltip等元素放在关闭的 `</body>`标签之前，如下：

```
<body>
  <div id="app">
    <!--main page content here-->
  </div>
  <!--modal here-->
</body>
复制代码
```

如果按照以往的思路，需要将模态的UI代码放在底部，如下：

```
<body>
  <div id="app">
    <h3>Tooltips with Vue 3 Teleport</h3>
  </div>
  <div>
    <my-modal></my-modal>
  </div>
</body>
复制代码
```

这样做是因为弹窗、tooltip需要显示在页面上层，需要正确处理父元素定位和z-index上下层级顺序，而最简单的解决方案是将这类DOM放在页面的最底部。这样的话这部分逻辑就脱离了整个项目的跟组件App的管理，就造成直接用JavaScript和CSS来修改UI，不规范并且失去响应式了。为了允许将一些UI片段段移动到页面中的其他位置，在Vue3中添加了一个新的`<teleport>`组件，并且`<teleport>`会在组件销毁时自动清空相应的dom，不用人工处理。

要使用 `<teleport>`，首先要在页面上添加一个元素，我们要将模态内容渲染到该元素下面。

代码如下：

```
<body>
  <div id="app">
    <h3>Tooltips with Vue 3 Teleport</h3>
  </div>
  <div id="endofbody"></div>
</body>
复制代码
```

\--

```
<template>
  <button @click="openModal">
    Click to open modal! (With teleport!)
  </button>
  <teleport to="#endofbody">
    <div v-if="isModalOpen" class="modal">
      ...
    </div>
  </teleport>
</template>

<script>
import { ref } from 'vue'

export default {
  setup() {
    const isModalOpen = ref(false)
    const openModal = function () {
      isModalOpen.value = true
    }
    return {
      isModalOpen,
      openModal
    }
  }
}
</script>
复制代码
```

### 7、Suspense

`<Suspense>`是一个特殊的组件，它将呈现回退内容，而不是对于的组件，直到满足条件为止，这种情况通常是组件`setup`功能中发生的异步操作或者是异步组件中使用。例如这里有一个场景，父组件展示的内容包含异步的子组件，异步的子组件需要一定的时间才可以加载并展示，这时就需要一个组件处理一些占位逻辑或者加载异常逻辑，要用到 `<Suspense>`，例如：

```
// vue2.x
<template>
  <div>
    <div v-if="!loading">
      ...
    </div>
    <div v-if="loading">Loading...</div>
  </div>
</template>
复制代码
```

或者在vue2.x中使用`vue-async-manager`

```
<template>
  <div>
    <Suspense>
      <div>
        ...
      </div>
      <div slot="fallback">Loading...</div>
    </Suspense>
  </div>
</template>
复制代码
```

\--

```
// vue3.x
<Suspense>
  <template >
    <Suspended-component />
  </template>
  <template #fallback>
    Loading...
  </template>
</Suspense>
复制代码
```

上面代码中，假设 `<Suspended-component>`是一个异步组件，直到它完全加载并渲染前都会显示占位内容：Loading，这就是 `<Suspense>`的简单用法，该特性和Fragment以及 一样，灵感来自React



<br>

<br>

<br>

<br>

[Vue Function-based API RFC](https://zhuanlan.zhihu.com/p/68477600)

[全家桶](https://juejin.im/post/6867123074148335624)