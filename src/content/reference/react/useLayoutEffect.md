---
title: useLayoutEffect
---

<Pitfall>

`useLayoutEffect` 可能会损害性能。尽可能优先使用 [`useEffect`](/reference/react/useEffect)。

</Pitfall>

<Intro>

`useLayoutEffect` 是 [`useEffect`](/reference/react/useEffect) 的一个版本，它会在浏览器重绘屏幕之前触发。

```js
useLayoutEffect(setup, dependencies?)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `useLayoutEffect(setup, dependencies?)` {/*useinsertioneffect*/}

调用 `useLayoutEffect` 在浏览器重绘屏幕之前执行布局测量：

```js
import { useState, useRef, useLayoutEffect } from 'react';

function Tooltip() {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0);

  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setTooltipHeight(height);
  }, []);
  // ...
```


[在下面查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `setup`：包含你的 Effect 逻辑的函数。你的 setup 函数也可以选择返回一个 *清理* 函数。在你的 [组件提交](/learn/render-and-commit#step-3-react-commits-changes-to-the-dom) 之前，React 会运行你的 setup 函数。每次依赖项发生变化并提交后，React 会先使用旧值运行清理函数（如果你提供了它），然后再使用新值运行你的 setup 函数。在你的组件从 DOM 中移除之前，React 会运行你的清理函数。

* **可选** `dependencies`：在 `setup` 代码中引用的所有响应式值的列表。响应式值包括 props、state，以及所有直接在组件函数体内声明的变量和函数。如果你的 linter 已为 [React 配置](/learn/editor-setup#linting)，它会验证每个响应式值都被正确指定为依赖项。依赖项列表必须具有固定数量的项目，并且像 `[dep1, dep2, dep3]` 这样内联编写。React 会使用 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较来将每个依赖项与其上一次的值进行比较。如果你省略此参数，你的 Effect 将在组件的每次提交后重新运行。

#### 返回值 {/*returns*/}

`useLayoutEffect` 返回 `undefined`。

#### 注意事项 {/*caveats*/}

* `useLayoutEffect` 是一个 Hook，所以你只能在**组件顶层**或你自己的 Hooks 中调用它。你不能在循环或条件语句中调用它。如果你需要这样做，请提取一个组件并把 Effect 移到那里。

* 当严格模式开启时，React 会在第一次真实 setup 之前**额外运行一次仅限开发环境的 setup+cleanup 循环**。这是一个压力测试，用来确保你的清理逻辑与 setup 逻辑“镜像对应”，并且能停止或撤销 setup 所做的一切。如果这导致问题，请[实现清理函数。](/learn/synchronizing-with-effects#how-to-handle-the-effect-firing-twice-in-development)

* 如果你的某些依赖项是在组件内部定义的对象或函数，那么它们有可能会**导致 Effect 比需要的更频繁地重新运行。**要修复这个问题，请移除不必要的 [对象](/reference/react/useEffect#removing-unnecessary-object-dependencies) 和 [函数](/reference/react/useEffect#removing-unnecessary-function-dependencies) 依赖项。你也可以将 [基于前一个 state 的更新](/reference/react/useEffect#updating-state-based-on-previous-state-from-an-effect) 和 [非响应式逻辑](/reference/react/useEffect#reading-the-latest-props-and-state-from-an-effect) 提取到 Effect 之外。

* Effect **只会在客户端运行。**它们不会在服务器渲染期间运行。

* `useLayoutEffect` 内部的代码以及从它触发的所有 state 更新都会**阻止浏览器重绘屏幕。**过度使用会让你的应用变慢。尽可能优先使用 [`useEffect`.](/reference/react/useEffect)

* 如果你在 `useLayoutEffect` 内触发 state 更新，React 将会立即执行所有剩余的 Effects，包括 `useEffect`。

---

## 用法 {/*usage*/}

### 在浏览器重绘屏幕之前测量布局 {/*measuring-layout-before-the-browser-repaints-the-screen*/}

大多数组件不需要知道它们在屏幕上的位置和大小来决定渲染什么。它们只需要返回一些 JSX。然后浏览器会计算它们的 *布局*（位置和大小）并重绘屏幕。

有时候，这还不够。想象一个悬浮提示在鼠标悬停时出现在某个元素旁边。如果空间足够，提示应显示在元素上方；但如果放不下，就应显示在下方。为了把提示渲染到正确的最终位置，你需要知道它的高度（即判断它能否放在上方）。

为此，你需要分两次渲染：

1. 先在任意位置渲染提示（即使位置不正确）。
2. 测量它的高度并决定把提示放在哪里。
3. 再次以正确的位置渲染提示。

**所有这些都需要在浏览器重绘屏幕之前完成。** 你不希望用户看到提示在移动。调用 `useLayoutEffect` 可以在浏览器重绘屏幕之前执行布局测量：

```js {5-8}
function Tooltip() {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0); // 你还不知道真实高度

  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setTooltipHeight(height); // 现在你知道真实高度了，重新渲染
  }, []);

  // ...在下面的渲染逻辑中使用 tooltipHeight...
}
```

下面是它的工作过程，逐步来看：

1. `Tooltip` 以初始的 `tooltipHeight = 0` 进行渲染（因此提示可能位置错误）。
2. React 将它放入 DOM，并运行 `useLayoutEffect` 中的代码。
3. 你的 `useLayoutEffect` [测量提示内容的高度](https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect)，并立即触发重新渲染。
4. `Tooltip` 再次使用真实的 `tooltipHeight` 进行渲染（因此提示位置正确）。
5. React 将其更新到 DOM 中，浏览器最终显示提示。

将鼠标悬停在下面的按钮上，看看提示会如何根据是否放得下而调整位置：

<Sandpack>

```js
import ButtonWithTooltip from './ButtonWithTooltip.js';

