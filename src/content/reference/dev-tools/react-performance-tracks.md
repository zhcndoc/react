---
title: React 性能轨道
---

<Intro>

React 性能轨道是一些专门的自定义条目，会显示在浏览器开发者工具的 Performance 面板时间线上。

</Intro>

这些轨道旨在通过将 React 特定事件和指标与其他关键数据源（例如网络请求、JavaScript 执行以及事件循环活动）一起可视化，并在 Performance 面板中的统一时间线上同步展示，帮助开发者全面了解其 React 应用的性能，从而完整掌握应用行为。

<div style={{display: 'flex', justifyContent: 'center', marginBottom: '1rem'}}>
  <img className="w-full light-image" src="/images/docs/performance-tracks/overview.png" alt="React 性能轨道" />
  <img className="w-full dark-image" src="/images/docs/performance-tracks/overview.dark.png" alt="React 性能轨道" />
</div>

<InlineToc />

---

## 用法 {/*usage*/}

React 性能轨道仅在 React 的开发构建和剖析构建中可用：

- **开发**：默认启用。
- **剖析**：默认仅启用 Scheduler 轨道。Components 轨道只会列出那些位于被 [`<Profiler>`](/reference/react/Profiler) 包裹的子树中的 Components。如果你启用了 [React Developer Tools 扩展](/learn/react-developer-tools)，即使没有被 `<Profiler>` 包裹，所有 Components 也都会包含在 Components 轨道中。Server 轨道在 profiling 构建中不可用。

