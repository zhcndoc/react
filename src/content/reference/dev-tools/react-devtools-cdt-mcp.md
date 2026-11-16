---
title: 面向智能体的 React DevTools
version: experimental
---

<Experimental>

**`react-devtools-cdt-mcp` 目前为实验性功能。** 它基于 [Chrome DevTools MCP](https://github.com/ChromeDevTools/chrome-devtools-mcp) 中的[实验性第三方开发者工具 API](https://developer.chrome.com/blog/devtools-for-agents-3p-tools) 构建，其工具和输出都可能随版本变化。

</Experimental>

<Intro>

`react-devtools-cdt-mcp` 为 AI 编程智能体提供与 React DevTools 相同的应用视图：组件树、props、Hooks、所有者、源代码位置和渲染耗时。

```js
import 'react-devtools-cdt-mcp/register';
```

</Intro>

<InlineToc />

---

## 概览 {/*overview*/}

通过 [Chrome DevTools MCP](https://github.com/ChromeDevTools/chrome-devtools-mcp) 操作浏览器的编程智能体可以点击页面、创建快照并读取 DOM，但 DOM 无法说明哪个组件渲染了某个按钮、该组件持有哪些状态，或某次交互为何缓慢。

`react-devtools-cdt-mcp` 是一个轻量级浏览器库，你可以将它导入正在调试的页面。它会在页面中注册一组 React 工具。Chrome DevTools MCP 连接该页面时会自动发现这些工具，智能体随后便可像调用其他 MCP 工具一样调用它们。

<Note>

`react-devtools-cdt-mcp` **不是** MCP 服务器，也不需要放入 MCP 客户端配置。它运行在页面内部。你只需配置 Chrome DevTools MCP 这一个 MCP 服务器。

</Note>

---

## 设置 {/*setup*/}

#### 安装软件包 {/*install-the-package*/}

<TerminalBlock>

npm install --save-dev react-devtools-cdt-mcp

</TerminalBlock>

#### 在 React 之前导入 {/*import-it-before-react*/}

在应用入口文件的最顶部导入 `register` 入口，确保任何导入 `react-dom` 的代码运行**之前**就完成导入：

```js {1}
import 'react-devtools-cdt-mcp/register';

import { createRoot } from 'react-dom/client';
import App from './App';

createRoot(document.getElementById('root')).render(<App />);
```

`react-devtools-cdt-mcp` 必须先于 React 加载。如果在 React 之后导入，工具将无法看到你的组件。

<Pitfall>

`react-devtools-cdt-mcp` 以与 React DevTools 浏览器扩展相同的方式连接 React，并让 React 将每次提交都报告给它。这会降低应用速度，因此只应将它包含在开发构建中。

</Pitfall>

#### 设置 Chrome DevTools MCP {/*set-up-chrome-devtools-mcp*/}

你的智能体需要启用[第三方开发者工具](https://developer.chrome.com/blog/devtools-for-agents-3p-tools)的 [Chrome DevTools MCP](https://github.com/ChromeDevTools/chrome-devtools-mcp)。请按照 [Chrome DevTools MCP 文档](https://github.com/ChromeDevTools/chrome-devtools-mcp#getting-started)完成设置。智能体打开导入了 `react-devtools-cdt-mcp/register` 的页面后，就可以列出并调用这些 React 工具。

---

## 试用 {/*try-it*/}

无需借助智能体也可以查看这些工具的返回结果。下方的 sandbox 会像你的应用一样导入 `react-devtools-cdt-mcp/register`，并在应用下方添加一个小型**工具控制台**。该控制台通过派发 `devtoolstooldiscovery` 事件来发现 React 工具，方式与 Chrome DevTools MCP 相同，并允许你手动调用这些工具。

选择工具、编辑 JSON 参数，然后点击**运行工具**。下面是一个供你跟随操作的调试流程，你可以把自己当成智能体：

1. **查找组件。** 使用 `{"name": "Row"}` 调用 `react_find_components`。每个结果都有一个类似 `"r12"` 的 `uid`，其他工具也接受这个值。
2. **检查组件。** 使用其中一个 uid 和 `"includeHooks": true` 调用 `react_get_component_by_uid`，查看组件的 props 和 Hooks。
3. **从屏幕开始。** 选择 `react_get_component_by_dom_element`，点击**选择元素**，再点击列表中的商品，以获取渲染该商品的宿主组件。
4. **分析交互性能。** 调用 `react_start_profiling`，在应用中点击**加入购物车**，然后调用 `react_stop_profiling`。
5. **查看渲染内容。** 调用 `react_get_trace_overview`，然后针对提交 `0` 调用 `react_get_commit_report`，查看该次提交中哪些组件进行了渲染，以及各自耗时。

<Sandpack>

```js src/App.js active
import { useState } from 'react';

const PRODUCTS = Array.from({ length: 20 }, (_, i) => ({
  id: i,
  name: 'Product ' + (i + 1),
  price: 10 + i,
}));

export default function App() {
  const [query, setQuery] = useState('');
  const visibleProducts = PRODUCTS.filter((product) =>
    product.name.toLowerCase().includes(query.toLowerCase())
  );
  return (
    <main>
      <Cart />
      <input
        value={query}
        onChange={(e) => setQuery(e.target.value)}
        placeholder="Filter products"
      />
      <ProductList products={visibleProducts} />
    </main>
  );
}

function Cart() {
  const [count, setCount] = useState(0);
  return (
    <header>
      <CartBadge count={count} />
      <button onClick={() => setCount((c) => c + 1)}>Add to cart</button>
    </header>
  );
}

function CartBadge({ count }) {
  return <span className="badge">Cart: {count}</span>;
}

function ProductList({ products }) {
  return (
    <ul>
      {products.map((product) => (
        <ProductRow key={product.id} product={product} />
      ))}
    </ul>
  );
}

function ProductRow({ product }) {
  return (
    <li>
      {product.name} <b>${product.price}</b>
    </li>
  );
}
```

```js src/index.js
import 'react-devtools-cdt-mcp/register';

import { createRoot } from 'react-dom/client';
import './styles.css';
import App from './App';
import { mountToolConsole } from './ToolConsole';

createRoot(document.getElementById('root')).render(<App />);
mountToolConsole();
```

```js src/ToolConsole.js hidden
// A tiny stand-in for chrome-devtools-mcp. It discovers the React tool group
// the same way chrome-devtools-mcp does (the `devtoolstooldiscovery` event)
// and lets you call the tools by hand.
function discoverToolGroup() {
  let toolGroup = null;
  const event = new Event('devtoolstooldiscovery');
  event.respondWith = group => {
    toolGroup = group;
  };
  window.dispatchEvent(event);
  return toolGroup;
}

function firstUid(tools, name) {
  const find = tools.find(t => t.name === 'react_find_components');
  const result = find.execute({name});
  return result.results && result.results[0] ? result.results[0].uid : 'r0';
}

function exampleParams(tools, toolName) {
  switch (toolName) {
    case 'react_get_component_tree':
      return {depth: 4};
    case 'react_find_components':
      return {name: 'Row'};
    case 'react_get_component_by_uid':
      return {uid: firstUid(tools, 'App'), includeHooks: true};
    case 'react_get_component_by_dom_element':
      return {element: '(click "Pick element", then click the page)'};
    case 'react_get_component_source':
    case 'react_get_owner_stack_trace':
    case 'react_get_parent_stack':
    case 'react_get_owner_stack':
      return {uid: firstUid(tools, 'CartBadge')};
    case 'react_start_profiling':
      return {traceName: 'add-to-cart'};
    case 'react_stop_profiling':
      return {};
    case 'react_get_trace_overview':
      return {traceName: 'add-to-cart'};
    case 'react_get_commit_report':
      return {traceName: 'add-to-cart', commitIndex: 0};
    default:
      return {};
  }
}

export function mountToolConsole() {
  const toolGroup = discoverToolGroup();
  const container = document.createElement('section');
  container.className = 'tool-console';
  document.body.appendChild(container);
  if (toolGroup === null) {
    container.textContent =
      'No tools found. Import react-devtools-cdt-mcp/register before React.';
    return;
  }
  const {tools} = toolGroup;
  container.innerHTML = `
    <h3>Tool console</h3>
    <select></select>
    <p class="description"></p>
    <textarea rows="3" spellcheck="false"></textarea>
    <div class="actions">
      <button class="run">Run tool</button>
      <button class="pick" hidden>Pick element</button>
    </div>
    <pre class="output"></pre>
  `;
  const select = container.querySelector('select');
  const description = container.querySelector('.description');
  const textarea = container.querySelector('textarea');
  const runButton = container.querySelector('.run');
  const pickButton = container.querySelector('.pick');
  const output = container.querySelector('pre');
  let pickedElement = null;

  for (const tool of tools) {
    const option = document.createElement('option');
    option.value = tool.name;
    option.textContent = tool.name;
    select.appendChild(option);
  }

  function selectTool() {
    const tool = tools.find(t => t.name === select.value);
    description.textContent = tool.description;
    textarea.value = JSON.stringify(exampleParams(tools, tool.name));
    pickButton.hidden = tool.name !== 'react_get_component_by_dom_element';
    pickedElement = null;
  }

  function run() {
    const tool = tools.find(t => t.name === select.value);
    let params;
    try {
      params = JSON.parse(textarea.value || '{}');
    } catch (error) {
      output.textContent = 'Invalid JSON: ' + error.message;
      return;
    }
    if (tool.name === 'react_get_component_by_dom_element') {
      params.element = pickedElement;
    }
    const result = tool.execute(params);
    output.textContent = JSON.stringify(result, null, 2);
  }

  pickButton.addEventListener('click', () => {
    output.textContent = 'Click any element in the app above...';
    document.addEventListener(
      'click',
      event => {
        if (container.contains(event.target)) {
          return;
        }
        event.preventDefault();
        event.stopPropagation();
        pickedElement = event.target;
        textarea.value = JSON.stringify({
          element: '<' + event.target.tagName.toLowerCase() + '>',
        });
        run();
      },
      {capture: true, once: true},
    );
  });
  select.addEventListener('change', selectTool);
  runButton.addEventListener('click', run);
  selectTool();
}
```

```css src/styles.css hidden
header { display: flex; gap: 8px; align-items: center; margin-bottom: 8px; }
.badge { padding: 2px 8px; border-radius: 10px; background: #087ea4; color: white; }
ul { padding-left: 20px; max-height: 160px; overflow: auto; }
.tool-console { margin-top: 20px; padding-top: 12px; border-top: 1px solid #ddd; }
.tool-console select, .tool-console textarea { width: 100%; font-family: monospace; }
.tool-console .actions { display: flex; gap: 8px; margin: 8px 0; }
.tool-console pre { background: #f6f7f9; padding: 8px; max-height: 300px; overflow: auto; }
```

```json package.json hidden
{
  "dependencies": {
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "react-devtools-cdt-mcp": "0.0.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

</Sandpack>

<Note>

此 sandbox 中的工具控制台用于模拟 Chrome DevTools MCP，让你可以试用本页提供的工具。在自己的应用中无需编写这些代码：Chrome DevTools MCP 会发现这些工具，智能体会决定调用哪些工具。

</Note>

---

## 参考 {/*reference*/}

### 工具 {/*tools*/}

`react-devtools-cdt-mcp` 会注册名为 `react` 的工具组。智能体通过 Chrome DevTools MCP 调用这些工具。

* `react_get_component_tree`：返回组件树的快照。
* `react_find_components`：按名称查找组件。
* `react_get_component_by_uid`：返回组件的 props 和 Hooks。
* `react_get_component_by_dom_element`：返回渲染某个 DOM 元素的 React 宿主组件。
* `react_get_parent_stack`：返回组件在树中的挂载位置。
* `react_get_owner_stack`：返回在 JSX 中创建该组件的组件。
* `react_get_owner_stack_trace`：以堆栈跟踪形式返回组件的所有者堆栈。
* `react_get_component_source`：返回组件的定义位置。
* `react_start_profiling`：开始记录渲染耗时。
* `react_stop_profiling`：停止记录渲染耗时。
* `react_get_trace_overview`：返回记录中每次提交的摘要。
* `react_get_commit_report`：返回某次提交中渲染的组件及各自耗时。

组件通过简短且稳定的 uid（例如 `"r5"`）来标识。某个 uid 在不同工具调用和重新渲染之间保持不变，但页面重新加载后会改变。

每个工具都返回一个普通的可序列化对象。如果工具调用失败，则会改为返回包含 `error` 消息的对象。

---

### `register(target?)` {/*register*/}

调用 `register` 可自行安装 React DevTools hook，并向 Chrome DevTools MCP 注册 React 工具，以替代导入 `react-devtools-cdt-mcp/register`。

```js
import { register } from 'react-devtools-cdt-mcp';

register();
```

#### 参数 {/*register-parameters*/}

* **可选** `target`：用于安装 hook 的全局对象。默认为 `globalThis`。

#### 返回值 {/*register-returns*/}

`register` 返回一个包含两个属性的对象：

* `facade`：已安装的 React DevTools hook，以及工具读取的 state。
* `unregister`：停止向 Chrome DevTools MCP 注册 React 工具的函数。

#### 注意事项 {/*register-caveats*/}

* 必须在 React 初始化前调用 `register`，否则 React 不会连接该 hook。
* 对同一个 `target` 再次调用 `register` 会返回现有注册对象。

---

## 故障排查 {/*troubleshooting*/}

#### 智能体看不到 React 工具 {/*my-agent-doesnt-see-any-react-tools*/}

检查 Chrome DevTools MCP 配置中是否启用了[第三方开发者工具](https://developer.chrome.com/blog/devtools-for-agents-3p-tools)。然后确认 Chrome DevTools MCP 当前选择的页面导入了 `react-devtools-cdt-mcp/register`。工具会在页面加载或被选中时按页面发现。

#### 组件树为空 {/*the-component-tree-is-empty*/}

`react-devtools-cdt-mcp/register` 的导入发生在 React 初始化之后。请将其移到入口文件的第一行，在任何会加载 `react-dom` 的导入之前。

#### 收到关于类浏览器环境的错误 {/*im-getting-an-error-about-a-browser-like-environment*/}

如果在浏览器之外导入 `react-devtools-cdt-mcp/register`，例如在服务端渲染或测试期间，导入会抛出错误。请仅在客户端代码中导入，或在合适的位置从软件包根目录调用 `register()`。

#### 源代码位置和所有者堆栈为空 {/*source-locations-and-owner-stacks-are-empty*/}

这些数据只存在于 React 的开发构建中。请确保运行的是应用的开发服务器，而不是生产构建。

#### uid 不再有效 {/*a-uid-stopped-working*/}

uid 在页面重新加载后不会保留。重新加载后，请使用 `react_get_component_tree` 或 `react_find_components` 获取新的 uid。
