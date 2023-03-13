## AST

#### 三个重要环节

- 词法分析器：字符流->单词流(tokens)
- 语法分析器：单词流->语法树(tree)
- 语义分析器：收集标识符属性、进行语义检查

源代码

`var answer = 6 * 7;`

tokens

```
[
    {
        "type": "Keyword",
        "value": "var"
    },
    {
        "type": "Identifier",
        "value": "answer"
    },
    {
        "type": "Punctuator",
        "value": "="
    },
    {
        "type": "Numeric",
        "value": "6"
    },
    {
        "type": "Punctuator",
        "value": "*"
    },
    {
        "type": "Numeric",
        "value": "7"
    },
    {
        "type": "Punctuator",
        "value": ";"
    }
]
```

语法树

```
{
  "type": "Program",
  "body": [
    {
      "type": "VariableDeclaration",
      "declarations": [
        {
          "type": "VariableDeclarator",
          "id": {
            "type": "Identifier",
            "name": "answer"
          },
          "init": {
            "type": "BinaryExpression",
            "operator": "*",
            "left": {
              "type": "Literal",
              "value": 6,
              "raw": "6"
            },
            "right": {
              "type": "Literal",
              "value": 7,
              "raw": "7"
            }
          }
        }
      ],
      "kind": "var"
    }
  ],
  "sourceType": "script"
}
```



#### 词法分析的作用

- 识别源程序中的单词是否有误
- 读入源程序字符流、组成词素，输出词法单元序列
- 过滤空白、换行、制表符、注释等
- 将词素添加到符号表中

#### 语法分析器的作用

- 利用语法检查单词流的语法结构
- 构造语法分析树
- 语法错误和修正
- 识别正确语法
- 报告错误

#### 语义分析的主要任务

**1.主要任务**：分析源程序的含义并做出相应的语义处理。  
    **1）语义处理**：静态语义检查、语义翻译。  
**2.静态语义检查**  
    **1）类型检查**。**eg**：赋值运算两边类型是否相容。  
    **2）控制流检查**。**eg：for**循环的嵌套关系是否正确。  
    **3）唯一性检查**。**eg**：switch中case标号是否重复定义。  
    **4）其它相关的语义检查**。**eg**：局部内部类不能访问非final型的局部变量。  
**3.语义翻译**：生成与硬件机器相对独立的中间语言代码。  
        **i**）可以进行与具体机器特性无关的反应代码本身特性的代码优化。  
        **ii**）当要将编译程序移植到新的目标机器时，前端几乎不变，只需修改后端即可. 

#### 示例

`npm i recast -S`

```
// 给你一把"螺丝刀"——recast
const recast = require("recast");

// 你的"机器"——一段代码
// 我们使用了很奇怪格式的代码，想测试是否能维持代码结构
const code =
  `
  function add(a, b) {
    return a +
      // 有什么奇怪的东西混进来了
      b
  }
  `
// 用螺丝刀解析机器
const ast = recast.parse(code);

// ast可以处理很巨大的代码文件
// 但我们现在只需要代码块的第一个body，即add函数
const add  = ast.program.body[0]

console.log(add)
```

```
FunctionDeclaration{
    type: 'FunctionDeclaration',
    id: ...
    params: ...
    body: ...
}
```





<br>

<br>

<br>

[ast-简单实践](https://segmentfault.com/a/1190000016231512)

[ast对象文档](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/SpiderMonkey/Parser_API#Node_objects)

[esprima工具](https://esprima.org/demo/parse.html#)

[js编译](https://zhuanlan.zhihu.com/p/55430043)

[应用范围](https://www.jianshu.com/p/019d449a9282)

