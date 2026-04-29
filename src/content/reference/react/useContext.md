---
title: useContext
---

<Intro>

`useContext` 是一个 React Hook，它让你可以从组件中读取并订阅 [context](/learn/passing-data-deeply-with-context)。

```js
const value = useContext(SomeContext)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `useContext(SomeContext)` {/*usecontext*/}

在组件的顶层调用 `useContext`，以读取并订阅 [context。](/learn/passing-data-deeply-with-context)

```js
import { useContext } from 'react';

function MyComponent() {
  const theme = useContext(ThemeContext);
  // ...
```

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `SomeContext`：你之前通过 [`createContext`](/reference/react/createContext) 创建的 context。context 本身并不保存信息，它只表示你可以向组件提供或从组件读取的信息类型。

#### 返回值 {/*returns*/}

`useContext` 返回调用该 Hook 的组件对应的 context 值。它由调用组件在树中上方最近的 `SomeContext` 提供者传入的 `value` 决定。如果没有这样的提供者，那么返回值将是你为该 context 传给 [`createContext`](/reference/react/createContext) 的 `defaultValue`。返回值始终是最新的。如果某个 context 发生变化，React 会自动重新渲染读取该 context 的组件。

#### 注意事项 {/*caveats*/}

* 组件中的 `useContext()` 调用不会受到*同一*组件返回的提供者影响。对应的 `<Context>` **需要位于**调用 `useContext()` 的组件**上方**。
* React 会从接收不同 `value` 的提供者开始，**自动重新渲染**所有使用某个特定 context 的子组件。前后两个值会通过 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 进行比较。使用 [`memo`](/reference/react/memo) 跳过重新渲染，并不会阻止子组件接收新的 context 值。
* 如果你的构建系统在输出中产生了重复模块（使用符号链接时可能发生），这会破坏 context。通过 context 传值只有在你用于提供 context 的 `SomeContext` 和用于读取它的 `SomeContext` **确实是*同一个*对象**时才有效，这一点由 `===` 比较决定。

---

## 用法 {/*usage*/}


### 将数据深入传递到树中 {/*passing-data-deeply-into-the-tree*/}

在组件的顶层调用 `useContext`，以读取并订阅 [context。](/learn/passing-data-deeply-with-context)

```js [[2, 4, "theme"], [1, 4, "ThemeContext"]]
import { useContext } from 'react';

function Button() {
  const theme = useContext(ThemeContext);
  // ...
```

`useContext` 会返回你传入的 <CodeStep step={2}>context 值</CodeStep>，对应于 <CodeStep step={1}>context</CodeStep>。为了确定 context 值，React 会搜索组件树，并找到该特定 context **上方最近的 context 提供者**。

要向 `Button` 传递 context，请将它或其某个父组件包裹在对应的 context 提供者中：

```js [[1, 3, "ThemeContext"], [2, 3, "\\"dark\\""], [1, 5, "ThemeContext"]]
function MyPage() {
  return (
    <ThemeContext value="dark">
      <Form />
    </ThemeContext>
  );
}

function Form() {
  // ... 在内部渲染按钮 ...
}
```

提供者和 `Button` 之间隔着多少层组件都没关系。当 `Form` 内部**任何地方**的 `Button` 调用 `useContext(ThemeContext)` 时，它都会接收到 `"dark"` 作为值。

<Pitfall>

`useContext()` 总是查找调用它的组件**上方**最近的提供者。它会向上搜索，**不会**考虑你调用 `useContext()` 的那个组件内部的提供者。

</Pitfall>

<Sandpack>

```js
import { createContext, useContext } from 'react';

const ThemeContext = createContext(null);

export default function MyApp() {
  return (
    <ThemeContext value="dark">
      <Form />
    </ThemeContext>
  )
}

function Form() {
  return (
    <Panel title="欢迎">
      <Button>注册</Button>
      <Button>登录</Button>
    </Panel>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ children }) {
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button className={className}>
      {children}
    </button>
  );
}
```

```css
.panel-light,
.panel-dark {
  border: 1px solid black;
  border-radius: 4px;
  padding: 20px;
}
.panel-light {
  color: #222;
  background: #fff;
}

.panel-dark {
  color: #fff;
  background: rgb(23, 32, 42);
}

.button-light,
.button-dark {
  border: 1px solid #777;
  padding: 5px;
  margin-right: 10px;
  margin-top: 10px;
}

.button-dark {
  background: #222;
  color: #fff;
}

.button-light {
  background: #fff;
  color: #222;
}
```

</Sandpack>

---

### 更新通过 context 传递的数据 {/*updating-data-passed-via-context*/}

通常，你会希望 context 随时间变化。要更新 context，请将它与 [state](/reference/react/useState) 结合起来。在父组件中声明一个 state 变量，然后将当前 state 作为 <CodeStep step={2}>context 值</CodeStep> 传给提供者。

```js {2} [[1, 4, "ThemeContext"], [2, 4, "theme"], [1, 11, "ThemeContext"]]
function MyPage() {
  const [theme, setTheme] = useState('dark');
  return (
    <ThemeContext value={theme}>
      <Form />
      <Button onClick={() => {
        setTheme('light');
      }}>
        切换到浅色主题
      </Button>
    </ThemeContext>
  );
}
```

现在，提供者内部的任何 `Button` 都会接收到当前的 `theme` 值。如果你调用 `setTheme` 来更新传给提供者的 `theme` 值，所有 `Button` 组件都会随着新的 `'light'` 值重新渲染。

<Recipes titleText="更新 context 的示例" titleId="examples-basic">

#### 通过 context 更新值 {/*updating-a-value-via-context*/}

在这个示例中，`MyApp` 组件持有一个 state 变量，然后将其传给 `ThemeContext` 提供者。勾选“Dark mode”复选框会更新 state。改变所提供的值会使所有使用该 context 的组件重新渲染。

<Sandpack>

```js
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext(null);

export default function MyApp() {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext value={theme}>
      <Form />
      <label>
        <input
          type="checkbox"
          checked={theme === 'dark'}
          onChange={(e) => {
            setTheme(e.target.checked ? 'dark' : 'light')
          }}
        />
        使用深色模式
      </label>
    </ThemeContext>
  )
}

