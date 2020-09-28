## 内核构成

浏览器内核又可以分成两部分：渲染引擎`(layout engineer 或者 Rendering Engine)`和 JS 引擎。
最开始渲染引擎和 JS 引擎并没有区分的很明确，后来 JS 引擎越来越独立，内核就倾向于只指渲染引擎。
常见的浏览器内核可以分这四种：Trident、Gecko、Blink、Webkit。

## Trident

Trident(IE内核) ([‘traɪd(ə)nt])：该内核程序在 1997 年的 IE4 中首次被采用，是微软在 Mosaic（”马赛克”，这是人类历史上第一个浏览器，从此网页可以在图形界面的窗口浏览） 代码的基础之上修改而来的，并沿用到 IE11，也被普遍称作 “IE内核”。
IE 从版本 11 开始，初步支持 WebGL 技术。IE8 的 JavaScript 引擎是 Jscript，IE9 开始用 Chakra，这两个版本区别很大，Chakra 无论是速度和标准化方面都很出色。
Window10 发布后，IE 将其内置浏览器命名为 Edge，Edge 最显著的特点就是新内核 EdgeHTML。

## Gecko

Gecko(Firefox 内核) ([‘gekəʊ])
开源内核。跨平台内核。同样基于Mosaic.

## Webkit

Webkit: Chrome, Webkit 的鼻祖其实是 Safari。现在很多人错误地把 webkit 叫做 chrome内核（即使 chrome内核已经是 blink 了）
WebKit 前身是 KDE 小组的 KHTML 引擎，可以说 WebKit 是 KHTML 的一个开源的分支。

## Chromium/Blink

Chromium/Blink(Chrome内核)
chromium fork 自开源引擎 webkit
谷歌公司还研发了自己的 Javascript 引擎，V8，极大地提高了 Javascript 的运算速度。
Chrome系浏览器：搜狗、360（双核）、QQ浏览器
Blink 其实是 WebKit 的分支，如同 WebKit 是 KHTML 的分支。

## Presto

Presto ([‘prestəʊ])
Presto 是挪威产浏览器 opera 的 “前任” 内核

## 移动端浏览器内核

移动端的浏览器内核主要说的是系统内置浏览器的内核。
目前移动设备浏览器上常用的内核有 Webkit，Blink，Trident，Gecko 等，其中 iPhone 和 iPad 等苹果 iOS 平台主要是 WebKit，Android 4.4 之前的 Android 系统浏览器内核是 WebKit，Android4.4 系统浏览器切换到了Chromium，内核是 Webkit 的分支 Blink，Windows Phone 8 系统浏览器内核是 Trident。