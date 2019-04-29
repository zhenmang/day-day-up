### **prettier**

每个开发人员都有自己的撸码风格，例如：



示例一

```
function foo(items) {
  return items
    .filter(item => item.checked)
    .map(item => item.value)
  ;
}
```

```
function foo(items) {
  return items.filter(item => item.checked).map(item => item.value);
}
```

示例二

```
const food = [
  'pizza',
  'burger',
  'pasta',
]
```

```
const food = ["pizza", "burger", "pasta"];
```

示例三

```
console.log(
  a
)
```

```
console.log(a)
```

示例四

```
let back = '' + a + JSON.stringify(b)
```

示例五

```
let b = {name:'xm', age:25, sex:'male', color:'yellow'}
```

```
let b = { name: 'xm', age: 25, sex: 'male', color: 'yellow' }
```



我们可以使用[ESLint ](https://eslint.org/)来统一风格。但是它并不能保证代码100%一致。

单看比较严格的airbnb配置，不能统一**示例一**、**示例二**、**示例三**，对**示例四**的修复是这样的：

```
const back = `${a}${JSON.stringify(b)}`;
```

而这种侵入式的改写，并不是我们期望的。



在standard和airbnb两种标准下，**示例五**修复后的格式也不一样：

standard：

```
let b = { name: 'xm', age: 25, sex: 'male', color: 'yellow' }
```

airbnb：

```
const b = {
  name: 'xm', age: 25, sex: 'male', color: 'yellow',
};
```

而有些小伙伴渴望是这样的

```
const c = {
  name: 'xm',
  age: 25,
  sex: 'male',
  color: 'yellow',
};
```



综上：

（1）standard标准和airbnb标准有出入，代码风格习惯不同。

（2）在单个标准下，也不能保障代码的100%一致。

（3）有些情况下，侵入式的改写了代码的表达方式。

看到仍然还有很多代码格式不一样的地方，关于数组的写法，A猿认为每一项都要换行，B猿认为没必换行，C猿认为超过一定长度再换行，D猿说，你们快统一下。

这时，很有主见的[Prettier](https://github.com/prettier/prettier)前来[发炎了](http://jlongster.com/A-Prettier-Formatter):"不管你们之前用的啥，老弟要亮剑了"。

- 几乎不需要做决定，因为 Prettier的配置选项很少。
- 团队成员不需要为规则去争论。
- 开源代码开发者不需要去学习项目的代码风格。
- 不需要去修复ESLint报告的风格问题。
- 保存文件的时候可以自动统一风格。

为了证明自己，prettier亮出了目前的支持者

![prettier_users](/imgs/prettier_users.png)



prettier即可单独使用，又可结合eslint，各取所长。走，去瞧瞧[如何使用](https://survivejs.com/maintenance/code-quality/code-formatting/)

单独使用prettier

```bash
npm install prettier --save-dev
```

package.json

```
{
	"scripts"**:** {
		"format"**:** "prettier --write '**/*.{js,css,md}'"
	 }
}
```

创建文件**.prettierrc**

```
{
  "printWidth": 50,
  "singleQuote": true,
  "trailingComma": "es5"
}
```

最后

```
npm run format
```



我们看到

**示例一**的内容自动规范成了

```
  let abc = items
    .filter(item => item.checked)
    .map(item => item.value);
```

**示例二**中的内容自动规范成了

```
const food = ['pizza', 'burger', 'pasta'];
```

**示例三**的内容自动规范成了

```
console.log(b);
```

**示例四**的代码保持原样，并没有做侵入式的修改

**示例五**自动规范成了

```
let b = {
  name: 'xm',
  age: 25,
  sex: 'male',
  color: 'yellow',
};
```

综上，prettier保持了代码的一致性，且不会侵入式修改代码



再来看下，eslint结合prettier的使用情况

```
npm install prettier eslint-plugin-prettier --save-dev
```

更新.eslintrc.js or .eslintrc.json

```
{
  "plugins": ["prettier"],
  "rules": {
    "prettier/prettier": [
      "error",
      {
        "printWidth": 100,
        "singleQuote": true,
        "trailingComma": "es5"
      }
    ]
  }
}
```

最后执行

```
npm run lint test.js
```

执行顺序是先进行了prettier，然后进行了eslint，若两者有冲突的地方，以prettier为主。