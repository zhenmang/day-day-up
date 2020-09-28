

## API

1. 在浏览器中，大多数时候做的是与 DOM 或其他 Web 平台 API（例如 Cookies）进行交互。 当然，那些在 Node.js 中是不存在的。 没有浏览器提供的 `document`、`window`、以及所有其他的对象。

2. 在浏览器中，不存在 Node.js 通过其模块提供的所有不错的 API，例如文件系统访问功能。

## 兼容

1. 由于 JavaScript 发展的速度非常快，但是浏览器发展得慢一些，并且用户的升级速度也慢一些，因此有时在 web 上，不得不使用较旧的 JavaScript / ECMAScript 版本。

2. 可以使用 Babel 将代码转换为与 ES5 兼容的代码，再交付给浏览器，但是在 Node.js 中，则不需要这样做。

## 模块

Node.js 使用 CommonJS 模块系统，而在浏览器中，则还正在实现 ES 模块标准。Node.js 中使用 `require()`，而在浏览器中则使用 `import`。