function Form({ children }) {
  return (
    <Panel title="欢迎">
      <Button>注册</Button>
      <Button>登录</Button>
    </Panel>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ children }) {
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button className={className}>
      {children}
    </button>
  );
}
```

```css
.panel-light,
.panel-dark {
  border: 1px solid black;
  border-radius: 4px;
  padding: 20px;
  margin-bottom: 10px;
}
.panel-light {
  color: #222;
  background: #fff;
}

.panel-dark {
  color: #fff;
  background: rgb(23, 32, 42);
}

.button-light,
.button-dark {
  border: 1px solid #777;
  padding: 5px;
  margin-right: 10px;
  margin-top: 10px;
}

.button-dark {
  background: #222;
  color: #fff;
}

.button-light {
  background: #fff;
  color: #222;
}
```

</Sandpack>

注意，`value="dark"` 传递的是 `"dark"` 字符串，而 `value={theme}` 通过 [JSX 花括号](/learn/javascript-in-jsx-with-curly-braces) 传递的是 JavaScript 变量 `theme` 的值。花括号也允许你传递不是字符串的 context 值。

<Solution />

#### 通过 context 更新对象 {/*updating-an-object-via-context*/}

在这个示例中，有一个保存对象的 `currentUser` state 变量。你把 `{ currentUser, setCurrentUser }` 合并成一个对象，并通过 `value={}` 在 context 中向下传递。这使得下方的任何组件，例如 `LoginButton`，都可以同时读取 `currentUser` 和 `setCurrentUser`，并在需要时调用 `setCurrentUser`。

<Sandpack>

```js
import { createContext, useContext, useState } from 'react';

const CurrentUserContext = createContext(null);

