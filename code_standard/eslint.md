### eslint

eslint的主流标准有google、standard、airbnb

**eslint-config-google**

出品人：google

趋势：star 1k ；周下载 90,260

**eslint-config-standard**

出品人：广大前端战士

趋势：star 1.5k ；周下载 844,036

**eslint-config-airbnb**

出品人：airbnb

趋势：star 无数据 ；周下载 1,029,237





​	以下代码，standard报16个问题(可修复10个)，airbnb报42个问题(可修复26个)。standard标准较为宽松，最受欢迎，airbnb代码格式最为严苛，通过lint的代码质量也比standard高。

```
var a = 123
let b = {name:'xm', age: 25, sex: 'male', color: 'yellow'}

function test () {
  var ret = {}
  for (var i in arguments) {
    var m = arguments[i]
    for (var j in m) ret[j] = m[j]
  }
  return ret
}

function final() {
  let result = test({a: 123}, {b: 456})
  if (result) {
    [-1, 1].forEach(delta => Math.floor(delta * 20))
    return '' + value + a + JSON.stringify(b)
  }
}

export default {
  final
}
```







eslint详细的可[配置规则](https://cn.eslint.org/docs/rules/)，如果你觉得里面的配置规则太多太繁琐，而且即便精通配置，也有会人持不同意见，好了，渴望懒惰、不想争吵的小伙伴，带你们进入[下一篇](./prettier.md)