如果启用，这些轨道应当会自动出现在你使用支持 [扩展 API](https://developer.chrome.com/docs/devtools/performance/extension) 的浏览器 Performance 面板记录的追踪中。

<Pitfall>

驱动 React 性能轨道的 profiling 埋点会增加一些额外开销，因此在生产构建中默认禁用。
Server Components 和 Server Requests 轨道仅在开发构建中可用。

</Pitfall>

### 使用 profiling 构建 {/*using-profiling-builds*/}

除了生产构建和开发构建之外，React 还包含一个特殊的 profiling 构建。
要使用 profiling 构建，你需要使用 `react-dom/profiling`，而不是 `react-dom/client`。
我们建议你在构建时通过 bundler alias 将 `react-dom/client` 映射到 `react-dom/profiling`，而不是手动更新每一个 `react-dom/client` 导入。
你的框架可能已经内置支持启用 React 的 profiling 构建。

---

## 轨道 {/*tracks*/}

### Scheduler {/*scheduler*/}

Scheduler 是 React 内部用于管理不同优先级任务的一个概念。这个轨道由 4 个子轨道组成，每个子轨道代表一种特定优先级的工作：

- **Blocking** - 同步更新，可能由用户交互触发。
- **Transition** - 在后台进行的非阻塞工作，通常通过 [`startTransition`](/reference/react/startTransition) 触发。
- **Suspense** - 与 Suspense 边界相关的工作，例如显示回退内容或展示内容。
- **Idle** - 在没有其他更高优先级任务时执行的最低优先级工作。

<div style={{display: 'flex', justifyContent: 'center', marginBottom: '1rem'}}>
  <img className="w-full light-image" src="/images/docs/performance-tracks/scheduler.png" alt="Scheduler 轨道" />
  <img className="w-full dark-image" src="/images/docs/performance-tracks/scheduler.dark.png" alt="Scheduler 轨道" />
</div>

#### 渲染 {/*renders*/}

每次渲染过程都包含多个阶段，你可以在时间线上看到：

- **Update** - 这就是导致新一轮渲染过程的原因。
- **Render** - React 通过调用组件的 render 函数来渲染更新后的子树。你可以在 [Components 轨道](#components) 上看到渲染出的组件子树，它遵循相同的配色方案。
- **Commit** - 在渲染组件之后，React 会将更改提交到 DOM，并运行布局效果，例如 [`useLayoutEffect`](/reference/react/useLayoutEffect)。
- **Remaining Effects** - React 执行已渲染子树的被动效果。这通常发生在绘制之后，也就是 React 运行诸如 [`useEffect`](/reference/react/useEffect) 之类的 Hook 的时候。一个已知的例外是用户交互，例如点击，或其他离散事件。在这种情况下，这个阶段可能会在绘制之前运行。

<div style={{display: 'flex', justifyContent: 'center', marginBottom: '1rem'}}>
  <img className="w-full light-image" src="/images/docs/performance-tracks/scheduler-update.png" alt="Scheduler 轨道：更新" />
  <img className="w-full dark-image" src="/images/docs/performance-tracks/scheduler-update.dark.png" alt="Scheduler 轨道：更新" />
</div>

[了解更多关于渲染和提交的信息](/learn/render-and-commit)。

#### 级联更新 {/*cascading-updates*/}

级联更新是性能回退的模式之一。如果在渲染过程中调度了更新，React 可能会丢弃已完成的工作并开始新的过程。

在开发构建中，React 可以显示是哪个组件调度了新更新。这包括普通更新和级联更新。你可以点击 “Cascading update” 条目来查看增强后的堆栈跟踪，其中还应显示调度更新的方法名称。

<div style={{display: 'flex', justifyContent: 'center', marginBottom: '1rem'}}>
  <img className="w-full light-image" src="/images/docs/performance-tracks/scheduler-cascading-update.png" alt="Scheduler 轨道：级联更新" />
  <img className="w-full dark-image" src="/images/docs/performance-tracks/scheduler-cascading-update.dark.png" alt="Scheduler 轨道：级联更新" />
</div>

[了解更多关于 Effects 的信息](/learn/you-might-not-need-an-effect)。

### Components {/*components*/}

Components 轨道会可视化 React 组件的持续时间。它们以火焰图形式显示，其中每个条目代表对应组件的渲染持续时间以及其所有子代子组件的持续时间。

<div style={{display: 'flex', justifyContent: 'center', marginBottom: '1rem'}}>
  <img className="w-full light-image" src="/images/docs/performance-tracks/components-render.png" alt="Components 轨道：渲染持续时间" />
  <img className="w-full dark-image" src="/images/docs/performance-tracks/components-render.dark.png" alt="Components 轨道：渲染持续时间" />
</div>

与渲染持续时间类似，效果持续时间也以火焰图形式表示，但使用不同的配色方案，以与 Scheduler 轨道上对应的阶段保持一致。

<div style={{display: 'flex', justifyContent: 'center', marginBottom: '1rem'}}>
  <img className="w-full light-image" src="/images/docs/performance-tracks/components-effects.png" alt="Components 轨道：效果持续时间" />
  <img className="w-full dark-image" src="/images/docs/performance-tracks/components-effects.dark.png" alt="Components 轨道：效果持续时间" />
</div>

<Note>

与渲染不同，并非所有效果都会默认显示在 Components 轨道上。

为了保持性能并防止界面杂乱，React 只会显示那些持续时间达到 0.05ms 或更长，或者触发了更新的效果。

</Note>

在渲染和效果阶段可能还会显示其他事件：

- <span style={{padding: '0.125rem 0.25rem', backgroundColor: '#facc15', color: '#1f1f1fff'}}>Mount</span> - 已挂载对应组件渲染或效果的子树。
- <span style={{padding: '0.125rem 0.25rem', backgroundColor: '#facc15', color: '#1f1f1fff'}}>Unmount</span> - 已卸载对应组件渲染或效果的子树。
- <span style={{padding: '0.125rem 0.25rem', backgroundColor: '#facc15', color: '#1f1f1fff'}}>Reconnect</span> - 与 Mount 类似，但仅限于使用了 [`<Activity>`](/reference/react/Activity) 的情况。
- <span style={{padding: '0.125rem 0.25rem', backgroundColor: '#facc15', color: '#1f1f1fff'}}>Disconnect</span> - 与 Unmount 类似，但仅限于使用了 [`<Activity>`](/reference/react/Activity) 的情况。

#### 变更的 props {/*changed-props*/}

在开发构建中，当你点击某个组件渲染条目时，可以检查 props 的潜在变化。你可以利用这些信息来识别不必要的渲染。

<div style={{display: 'flex', justifyContent: 'center', marginBottom: '1rem'}}>
  <img className="w-full light-image" src="/images/docs/performance-tracks/changed-props.png" alt="Components 轨道：变更的 props" />
  <img className="w-full dark-image" src="/images/docs/performance-tracks/changed-props.dark.png" alt="Components 轨道：变更的 props" />
</div>

### Server {/*server*/}

<div style={{display: 'flex', justifyContent: 'center', marginBottom: '1rem'}}>
  <img className="w-full light-image" src="/images/docs/performance-tracks/server-overview.png" alt="React Server 性能轨道" />
  <img className="w-full dark-image" src="/images/docs/performance-tracks/server-overview.dark.png" alt="React Server 性能轨道" />
</div>

#### Server Requests {/*server-requests*/}

Server Requests 轨道会可视化所有最终会进入 React Server Component 的 Promise。这包括任何 `async` 操作，例如调用 `fetch` 或异步 Node.js 文件操作。

React 会尝试将从第三方代码内部发起的 Promise 合并为单个跨度，表示阻塞一方代码的整个操作持续时间。
例如，一个名为 `getUser` 的第三方库方法在内部多次调用 `fetch`，将会显示为一个名为 `getUser` 的单个跨度，而不是显示多个 `fetch` 跨度。

点击跨度会显示 Promise 是在哪里创建的堆栈跟踪，以及 Promise resolve 后的值视图（如果可用）。

被拒绝的 Promise 会以红色显示，并显示其拒绝值。

#### Server Components {/*server-components*/}

Server Components 轨道会可视化 React Server Components 所等待的 Promise 的持续时间。时间以火焰图形式显示，其中每个条目代表对应组件的渲染持续时间以及其所有子代子组件的持续时间。

如果你 await 一个 Promise，React 会显示该 Promise 的持续时间。要查看所有 I/O 操作，请使用 Server Requests 轨道。

不同的颜色用于表示组件渲染的持续时间。颜色越深，持续时间越长。

Server Components 轨道组始终会包含一个 “Primary” 轨道。如果 React 能够并发渲染 Server Components，它会显示额外的 “Parallel” 轨道。
如果并发渲染的 Server Components 超过 8 个，React 会将它们关联到最后一个 “Parallel” 轨道，而不是添加更多轨道。