export default function App() {
  return (
    <div>
      <ButtonWithTooltip
        tooltipContent={
          <div>
            This tooltip does not fit above the button.
            <br />
            This is why it's displayed below instead!
          </div>
        }
      >
        Hover over me (tooltip above)
      </ButtonWithTooltip>
      <div style={{ height: 50 }} />
      <ButtonWithTooltip
        tooltipContent={
          <div>This tooltip fits above the button</div>
        }
      >
        Hover over me (tooltip below)
      </ButtonWithTooltip>
      <div style={{ height: 50 }} />
      <ButtonWithTooltip
        tooltipContent={
          <div>This tooltip fits above the button</div>
        }
      >
        Hover over me (tooltip below)
      </ButtonWithTooltip>
    </div>
  );
}
```

```js src/ButtonWithTooltip.js
import { useState, useRef } from 'react';
import Tooltip from './Tooltip.js';

export default function ButtonWithTooltip({ tooltipContent, ...rest }) {
  const [targetRect, setTargetRect] = useState(null);
  const buttonRef = useRef(null);
  return (
    <>
      <button
        {...rest}
        ref={buttonRef}
        onPointerEnter={() => {
          const rect = buttonRef.current.getBoundingClientRect();
          setTargetRect({
            left: rect.left,
            top: rect.top,
            right: rect.right,
            bottom: rect.bottom,
          });
        }}
        onPointerLeave={() => {
          setTargetRect(null);
        }}
      />
      {targetRect !== null && (
        <Tooltip targetRect={targetRect}>
          {tooltipContent}
        </Tooltip>
      )
    }
    </>
  );
}
```

```js src/Tooltip.js active
import { useRef, useLayoutEffect, useState } from 'react';
import { createPortal } from 'react-dom';
import TooltipContainer from './TooltipContainer.js';

export default function Tooltip({ children, targetRect }) {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0);

  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setTooltipHeight(height);
    console.log('测量到的提示高度：' + height);
  }, []);

  let tooltipX = 0;
  let tooltipY = 0;
  if (targetRect !== null) {
    tooltipX = targetRect.left;
    tooltipY = targetRect.top - tooltipHeight;
    if (tooltipY < 0) {
      // 上方放不下，所以放在下方。
      tooltipY = targetRect.bottom;
    }
  }

  return createPortal(
    <TooltipContainer x={tooltipX} y={tooltipY} contentRef={ref}>
      {children}
    </TooltipContainer>,
    document.body
  );
}
```

```js src/TooltipContainer.js
export default function TooltipContainer({ children, x, y, contentRef }) {
  return (
    <div
      style={{
        position: 'absolute',
        pointerEvents: 'none',
        left: 0,
        top: 0,
        transform: `translate3d(${x}px, ${y}px, 0)`
      }}
    >
      <div ref={contentRef} className="tooltip">
        {children}
      </div>
    </div>
  );
}
```

```css
.tooltip {
  color: white;
  background: #222;
  border-radius: 4px;
  padding: 4px;
}
```

</Sandpack>

注意，尽管 `Tooltip` 组件必须分两次渲染（第一次 `tooltipHeight` 初始化为 `0`，然后再使用实际测得的高度渲染），你只会看到最终结果。这就是为什么在这个示例中你需要使用 `useLayoutEffect` 而不是 [`useEffect`](/reference/react/useEffect)。下面我们将详细看看它们之间的区别。

<Recipes titleText="useLayoutEffect vs useEffect" titleId="examples">

#### `useLayoutEffect` 会阻止浏览器重绘 {/*uselayouteffect-blocks-the-browser-from-repainting*/}

React 保证 `useLayoutEffect` 内部的代码以及其中安排的任何 state 更新都会在**浏览器重绘屏幕之前**被处理。这样你就可以渲染提示、测量它，然后再次重新渲染提示，而用户不会注意到第一次额外渲染。换句话说，`useLayoutEffect` 会阻止浏览器绘制。

<Sandpack>

```js
import ButtonWithTooltip from './ButtonWithTooltip.js';

