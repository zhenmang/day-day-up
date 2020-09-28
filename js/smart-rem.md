再不用smart-rem，可能就out了。这是一个可以自动计算rem的代码库，它适用于所有主流开发环境：

**浏览器环境**

**vue框架**

**react框架**

**angular框架**

**nuxt框架**

**next框架**

[英文版介绍](https://www.npmjs.com/package/smart-rem)

各种环境的用法分列如下：

### 1.script 标签

**第一步:**

在html文件中

```
<script src="./smart-rem.js"></script>
复制代码
```

**第二步:**

在html文件中

```
<script>
  smartRem(Arguments)
</script>
复制代码
```

**注释**: 参数是设计稿的宽度。如果设计稿的宽度是750px，那么smartRem的参数就是750，注意不带px，写法如下：

```
smartRem(750)
复制代码
```

**第三步:**

在css文件中

```
.class-name {
    width: 1.5rem;
    font-size: 0.5rem;
}
复制代码
```

**注释**:如果元素的宽度是150px，那么它的rem值就是1.5rem，计算方式简单如下：

```
150 / 100 = 1.5
150px     =>1.5rem
--------------------
50 / 100 = 0.5
50px     => 0.5rem
复制代码
```

### 2.vue 框架

**第一步:**

```
npm install smart-rem -S
复制代码
```

**第二步:**

在文件 src/main.js 中

```
import smartRem from 'smart-rem'
smartRem(Arguments)
复制代码
```

**注释**: 参数是设计稿的宽度。如果设计稿的宽度是750px，那么smartRem的参数就是750，注意不带px，写法如下：

```
smartRem(750)
复制代码
```

**第三步:**

在文件 src/*.vue 中

```
.class-name {
    width: 1.5rem;
    font-size: 0.5rem;
}

复制代码
```

**注释**:如果元素的宽度是150px，那么它的rem值就是1.5rem，计算方式简单如下：

```
150 / 100 = 1.5
150px     =>1.5rem
--------------------
50 / 100 = 0.5
50px     => 0.5rem

复制代码
```

### 3.react 框架

**第一步:**

```
npm install smart-rem -S

复制代码
```

**第二步:**

在文件 src/index.js 中

```
import smartRem from 'smart-rem'
smartRem(Arguments)

复制代码
```

**注释**: 参数是设计稿的宽度。如果设计稿的宽度是750px，那么smartRem的参数就是750，注意不带px，写法如下：

```
smartRem(750)
复制代码
```

**第三步:**

在文件 src/*.css 中

```
.class-name {
    width: 1.5rem;
    font-size: 0.5rem;
}

复制代码
```

**注释**:如果元素的宽度是150px，那么它的rem值就是1.5rem，计算方式简单如下：

```
150 / 100 = 1.5
150px     =>1.5rem
--------------------
50 / 100 = 0.5
50px     => 0.5rem

复制代码
```

### 4.angular 框架

**第一步:**

```
npm install smart-rem -S

复制代码
```

**第二步:**

在文件 src/main.ts 中

```
import smartRem from 'smart-rem'
smartRem(Arguments)
复制代码
```

**Notes**:参数是设计稿的宽度。如果设计稿的宽度是750px，那么smartRem的参数就是750，注意不带px，写法如下：

```
smartRem(750)
复制代码
```

**第三步:**

在文件 src/**/*.styl 中

```
.class-name {
    width: 1.5rem;
    font-size: 0.5rem;
}

复制代码
```

**注释**:如果元素的宽度是150px，那么它的rem值就是1.5rem，计算方式简单如下：

```
150 / 100 = 1.5
150px     =>1.5rem
--------------------
50 / 100 = 0.5
50px     => 0.5rem

复制代码
```

### 5.nuxt 框架

**第一步:**

```
npm install smart-rem -S
复制代码
```

**第二步:**

在文件 nuxt.config.js中，在head中添加script，操作如下：

```
head: {
    title: pkg.name,
    meta: [
      { charset: 'utf-8' },
      { name: 'viewport', content: 'width=device-width, initial-scale=1' },
      { hid: 'description', name: 'description', content: pkg.description }
    ],
    link: [{ rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }],
    // ** 开始添加 ** //
    script: [
      {
        innerHTML: require('smart-rem') + 'smartRem(Arguments)',
        type: 'text/javascript',
        charset: 'utf-8'
      }
    ],
    __dangerouslyDisableSanitizers: ['script']
    // ** 结束添加 **//
  },

复制代码
```

**注释**:参数是设计稿的宽度。如果设计稿的宽度是750px，那么smartRem的参数就是750，注意不带px，写法如下：

```
'smartRem(750)'
复制代码
```

**第三步:**

pages/*.vue,

components/*.vue,

layouts/*.vue.

在以上文件中，写法如下

```
.class-name {
    width: 1.5rem;
    font-size: 0.5rem;
}

复制代码
```

**注释**:如果元素的宽度是150px，那么它的rem值就是1.5rem，计算方式简单如下：

```
150 / 100 = 1.5
150px     =>1.5rem
--------------------
50 / 100 = 0.5
50px     => 0.5rem

复制代码
```

### 6.next 框架

**第一步:**

```
npm install smart-rem -S
复制代码
```

**第二步:**

创建一个文件 `pages/_document.js` ，添加如下内容：

```
// 提醒：_document 只在服务端渲染，不在客户端渲染，类似onClick的回调函数不能在此添加

// ./pages/_document.js
import Document, { Head, Main, NextScript } from 'next/document'

export default class MyDocument extends Document {
  static async getInitialProps(ctx) {
    const initialProps = await Document.getInitialProps(ctx)
    return { ...initialProps }
  }

  render() {
    return (
      <html>
        <Head>
          <style>{`body { margin: 0 } /* custom! */`}</style>
          // ** 开始添加 ** //
          <script dangerouslySetInnerHTML={{__html: require('smart-rem') + 'smartRem(Arguments)'}}></script>
          // ** 结束添加 ** //
        </Head>
        <body className="custom_class">
          <Main />
          <NextScript />
        </body>
      </html>
    )
  }
}

复制代码
```

**注释**:参数是设计稿的宽度。如果设计稿的宽度是750px，那么smartRem的参数就是750，注意不带px，写法如下：

```
'smartRem(750)'
复制代码
```

**第三步:**

在文件 pages/*.js 中，书写如下：

```
.class-name {
    width: 1.5rem;
    font-size: 0.5rem;
}

复制代码
```

**注释**:如果元素的宽度是150px，那么它的rem值就是1.5rem，计算方式简单如下：

```
150 / 100 = 1.5
150px     =>1.5rem
--------------------
50 / 100 = 0.5
50px     => 0.5rem

复制代码
```

------

其它：

​	如果你已经使用过代码编辑器插件cssrem，你可以做以下设置：

​			在VSCode中，把Root Font Size的值设置为100

​			在Sublime中，把px_to_rem的值设置为100

​	如果你已经使用过开发依赖包postcss-pxtorem，你可以将rootValue值设为100