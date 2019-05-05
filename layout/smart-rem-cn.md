### smart-rem

**smart-rem是一个可以自动计算rem的代码库，它适用于以下开发环境：**

**浏览器环境**

**vue框架**

**react框架**

**angular框架**

**nuxt框架**

**next框架**



各种环境的用法分列如下：

### 1.浏览器环境

**第一步:**

在html文件中

```
<script src="./smart-rem.js"></script>
```

**第二步:**

在html文件中

```
<script>
  smartRem(Arguments)
</script>
```

**注释**: 参数是设计稿的宽度。如果设计稿的宽度是750px，那么smartRem的参数就是750，注意不带px，写法如下：

```
smartRem(750)
```

**第三步:**

在css文件中

```
.class-name {
    width: 1.5rem;
    font-size: 0.5rem;
}
```

**注释**:如果元素的宽度是150px，那么它的rem值就是1.5rem，计算方式简单如下：

```
150 / 100 = 1.5
150px     =>1.5rem
--------------------
50 / 100 = 0.5
50px     => 0.5rem
```

### 2.vue

**第一步:**

```
npm install smart-rem -S
```

**第二步:**

在文件 src/main.js 中

```
import smartRem from 'smart-rem'
smartRem(Arguments)
```

**注释**: 参数是设计稿的宽度。如果设计稿的宽度是750px，那么smartRem的参数就是750，注意不带px，写法如下：

```
smartRem(750)
```

**第三步:**

在文件 src/*.vue 中

```
.class-name {
    width: 1.5rem;
    font-size: 0.5rem;
}

```

**注释**:如果元素的宽度是150px，那么它的rem值就是1.5rem，计算方式简单如下：

```
150 / 100 = 1.5
150px     =>1.5rem
--------------------
50 / 100 = 0.5
50px     => 0.5rem

```

### 3.react

**第一步:**

```
npm install smart-rem -S

```

**第二步:**

在文件 src/index.js 中

```
import smartRem from 'smart-rem'
smartRem(Arguments)

```

**注释**: 参数是设计稿的宽度。如果设计稿的宽度是750px，那么smartRem的参数就是750，注意不带px，写法如下：

```
smartRem(750)
```

**第三步:**

在文件 src/*.css 中

```
.class-name {
    width: 1.5rem;
    font-size: 0.5rem;
}

```

**注释**:如果元素的宽度是150px，那么它的rem值就是1.5rem，计算方式简单如下：

```
150 / 100 = 1.5
150px     =>1.5rem
--------------------
50 / 100 = 0.5
50px     => 0.5rem

```

### 4.angular

**第一步:**

```
npm install smart-rem -S

```

**第二步:**

在文件 src/main.ts 中

```
import smartRem from 'smart-rem'
smartRem(Arguments)
```

**Notes**:参数是设计稿的宽度。如果设计稿的宽度是750px，那么smartRem的参数就是750，注意不带px，写法如下：

```
smartRem(750)
```

**第三步:**

在文件 src/**/*.styl 中

```
.class-name {
    width: 1.5rem;
    font-size: 0.5rem;
}

```

**注释**:如果元素的宽度是150px，那么它的rem值就是1.5rem，计算方式简单如下：

```
150 / 100 = 1.5
150px     =>1.5rem
--------------------
50 / 100 = 0.5
50px     => 0.5rem

```

### 5.nuxt

**第一步:**

```
npm install smart-rem -S
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

```

**注释**:参数是设计稿的宽度。如果设计稿的宽度是750px，那么smartRem的参数就是750，注意不带px，写法如下：

```
'smartRem(750)'
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

```

**注释**:如果元素的宽度是150px，那么它的rem值就是1.5rem，计算方式简单如下：

```
150 / 100 = 1.5
150px     =>1.5rem
--------------------
50 / 100 = 0.5
50px     => 0.5rem

```

### 6.next

**第一步:**

```
npm install smart-rem -S
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

```

**注释**:参数是设计稿的宽度。如果设计稿的宽度是750px，那么smartRem的参数就是750，注意不带px，写法如下：

```
'smartRem(750)'
```

**第三步:**

在文件 pages/*.js 中，书写如下：

```
.class-name {
    width: 1.5rem;
    font-size: 0.5rem;
}

```

**注释**:如果元素的宽度是150px，那么它的rem值就是1.5rem，计算方式简单如下：

```
150 / 100 = 1.5
150px     =>1.5rem
--------------------
50 / 100 = 0.5
50px     => 0.5rem

```