export default function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);
  return (
    <CurrentUserContext
      value={{
        currentUser,
        setCurrentUser
      }}
    >
      <Form />
    </CurrentUserContext>
  );
}

function Form({ children }) {
  return (
    <Panel title="欢迎">
      <LoginButton />
    </Panel>
  );
}

function LoginButton() {
  const {
    currentUser,
    setCurrentUser
  } = useContext(CurrentUserContext);

  if (currentUser !== null) {
    return <p>你已登录为 {currentUser.name}。</p>;
  }

  return (
    <Button onClick={() => {
      setCurrentUser({ name: 'Advika' })
    }}>以 Advika 身份登录</Button>
  );
}

function Panel({ title, children }) {
  return (
    <section className="panel">
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ children, onClick }) {
  return (
    <button className="button" onClick={onClick}>
      {children}
    </button>
  );
}
```

```css
label {
  display: block;
}

.panel {
  border: 1px solid black;
  border-radius: 4px;
  padding: 20px;
  margin-bottom: 10px;
}

.button {
  border: 1px solid #777;
  padding: 5px;
  margin-right: 10px;
  margin-top: 10px;
}
```

</Sandpack>

<Solution />

#### 多个 contexts {/*multiple-contexts*/}

在这个示例中，有两个彼此独立的 context。`ThemeContext` 提供当前主题，它是一个字符串；而 `CurrentUserContext` 持有表示当前用户的对象。

<Sandpack>

```js
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext(null);
const CurrentUserContext = createContext(null);

export default function MyApp() {
  const [theme, setTheme] = useState('light');
  const [currentUser, setCurrentUser] = useState(null);
  return (
    <ThemeContext value={theme}>
      <CurrentUserContext
        value={{
          currentUser,
          setCurrentUser
        }}
      >
        <WelcomePanel />
        <label>
          <input
            type="checkbox"
            checked={theme === 'dark'}
            onChange={(e) => {
              setTheme(e.target.checked ? 'dark' : 'light')
            }}
          />
          使用深色模式
        </label>
      </CurrentUserContext>
    </ThemeContext>
  )
}

function WelcomePanel({ children }) {
  const {currentUser} = useContext(CurrentUserContext);
  return (
    <Panel title="欢迎">
      {currentUser !== null ?
        <Greeting /> :
        <LoginForm />
      }
    </Panel>
  );
}

function Greeting() {
  const {currentUser} = useContext(CurrentUserContext);
  return (
    <p>你已登录为 {currentUser.name}。</p>
  )
}

function LoginForm() {
  const {setCurrentUser} = useContext(CurrentUserContext);
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const canLogin = firstName.trim() !== '' && lastName.trim() !== '';
  return (
    <>
      <label>
        名字{': '}
        <input
          required
          value={firstName}
          onChange={e => setFirstName(e.target.value)}
        />
      </label>
      <label>
        姓氏{': '}
        <input
        required
          value={lastName}
          onChange={e => setLastName(e.target.value)}
        />
      </label>
      <Button
        disabled={!canLogin}
        onClick={() => {
          setCurrentUser({
            name: firstName + ' ' + lastName
          });
        }}
      >
        登录
      </Button>
      {!canLogin && <i>请填写两个字段。</i>}
    </>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ children, disabled, onClick }) {
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button
      className={className}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  );
}
```

```css
label {
  display: block;
}

.panel-light,
.panel-dark {
  border: 1px solid black;
  border-radius: 4px;
  padding: 20px;
  margin-bottom: 10px;
}
.panel-light {
  color: #222;
  background: #fff;
}

.panel-dark {
  color: #fff;
  background: rgb(23, 32, 42);
}

.button-light,
.button-dark {
  border: 1px solid #777;
  padding: 5px;
  margin-right: 10px;
  margin-top: 10px;
}

.button-dark {
  background: #222;
  color: #fff;
}

