---
title: "React DOM 组件"
---

<Intro>

React 支持所有浏览器内置的 [HTML](https://developer.mozilla.org/en-US/docs/Web/HTML/Element) 和 [SVG](https://developer.mozilla.org/en-US/docs/Web/SVG/Element) 组件。

</Intro>

---

## 常见组件 {/*common-components*/}

所有内置的浏览器组件都支持一些 props 和事件。

* [常见组件（例如 `<div>`）](/reference/react-dom/components/common)

这包括像 `ref` 和 `dangerouslySetInnerHTML` 这样的 React 特有 props。

---

## 表单组件 {/*form-components*/}

这些内置的浏览器组件接受用户输入：

* [`<input>`](/reference/react-dom/components/input)
* [`<select>`](/reference/react-dom/components/select)
* [`<textarea>`](/reference/react-dom/components/textarea)

它们在 React 中是特殊的，因为向它们传递 `value` prop 会使它们变为 *[受控。](/reference/react-dom/components/input#controlling-an-input-with-a-state-variable)*

---

## 资源和元数据组件 {/*resource-and-metadata-components*/}

这些内置的浏览器组件让你可以加载外部资源，或用元数据标注文档：

* [`<link>`](/reference/react-dom/components/link)
* [`<meta>`](/reference/react-dom/components/meta)
* [`<script>`](/reference/react-dom/components/script)
* [`<style>`](/reference/react-dom/components/style)
* [`<title>`](/reference/react-dom/components/title)

它们在 React 中是特殊的，因为 React 可以将它们渲染到 document head 中，在资源加载时挂起，并执行各组件参考页中描述的其他行为。

---

## 所有 HTML 组件 {/*all-html-components*/}

React 支持所有内置的浏览器 HTML 组件。这包括：

* [`<aside>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/aside)
* [`<audio>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/audio)
* [`<b>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/b)
* [`<base>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/base)
* [`<bdi>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/bdi)
* [`<bdo>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/bdo)
* [`<blockquote>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/blockquote)
* [`<body>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/body)
* [`<br>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/br)
* [`<button>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/button)
* [`<canvas>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/canvas)
* [`<caption>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/caption)
* [`<cite>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/cite)
* [`<code>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/code)
* [`<col>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/col)
* [`<colgroup>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/colgroup)
* [`<data>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/data)
* [`<datalist>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/datalist)
* [`<dd>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dd)
* [`<del>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/del)
* [`<details>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/details)
* [`<dfn>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dfn)
* [`<dialog>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dialog)
* [`<div>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/div)
* [`<dl>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dl)
* [`<dt>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/dt)
* [`<em>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/em)
* [`<embed>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/embed)
* [`<fieldset>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/fieldset)
* [`<figcaption>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/figcaption)
* [`<figure>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/figure)
* [`<footer>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/footer)
* [`<form>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/form)
* [`<h1>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/h1)
* [`<head>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/head)
* [`<header>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/header)
* [`<hgroup>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/hgroup)
* [`<hr>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/hr)
* [`<html>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/html)
* [`<i>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/i)
* [`<iframe>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe)
* [`<img>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/img)
* [`<input>`](/reference/react-dom/components/input)
* [`<ins>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/ins)
* [`<kbd>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/kbd)
* [`<label>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/label)
* [`<legend>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/legend)
* [`<li>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/li)
* [`<link>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/link)
* [`<main>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/main)
* [`<map>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/map)
* [`<mark>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/mark)
* [`<menu>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/menu)
* [`<meta>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/meta)
* [`<meter>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/meter)
* [`<nav>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/nav)
* [`<noscript>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/noscript)
* [`<object>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/object)
* [`<ol>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/ol)
* [`<optgroup>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/optgroup)
* [`<option>`](/reference/react-dom/components/option)
* [`<output>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/output)
* [`<p>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/p)
* [`<picture>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/picture)
* [`<pre>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/pre)
* [`<progress>`](/reference/react-dom/components/progress)
* [`<q>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/q)
* [`<rp>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/rp)
* [`<rt>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/rt)
* [`<ruby>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/ruby)
* [`<s>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/s)
* [`<samp>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/samp)
* [`<script>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script)
* [`<section>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/section)
* [`<select>`](/reference/react-dom/components/select)
* [`<slot>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/slot)
* [`<small>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/small)
* [`<source>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/source)
* [`<span>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/span)
* [`<strong>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/strong)
* [`<style>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/style)
* [`<sub>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/sub)
* [`<summary>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/summary)
* [`<sup>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/sup)
* [`<table>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/table)
* [`<tbody>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/tbody)
* [`<td>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/td)
* [`<template>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/template)
* [`<textarea>`](/reference/react-dom/components/textarea)
* [`<tfoot>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/tfoot)
* [`<th>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/th)
* [`<thead>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/thead)
* [`<time>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/time)
* [`<title>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/title)
* [`<tr>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/tr)
* [`<track>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/track)
* [`<u>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/u)
* [`<ul>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/ul)
* [`<var>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/var)
* [`<video>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/video)
* [`<wbr>`](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/wbr)

<Note>

类似于 [DOM 标准，](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model)React 对 prop 名称使用 `camelCase` 约定。例如，你会写 `tabIndex` 而不是 `tabindex`。你可以使用 [在线转换器。](https://transform.tools/html-to-jsx)将现有 HTML 转换为 JSX。

</Note>

---

### 自定义 HTML 元素 {/*custom-html-elements*/}

如果你渲染一个带有连字符的标签，比如 `<my-element>`，React 会假定你想渲染一个 [自定义 HTML 元素。](https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_custom_elements)

如果你渲染一个带有 [`is`](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/is) 属性的内置浏览器 HTML 元素，它也会被视为自定义元素。

#### 在自定义元素上设置值 {/*attributes-vs-properties*/}

自定义元素有两种向其传递数据的方法：

1) 属性：会显示在标记中，并且只能设置为字符串值
2) 属性（properties）：不会显示在标记中，并且可以设置为任意 JavaScript 值

默认情况下，React 会将 JSX 中绑定的值作为属性传递：

```jsx
<my-element value="Hello, world!"></my-element>
```

传递给自定义元素的非字符串 JavaScript 值默认会被序列化：

```jsx
// 将作为 `"1,2,3"` 传递，等同于 `[1,2,3].toString()` 的输出
<my-element value={[1,2,3]}></my-element>
```

不过，如果某个自定义元素的属性名在构造期间出现在类上，React 就会将其识别为可以向其传递任意值的属性：

<Sandpack>

```js src/index.js hidden
import {MyElement} from './MyElement.js';
import { createRoot } from 'react-dom/client';
import {App} from "./App.js";

customElements.define('my-element', MyElement);

const root = createRoot(document.getElementById('root'))
root.render(<App />);
```

```js src/MyElement.js active
export class MyElement extends HTMLElement {
  constructor() {
    super();
    // 此处的值将被 React 覆盖
    // 当其作为元素初始化时
    this.value = undefined;
  }

  connectedCallback() {
    this.innerHTML = this.value.join(", ");
  }
}
```

```js src/App.js
export function App() {
  return <my-element value={[1,2,3]}></my-element>
}
```

</Sandpack>

#### 监听自定义元素上的事件 {/*custom-element-events*/}

使用自定义元素时，一个常见模式是它们可能会派发 [`CustomEvent`s](https://developer.mozilla.org/en-US/docs/Web/API/CustomEvent)，而不是接受一个在事件发生时调用的函数。你可以在通过 JSX 绑定事件时使用 `on` 前缀来监听这些事件。

<Sandpack>

```js src/index.js hidden
import {MyElement} from './MyElement.js';
import { createRoot } from 'react-dom/client';
import {App} from "./App.js";

customElements.define('my-element', MyElement);

const root = createRoot(document.getElementById('root'))
root.render(<App />);
```

```javascript src/MyElement.js
export class MyElement extends HTMLElement {
  constructor() {
    super();
    this.test = undefined;
    this.emitEvent = this._emitEvent.bind(this);
  }

  _emitEvent() {
    const event = new CustomEvent('speak', {
      detail: {
        message: 'Hello, world!',
      },
    });
    this.dispatchEvent(event);
  }

  connectedCallback() {
    this.el = document.createElement('button');
    this.el.innerText = '问好';
    this.el.addEventListener('click', this.emitEvent);
    this.appendChild(this.el);
  }

  disconnectedCallback() {
    this.el.removeEventListener('click', this.emitEvent);
  }
}
```

```jsx src/App.js active
export function App() {
  return (
    <my-element
      onspeak={e => console.log(e.detail.message)}
    ></my-element>
  )
}
```

</Sandpack>

<Note>

事件区分大小写，并支持连字符（`-`）。监听自定义元素的事件时，请保持事件名称的大小写，并包含所有连字符：

```jsx
// 监听 `say-hi` 事件
<my-element onsay-hi={console.log}></my-element>
// 监听 `sayHi` 事件
<my-element onsayHi={console.log}></my-element>
```

</Note>
---

## 所有 SVG 组件 {/*all-svg-components*/}

React 支持所有内置的浏览器 SVG 组件。这包括：

* [`<a>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/a)
* [`<animate>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/animate)
* [`<animateMotion>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/animateMotion)
* [`<animateTransform>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/animateTransform)
* [`<circle>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/circle)
* [`<clipPath>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/clipPath)
* [`<defs>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/defs)
* [`<desc>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/desc)
* [`<discard>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/discard)
* [`<ellipse>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/ellipse)
* [`<feBlend>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feBlend)
* [`<feColorMatrix>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feColorMatrix)
* [`<feComponentTransfer>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feComponentTransfer)
* [`<feComposite>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feComposite)
* [`<feConvolveMatrix>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feConvolveMatrix)
* [`<feDiffuseLighting>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feDiffuseLighting)
* [`<feDisplacementMap>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feDisplacementMap)
* [`<feDistantLight>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feDistantLight)
* [`<feDropShadow>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feDropShadow)
* [`<feFlood>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feFlood)
* [`<feFuncA>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feFuncA)
* [`<feFuncB>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feFuncB)
* [`<feFuncG>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feFuncG)
* [`<feFuncR>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feFuncR)
* [`<feGaussianBlur>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feGaussianBlur)
* [`<feImage>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feImage)
* [`<feMerge>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feMerge)
* [`<feMergeNode>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feMergeNode)
* [`<feMorphology>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feMorphology)
* [`<feOffset>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feOffset)
* [`<fePointLight>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/fePointLight)
* [`<feSpecularLighting>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feSpecularLighting)
* [`<feSpotLight>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feSpotLight)
* [`<feTile>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feTile)
* [`<feTurbulence>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/feTurbulence)
* [`<filter>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/filter)
* [`<foreignObject>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/foreignObject)
* [`<g>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/g)
* `<hatch>`
* `<hatchpath>`
* [`<image>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/image)
* [`<line>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/line)
* [`<linearGradient>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/linearGradient)
* [`<marker>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/marker)
* [`<mask>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/mask)
* [`<metadata>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/metadata)
* [`<mpath>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/mpath)
* [`<path>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/path)
* [`<pattern>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/pattern)
* [`<polygon>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/polygon)
* [`<polyline>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/polyline)
* [`<radialGradient>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/radialGradient)
* [`<rect>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/rect)
* [`<script>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/script)
* [`<set>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/set)
* [`<stop>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/stop)
* [`<style>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/style)
* [`<svg>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/svg)
* [`<switch>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/switch)
* [`<symbol>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/symbol)
* [`<text>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/text)
* [`<textPath>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/textPath)
* [`<title>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/title)
* [`<tspan>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/tspan)
* [`<use>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/use)
* [`<view>`](https://developer.mozilla.org/en-US/docs/Web/SVG/Element/view)

<Note>

与 [DOM 标准](https://developer.mozilla.org/en-US/docs/Web/API/Document_Object_Model) 类似，React 对属性名使用 `camelCase` 约定。例如，你会写 `tabIndex` 而不是 `tabindex`。你可以使用[在线转换器](https://transform.tools/)将现有的 SVG 转换为 JSX。

带命名空间的属性也必须去掉冒号来书写：

* `xlink:actuate` 变为 `xlinkActuate`。
* `xlink:arcrole` 变为 `xlinkArcrole`。
* `xlink:href` 变为 `xlinkHref`。
* `xlink:role` 变为 `xlinkRole`。
* `xlink:show` 变为 `xlinkShow`。
* `xlink:title` 变为 `xlinkTitle`。
* `xlink:type` 变为 `xlinkType`。
* `xml:base` 变为 `xmlBase`。
* `xml:lang` 变为 `xmlLang`。
* `xml:space` 变为 `xmlSpace`。
* `xmlns:xlink` 变为 `xmlnsXlink`。

</Note>
