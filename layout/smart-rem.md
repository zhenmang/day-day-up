### smart-rem

​	**smart-rem is library for automatic REM calculation, it's suitable for browser environment, vue frame, react frame, angular frame, nuxt frame, next frame.List the usages as follows.**

# Usage

**smart-rem is suitable for browser environment, vue frame, react frame, angular frame, nuxt frame, next frame.List the usages as follows.**

### 1.browser

**Step one:**

In file *.html

```
<script src="./smart-rem.js"></script>
```

**Step two:**

In file *.html

```
<script>
  smartRem(Arguments)
</script>
```

**Notes**: Arguments is the width of design draft. If design draft's width is 750px, the argument of smartRem is 750 without 'px' as follows:

```
smartRem(750)
```

**Step three:**

In file *.css

```
.class-name {
    width: 1.5rem;
    font-size: 0.5rem;
}
```

**Notes**: If element'width of design draft is 150px, then it's rem is 1.5. The formula is simple as follows:

```
150 / 100 = 1.5
150px     =>1.5rem
--------------------
50 / 100 = 0.5
50px     => 0.5rem
```

### 2.vue

**Step one:**

```
npm install smart-rem -S
```

**Step two:**

In file src/main.js

```
import smartRem from 'smart-rem'
smartRem(Arguments)
```

**Notes**: Arguments is the width of design draft. If design draft's width is 750px, the argument of smartRem is 750 without 'px' as follows:

```
smartRem(750)
```

**Step three:**

In file src/*.vue

```
.class-name {
    width: 1.5rem;
    font-size: 0.5rem;
}
```

**Notes**: If element'width of design draft is 150px, then it's rem is 1.5. The formula is simple as follows:

```
150 / 100 = 1.5
150px     =>1.5rem
--------------------
50 / 100 = 0.5
50px     => 0.5rem
```

### 3.react

**Step one:**

```
npm install smart-rem -S
```

**Step two:**

In file src/index.js

```
import smartRem from 'smart-rem'
smartRem(Arguments)
```

**Notes**: Arguments is the width of design draft. If design draft's width is 750px, the argument of smartRem is 750 without 'px' as follows:

```
smartRem(750)
```

**Step three:**

In file src/*.css

```
.class-name {
    width: 1.5rem;
    font-size: 0.5rem;
}
```

**Notes**: If element'width of design draft is 150px, then it's rem is 1.5. The formula is simple as follows:

```
150 / 100 = 1.5
150px     =>1.5rem
--------------------
50 / 100 = 0.5
50px     => 0.5rem
```

### 4.angular

**Step one:**

```
npm install smart-rem -S
```

**Step two:**

In file src/main.ts

```
import smartRem from 'smart-rem'
smartRem(Arguments)
```

**Notes**: Arguments is the width of design draft. If design draft's width is 750px, the argument of smartRem is 750 without 'px' as follows:

```
smartRem(750)
```

**Step three:**

In file src/**/*.styl

```
.class-name {
    width: 1.5rem;
    font-size: 0.5rem;
}
```

**Notes**: If element'width of design draft is 150px, then it's rem is 1.5. The formula is simple as follows:

```
150 / 100 = 1.5
150px     =>1.5rem
--------------------
50 / 100 = 0.5
50px     => 0.5rem
```

### 5.nuxt

**Step one:**

```
npm install smart-rem -S
```

**Step two:**

In file nuxt.config.js, add script to head as follows:

```
head: {
    title: pkg.name,
    meta: [
      { charset: 'utf-8' },
      { name: 'viewport', content: 'width=device-width, initial-scale=1' },
      { hid: 'description', name: 'description', content: pkg.description }
    ],
    link: [{ rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }],
    // ** Begin Adding
    script: [
      {
        innerHTML: require('smart-rem') + 'smartRem(Arguments)',
        type: 'text/javascript',
        charset: 'utf-8'
      }
    ],
    __dangerouslyDisableSanitizers: ['script']
    // ** End Adding
  },
```

**Notes**: Arguments is the width of design draft. If design draft's width is 750px, the argument of smartRem is 750 without 'px' as follows:

```
'smartRem(750)'
```

**Step three:**

In file pages/*.vue,

​	components/*.vue,

 and layouts/*.vue.

```
.class-name {
    width: 1.5rem;
    font-size: 0.5rem;
}
```

**Notes**: If element'width of design draft is 150px, then it's rem is 1.5. The formula is simple as follows:

```
150 / 100 = 1.5
150px     =>1.5rem
--------------------
50 / 100 = 0.5
50px     => 0.5rem
```

### 6.next

**Step one:**

```
npm install smart-rem -S
```

**Step two:**

Create a file named `pages/_document.js` and add the following content:

```
// _document is only rendered on the server side and not on the client side
// Event handlers like onClick can't be added to this file

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
          // ** Begin Adding
          <script dangerouslySetInnerHTML={{__html: require('smart-rem') + 'smartRem(Arguments)'}}></script>
          // ** End Adding
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

**Notes**: Arguments is the width of design draft. If design draft's width is 750px, the argument of smartRem is 750 without 'px' as follows:

```
'smartRem(750)'
```

**Step three:**

In file pages/*.js

```
.class-name {
    width: 1.5rem;
    font-size: 0.5rem;
}
```

**Notes**: If element'width of design draft is 150px, then it's rem is 1.5. The formula is simple as follows:

```
150 / 100 = 1.5
150px     =>1.5rem
--------------------
50 / 100 = 0.5
50px     => 0.5rem
```