.button-light {
  background: #fff;
  color: #222;
}
```

</Sandpack>

<Solution />

#### 将 providers 提取到一个组件中 {/*extracting-providers-to-a-component*/}

随着应用增长，你很可能会在应用根部附近拥有一“金字塔”式的 contexts。这样并没有问题。不过，如果你从审美上不喜欢这种嵌套，可以将这些提供者提取到单独的组件中。在这个示例中，`MyProviders` 隐藏了这些“管线”细节，并在必要的提供者内部渲染传给它的 children。注意，`theme` 和 `setTheme` state 仍然需要在 `MyApp` 本身中使用，所以 `MyApp` 仍然拥有这部分 state。

<Sandpack>

```js
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext(null);
const CurrentUserContext = createContext(null);

export default function MyApp() {
  const [theme, setTheme] = useState('light');
  return (
    <MyProviders theme={theme} setTheme={setTheme}>
      <WelcomePanel />
      <label>
        <input
          type="checkbox"
          checked={theme === 'dark'}
          onChange={(e) => {
            setTheme(e.target.checked ? 'dark' : 'light')
          }}
        />
        使用深色模式
      </label>
    </MyProviders>
  );
}

function MyProviders({ children, theme, setTheme }) {
  const [currentUser, setCurrentUser] = useState(null);
  return (
    <ThemeContext value={theme}>
      <CurrentUserContext
        value={{
          currentUser,
          setCurrentUser
        }}
      >
        {children}
      </CurrentUserContext>
    </ThemeContext>
  );
}

function WelcomePanel({ children }) {
  const {currentUser} = useContext(CurrentUserContext);
  return (
    <Panel title="欢迎">
      {currentUser !== null ?
        <Greeting /> :
        <LoginForm />
      }
    </Panel>
  );
}

function Greeting() {
  const {currentUser} = useContext(CurrentUserContext);
  return (
    <p>你已登录为 {currentUser.name}。</p>
  )
}

function LoginForm() {
  const {setCurrentUser} = useContext(CurrentUserContext);
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const canLogin = firstName !== '' && lastName !== '';
  return (
    <>
      <label>
        名字{': '}
        <input
          required
          value={firstName}
          onChange={e => setFirstName(e.target.value)}
        />
      </label>
      <label>
        姓氏{': '}
        <input
        required
          value={lastName}
          onChange={e => setLastName(e.target.value)}
        />
      </label>
      <Button
        disabled={!canLogin}
        onClick={() => {
          setCurrentUser({
            name: firstName + ' ' + lastName
          });
        }}
      >
        登录
      </Button>
      {!canLogin && <i>请填写两个字段。</i>}
    </>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ children, disabled, onClick }) {
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button
      className={className}
      disabled={disabled}
      onClick={onClick}
    >
      {children}
    </button>
  );
}
```

```css
label {
  display: block;
}

.panel-light,
.panel-dark {
  border: 1px solid black;
  border-radius: 4px;
  padding: 20px;
  margin-bottom: 10px;
}
.panel-light {
  color: #222;
  background: #fff;
}

.panel-dark {
  color: #fff;
  background: rgb(23, 32, 42);
}

.button-light,
.button-dark {
  border: 1px solid #777;
  padding: 5px;
  margin-right: 10px;
  margin-top: 10px;
}

.button-dark {
  background: #222;
  color: #fff;
}

.button-light {
  background: #fff;
  color: #222;
}
```

</Sandpack>

<Solution />

#### 使用 context 和 reducer 扩展规模 {/*scaling-up-with-context-and-a-reducer*/}

在更大的应用中，通常会将 context 与 [reducer](/reference/react/useReducer) 结合起来，以便将与某些 state 相关的逻辑从组件中提取出来。在这个示例中，所有“连接”细节都隐藏在 `TasksContext.js` 中，其中包含一个 reducer 和两个独立的 context。

阅读这个示例的[完整讲解](/learn/scaling-up-with-reducer-and-context)。

<Sandpack>

```js src/App.js
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import { TasksProvider } from './TasksContext.js';

export default function TaskApp() {
  return (
    <TasksProvider>
      <h1>京都的休息日</h1>
      <AddTask />
      <TaskList />
    </TasksProvider>
  );
}
```

```js src/TasksContext.js
import { createContext, useContext, useReducer } from 'react';