export default function App() {
  return (
    <div>
      <ButtonWithTooltip
        tooltipContent={
          <div>
            This tooltip does not fit above the button.
            <br />
            This is why it's displayed below instead!
          </div>
        }
      >
        Hover over me (tooltip above)
      </ButtonWithTooltip>
      <div style={{ height: 50 }} />
      <ButtonWithTooltip
        tooltipContent={
          <div>This tooltip fits above the button</div>
        }
      >
        Hover over me (tooltip below)
      </ButtonWithTooltip>
      <div style={{ height: 50 }} />
      <ButtonWithTooltip
        tooltipContent={
          <div>This tooltip fits above the button</div>
        }
      >
        Hover over me (tooltip below)
      </ButtonWithTooltip>
    </div>
  );
}
```

```js src/ButtonWithTooltip.js
import { useState, useRef } from 'react';
import Tooltip from './Tooltip.js';

export default function ButtonWithTooltip({ tooltipContent, ...rest }) {
  const [targetRect, setTargetRect] = useState(null);
  const buttonRef = useRef(null);
  return (
    <>
      <button
        {...rest}
        ref={buttonRef}
        onPointerEnter={() => {
          const rect = buttonRef.current.getBoundingClientRect();
          setTargetRect({
            left: rect.left,
            top: rect.top,
            right: rect.right,
            bottom: rect.bottom,
          });
        }}
        onPointerLeave={() => {
          setTargetRect(null);
        }}
      />
      {targetRect !== null && (
        <Tooltip targetRect={targetRect}>
          {tooltipContent}
        </Tooltip>
      )
    }
    </>
  );
}
```

```js src/Tooltip.js active
import { useRef, useLayoutEffect, useState } from 'react';
import { createPortal } from 'react-dom';
import TooltipContainer from './TooltipContainer.js';

export default function Tooltip({ children, targetRect }) {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0);

  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setTooltipHeight(height);
  }, []);

  let tooltipX = 0;
  let tooltipY = 0;
  if (targetRect !== null) {
    tooltipX = targetRect.left;
    tooltipY = targetRect.top - tooltipHeight;
    if (tooltipY < 0) {
      // 上方放不下，所以放在下方。
      tooltipY = targetRect.bottom;
    }
  }

  return createPortal(
    <TooltipContainer x={tooltipX} y={tooltipY} contentRef={ref}>
      {children}
    </TooltipContainer>,
    document.body
  );
}
```

```js src/TooltipContainer.js
export default function TooltipContainer({ children, x, y, contentRef }) {
  return (
    <div
      style={{
        position: 'absolute',
        pointerEvents: 'none',
        left: 0,
        top: 0,
        transform: `translate3d(${x}px, ${y}px, 0)`
      }}
    >
      <div ref={contentRef} className="tooltip">
        {children}
      </div>
    </div>
  );
}
```

```css
.tooltip {
  color: white;
  background: #222;
  border-radius: 4px;
  padding: 4px;
}
```

</Sandpack>

<Solution />

#### `useEffect` 不会阻止浏览器 {/*useeffect-does-not-block-the-browser*/}

下面是同样的示例，但使用 [`useEffect`](/reference/react/useEffect) 而不是 `useLayoutEffect`。如果你使用的是较慢的设备，可能会注意到有时提示会“闪烁”，你会短暂看到它的初始位置，然后才变成修正后的位置。

<Sandpack>

```js
import ButtonWithTooltip from './ButtonWithTooltip.js';

export default function App() {
  return (
    <div>
      <ButtonWithTooltip
        tooltipContent={
          <div>
            This tooltip does not fit above the button.
            <br />
            This is why it's displayed below instead!
          </div>
        }
      >
        Hover over me (tooltip above)
      </ButtonWithTooltip>
      <div style={{ height: 50 }} />
      <ButtonWithTooltip
        tooltipContent={
          <div>This tooltip fits above the button</div>
        }
      >
        Hover over me (tooltip below)
      </ButtonWithTooltip>
      <div style={{ height: 50 }} />
      <ButtonWithTooltip
        tooltipContent={
          <div>This tooltip fits above the button</div>
        }
      >
        Hover over me (tooltip below)
      </ButtonWithTooltip>
    </div>
  );
}
```

```js src/ButtonWithTooltip.js
import { useState, useRef } from 'react';
import Tooltip from './Tooltip.js';

export default function ButtonWithTooltip({ tooltipContent, ...rest }) {
  const [targetRect, setTargetRect] = useState(null);
  const buttonRef = useRef(null);
  return (
    <>
      <button
        {...rest}
        ref={buttonRef}
        onPointerEnter={() => {
          const rect = buttonRef.current.getBoundingClientRect();
          setTargetRect({
            left: rect.left,
            top: rect.top,
            right: rect.right,
            bottom: rect.bottom,
          });
        }}
        onPointerLeave={() => {
          setTargetRect(null);
        }}
      />
      {targetRect !== null && (
        <Tooltip targetRect={targetRect}>
          {tooltipContent}
        </Tooltip>
      )
    }
    </>
  );
}
```

```js src/Tooltip.js active
import { useRef, useEffect, useState } from 'react';
import { createPortal } from 'react-dom';
import TooltipContainer from './TooltipContainer.js';

export default function Tooltip({ children, targetRect }) {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0);

  useEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setTooltipHeight(height);
  }, []);

  let tooltipX = 0;
  let tooltipY = 0;
  if (targetRect !== null) {
    tooltipX = targetRect.left;
    tooltipY = targetRect.top - tooltipHeight;
    if (tooltipY < 0) {
      // 上方放不下，所以放在下方。
      tooltipY = targetRect.bottom;
    }
  }

  return createPortal(
    <TooltipContainer x={tooltipX} y={tooltipY} contentRef={ref}>
      {children}
    </TooltipContainer>,
    document.body
  );
}
```

```js src/TooltipContainer.js
export default function TooltipContainer({ children, x, y, contentRef }) {
  return (
    <div
      style={{
        position: 'absolute',
        pointerEvents: 'none',
        left: 0,
        top: 0,
        transform: `translate3d(${x}px, ${y}px, 0)`
      }}
    >
      <div ref={contentRef} className="tooltip">
        {children}
      </div>
    </div>
  );
}
```

```css
.tooltip {
  color: white;
  background: #222;
  border-radius: 4px;
  padding: 4px;
}
```

</Sandpack>

为了更容易复现这个 bug，这个版本在渲染期间加入了人为延迟。React 会在处理 `useEffect` 内部的 state 更新之前让浏览器先绘制屏幕。因此，提示会闪烁：

<Sandpack>

```js
import ButtonWithTooltip from './ButtonWithTooltip.js';

export default function App() {
  return (
    <div>
      <ButtonWithTooltip
        tooltipContent={
          <div>
            This tooltip does not fit above the button.
            <br />
            This is why it's displayed below instead!
          </div>
        }
      >
        Hover over me (tooltip above)
      </ButtonWithTooltip>
      <div style={{ height: 50 }} />
      <ButtonWithTooltip
        tooltipContent={
          <div>This tooltip fits above the button</div>
        }
      >
        Hover over me (tooltip below)
      </ButtonWithTooltip>
      <div style={{ height: 50 }} />
      <ButtonWithTooltip
        tooltipContent={
          <div>This tooltip fits above the button</div>
        }
      >
        Hover over me (tooltip below)
      </ButtonWithTooltip>
    </div>
  );
}
```

```js {expectedErrors: {'react-compiler': [10, 11]}} src/Tooltip.js active
import { useRef, useEffect, useState } from 'react';
import { createPortal } from 'react-dom';
import TooltipContainer from './TooltipContainer.js';