const TasksContext = createContext(null);

const TasksDispatchContext = createContext(null);

export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  return (
    <TasksContext value={tasks}>
      <TasksDispatchContext value={dispatch}>
        {children}
      </TasksDispatchContext>
    </TasksContext>
  );
}

export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('未知操作：' + action.type);
    }
  }
}

const initialTasks = [
  { id: 0, text: 'Philosopher’s Path', done: true },
  { id: 1, text: 'Visit the temple', done: false },
  { id: 2, text: 'Drink matcha', done: false }
];
```

```js src/AddTask.js
import { useState, useContext } from 'react';
import { useTasksDispatch } from './TasksContext.js';

export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useTasksDispatch();
  return (
    <>
      <input
        placeholder="Add task"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        dispatch({
          type: 'added',
          id: nextId++,
          text: text,
        });
      }}>添加</button>
    </>
  );
}

let nextId = 3;
```

```js src/TaskList.js
import { useState, useContext } from 'react';
import { useTasks, useTasksDispatch } from './TasksContext.js';

export default function TaskList() {
  const tasks = useTasks();
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task task={task} />
        </li>
      ))}
    </ul>
  );
}

function Task({ task }) {
  const [isEditing, setIsEditing] = useState(false);
  const dispatch = useTasksDispatch();
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={e => {
            dispatch({
              type: 'changed',
              task: {
                ...task,
                text: e.target.value
              }
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          保存
        </button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>
          编辑
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={e => {
          dispatch({
            type: 'changed',
            task: {
              ...task,
              done: e.target.checked
            }
          });
        }}
      />
      {taskContent}
      <button onClick={() => {
        dispatch({
          type: 'deleted',
          id: task.id
        });
      }}>
        删除
      </button>
    </label>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
```

</Sandpack>

<Solution />

</Recipes>

---

### 指定回退默认值 {/*specifying-a-fallback-default-value*/}

如果 React 在父组件树中找不到该特定 <CodeStep step={1}>context</CodeStep> 的任何提供者，那么 `useContext()` 返回的 context 值将等于你在[创建该 context](/reference/react/createContext) 时指定的 <CodeStep step={3}>默认值</CodeStep>：

```js [[1, 1, "ThemeContext"], [3, 1, "null"]]
const ThemeContext = createContext(null);
```

默认值**永远不会改变**。如果你想更新 context，请像[上面所述](#updating-data-passed-via-context)那样将它与 state 一起使用。

通常，与其使用 `null`，不如使用更有意义的默认值，例如：

```js [[1, 1, "ThemeContext"], [3, 1, "light"]]
const ThemeContext = createContext('light');
```

这样，如果你不小心渲染了某个没有对应提供者的组件，也不会出错。这也有助于你的组件在测试环境中良好工作，而无需在测试里设置大量提供者。

在下面的示例中，“Toggle theme”按钮始终是浅色的，因为它**位于任何 theme context 提供者之外**，而默认的 context 主题值是 `'light'`。尝试将默认主题改为 `'dark'`。

<Sandpack>

```js
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext('light');

export default function MyApp() {
  const [theme, setTheme] = useState('light');
  return (
    <>
      <ThemeContext value={theme}>
        <Form />
      </ThemeContext>
      <Button onClick={() => {
        setTheme(theme === 'dark' ? 'light' : 'dark');
      }}>
        切换主题
      </Button>
    </>
  )
}

function Form({ children }) {
  return (
    <Panel title="欢迎">
      <Button>注册</Button>
      <Button>登录</Button>
    </Panel>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ children, onClick }) {
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button className={className} onClick={onClick}>
      {children}
    </button>
  );
}
```

```css
.panel-light,
.panel-dark {
  border: 1px solid black;
  border-radius: 4px;
  padding: 20px;
  margin-bottom: 10px;
}
.panel-light {
  color: #222;
  background: #fff;
}

.panel-dark {
  color: #fff;
  background: rgb(23, 32, 42);
}

.button-light,
.button-dark {
  border: 1px solid #777;
  padding: 5px;
  margin-right: 10px;
  margin-top: 10px;
}

.button-dark {
  background: #222;
  color: #fff;
}

.button-light {
  background: #fff;
  color: #222;
}
```

</Sandpack>

---

### 覆盖树中某一部分的 context {/*overriding-context-for-a-part-of-the-tree*/}

你可以通过把树的某一部分包裹在具有不同值的提供者中，来覆盖该部分的 context。

```js {3,5}
<ThemeContext value="dark">
  ...
  <ThemeContext value="light">
    <Footer />
  </ThemeContext>
  ...
</ThemeContext>
```

你可以根据需要多次嵌套并覆盖提供者。

<Recipes titleText="覆盖 context 的示例">

#### 覆盖主题 {/*overriding-a-theme*/}

这里，`Footer` 内部的按钮接收到的 context 值（`"light"`）与外部按钮（`"dark"`）不同。

<Sandpack>

```js
import { createContext, useContext } from 'react';

const ThemeContext = createContext(null);

export default function MyApp() {
  return (
    <ThemeContext value="dark">
      <Form />
    </ThemeContext>
  )
}

function Form() {
  return (
    <Panel title="欢迎">
      <Button>注册</Button>
      <Button>登录</Button>
      <ThemeContext value="light">
        <Footer />
      </ThemeContext>
    </Panel>
  );
}

function Footer() {
  return (
    <footer>
      <Button>设置</Button>
    </footer>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      {title && <h1>{title}</h1>}
      {children}
    </section>
  )
}

function Button({ children }) {
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button className={className}>
      {children}
    </button>
  );
}
```

```css
footer {
  margin-top: 20px;
  border-top: 1px solid #aaa;
}

.panel-light,
.panel-dark {
  border: 1px solid black;
  border-radius: 4px;
  padding: 20px;
}
.panel-light {
  color: #222;
  background: #fff;
}

.panel-dark {
  color: #fff;
  background: rgb(23, 32, 42);
}

.button-light,
.button-dark {
  border: 1px solid #777;
  padding: 5px;
  margin-right: 10px;
  margin-top: 10px;
}

.button-dark {
  background: #222;
  color: #fff;
}

.button-light {
  background: #fff;
  color: #222;
}
```

</Sandpack>

<Solution />

#### 自动嵌套标题 {/*automatically-nested-headings*/}

当你嵌套 context 提供者时，可以“累积”信息。在这个示例中，`Section` 组件会跟踪 `LevelContext`，它指定了 section 嵌套的深度。它从父 section 读取 `LevelContext`，并向其子组件提供一个加一后的 `LevelContext` 数值。因此，`Heading` 组件可以根据自己嵌套在多少个 `Section` 组件内部，自动决定使用 `<h1>`、`<h2>`、`<h3>`、……中的哪一个标签。

阅读这个示例的[详细讲解](/learn/passing-data-deeply-with-context)。

<Sandpack>

```js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading>标题</Heading>
      <Section>
        <Heading>标题</Heading>
        <Heading>标题</Heading>
        <Heading>标题</Heading>
        <Section>
          <Heading>子标题</Heading>
          <Heading>子标题</Heading>
          <Heading>子标题</Heading>
          <Section>
            <Heading>二级子标题</Heading>
            <Heading>二级子标题</Heading>
            <Heading>二级子标题</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

```js src/Section.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext value={level + 1}>
        {children}
      </LevelContext>
    </section>
  );
}
```

```js src/Heading.js
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 0:
      throw Error('Heading 必须位于 Section 内部！');
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('未知层级：' + level);
  }
}
```

```js src/LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(0);
```

```css
.section {
  padding: 10px;
  margin: 5px;
  border-radius: 5px;
  border: 1px solid #aaa;
}
```

</Sandpack>

<Solution />

</Recipes>

---

### 在传递对象和函数时优化重新渲染 {/*optimizing-re-renders-when-passing-objects-and-functions*/}

你可以通过 context 传递任何值，包括对象和函数。

```js [[2, 10, "{ currentUser, login }"]]
function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  function login(response) {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }

  return (
    <AuthContext value={{ currentUser, login }}>
      <Page />
    </AuthContext>
  );
}
```

这里，<CodeStep step={2}>context 值</CodeStep> 是一个包含两个属性的 JavaScript 对象，其中一个属性是函数。每当 `MyApp` 重新渲染时（例如路由更新时），这都会变成一个指向不同函数的*不同*对象，因此 React 也必须重新渲染树中深处所有调用 `useContext(AuthContext)` 的组件。

在较小的应用中，这不是问题。不过，如果底层数据，例如 `currentUser`，没有变化，就没有必要让它们重新渲染。为了帮助 React 利用这一点，你可以用 [`useCallback`](/reference/react/useCallback) 包裹 `login` 函数，并用 [`useMemo`](/reference/react/useMemo) 包裹对象创建。这是一种性能优化：

```js {6,9,11,14,17}
import { useCallback, useMemo } from 'react';

function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  const login = useCallback((response) => {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }, []);

  const contextValue = useMemo(() => ({
    currentUser,
    login
  }), [currentUser, login]);

  return (
    <AuthContext value={contextValue}>
      <Page />
    </AuthContext>
  );
}
```

由于这个改动，即使 `MyApp` 需要重新渲染，调用 `useContext(AuthContext)` 的组件也不必重新渲染，除非 `currentUser` 发生了变化。

了解更多关于 [`useMemo`](/reference/react/useMemo#skipping-re-rendering-of-components) 和 [`useCallback`](/reference/react/useCallback#skipping-re-rendering-of-components) 的内容。

---

## 故障排除 {/*troubleshooting*/}

### 我的组件没有看到来自提供者的值 {/*my-component-doesnt-see-the-value-from-my-provider*/}

这可能由以下几个常见原因导致：

1. 你在调用 `useContext()` 的同一个组件中（或其下方）渲染了 `<SomeContext>`。请将 `<SomeContext>` 移到调用 `useContext()` 的组件之上并置于其外部。
2. 你可能忘记用 `<SomeContext>` 包裹你的组件了，或者你可能把它放在了你以为的树结构中的不同位置。请使用 [React DevTools.](/learn/react-developer-tools) 检查层级是否正确。
3. 你可能遇到了工具链中的某个构建问题，导致提供者组件看到的 `SomeContext` 和读取组件看到的 `SomeContext` 是两个不同的对象。例如，如果你使用了符号链接，就可能发生这种情况。你可以通过将它们分别赋值给 `window.SomeContext1` 和 `window.SomeContext2`，然后在控制台中检查 `window.SomeContext1 === window.SomeContext2` 来验证。如果它们不相同，请在构建工具层面修复该问题。

### 即使默认值不同，我从上下文中始终得到 `undefined` {/*i-am-always-getting-undefined-from-my-context-although-the-default-value-is-different*/}

你的树中可能存在一个没有 `value` 的提供者：

```js {1,2}
// 🚩 无法工作：没有 value 属性
<ThemeContext>
   <Button />
</ThemeContext>
```

如果你忘记指定 `value`，就相当于传入了 `value={undefined}`。

你也可能是误用了不同的属性名：

```js {1,2}
// 🚩 无法工作：属性应该叫 "value"
<ThemeContext theme={theme}>
   <Button />
</ThemeContext>
```

在这两种情况下，你都应该在控制台中看到来自 React 的警告。要修复它们，请将属性命名为 `value`：

```js {1,2}
// ✅ 传递 value 属性
<ThemeContext value={theme}>
   <Button />
</ThemeContext>
```

请注意，[`createContext(defaultValue)` 调用中的默认值](#specifying-a-fallback-default-value) 只有在**上方完全没有匹配的提供者**时才会使用。 如果父级树中的某处存在 `<SomeContext value={undefined}>` 组件，那么调用 `useContext(SomeContext)` 的组件*将会*收到 `undefined` 作为上下文值。