export default function Tooltip({ children, targetRect }) {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0);

  // 这里人为降低渲染速度
  let now = performance.now();
  while (performance.now() - now < 100) {
    // 先什么都不做一会儿...
  }

  useEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setTooltipHeight(height);
  }, []);

  let tooltipX = 0;
  let tooltipY = 0;
  if (targetRect !== null) {
    tooltipX = targetRect.left;
    tooltipY = targetRect.top - tooltipHeight;
    if (tooltipY < 0) {
      // 上方放不下，所以放在下方。
      tooltipY = targetRect.bottom;
    }
  }

  return createPortal(
    <TooltipContainer x={tooltipX} y={tooltipY} contentRef={ref}>
      {children}
    </TooltipContainer>,
    document.body
  );
}
```

```js src/TooltipContainer.js
export default function TooltipContainer({ children, x, y, contentRef }) {
  return (
    <div
      style={{
        position: 'absolute',
        pointerEvents: 'none',
        left: 0,
        top: 0,
        transform: `translate3d(${x}px, ${y}px, 0)`
      }}
    >
      <div ref={contentRef} className="tooltip">
        {children}
      </div>
    </div>
  );
}
```

```css
.tooltip {
  color: white;
  background: #222;
  border-radius: 4px;
  padding: 4px;
}
```

</Sandpack>

将此示例中的 `useEffect` 改为 `useLayoutEffect`，你会发现即使渲染变慢，它也会阻止绘制。

<Solution />

</Recipes>

<Note>

分两次渲染并阻止浏览器会损害性能。尽量避免这样做。

</Note>

---

## 故障排除 {/*troubleshooting*/}

### 我遇到一个错误："`useLayoutEffect` 在服务器上不起作用" {/*im-getting-an-error-uselayouteffect-does-nothing-on-the-server*/}

`useLayoutEffect` 的目的，是让你的组件[在渲染时使用布局信息：](#measuring-layout-before-the-browser-repaints-the-screen)

1. 渲染初始内容。
2. 在*浏览器重绘屏幕之前*测量布局。
3. 使用你读取到的布局信息渲染最终内容。

当你或你的框架使用[服务端渲染](/reference/react-dom/server)时，你的 React 应用会在首次渲染时于服务器上输出为 HTML。这样你就可以在 JavaScript 代码加载之前先显示初始 HTML。

问题在于，服务器上没有布局信息。

在[前面的示例](#measuring-layout-before-the-browser-repaints-the-screen)中，`Tooltip` 组件里的 `useLayoutEffect` 调用让它能够根据内容高度正确地定位自己（在内容上方或下方）。如果你尝试将 `Tooltip` 作为初始服务器 HTML 的一部分进行渲染，那么这是无法确定的。服务器上还没有布局！因此，即使你在服务器上渲染了它，等 JavaScript 加载并运行后，它的位置也会在客户端“跳动”一下。

通常，依赖布局信息的组件本来也不需要在服务器上渲染。例如，在初始渲染时显示一个 `Tooltip` 可能并没有意义。它是由客户端交互触发的。

不过，如果你遇到了这个问题，可以考虑以下几种不同的方案：

- 将 `useLayoutEffect` 替换为 [`useEffect`。](/reference/react/useEffect) 这会告诉 React，可以在不阻塞绘制的情况下显示初始渲染结果（因为原始 HTML 会在你的 Effect 运行之前先变为可见）。

- 另外一种方式是，[将你的组件标记为仅客户端渲染。](/reference/react/Suspense#providing-a-fallback-for-server-errors-and-client-only-content) 这会告诉 React：在服务端渲染期间，用加载占位内容替换其内容，直到最近的 [`<Suspense>`](/reference/react/Suspense) 边界为止（例如，加载指示器或闪烁占位）。

- 另外，你也可以只在 hydration 之后渲染包含 `useLayoutEffect` 的组件。保留一个布尔值 `isMounted` 状态，初始值为 `false`，然后在 `useEffect` 调用中将其设为 `true`。你的渲染逻辑可以写成 `return isMounted ? <RealContent /> : <FallbackContent />`。在服务器上以及 hydration 过程中，用户会看到 `FallbackContent`，它不应该调用 `useLayoutEffect`。随后 React 会将其替换为仅在客户端运行、并且可以包含 `useLayoutEffect` 调用的 `RealContent`。

- 如果你的组件是与外部数据存储同步，并且依赖 `useLayoutEffect` 的原因并非测量布局，那么可以考虑改用 [`useSyncExternalStore`](/reference/react/useSyncExternalStore)，它[支持服务端渲染。](/reference/react/useSyncExternalStore#adding-support-for-server-rendering)
