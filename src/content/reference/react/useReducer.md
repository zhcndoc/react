---
title: useReducer
---

<Intro>

`useReducer` 是一个 React Hook，可让你向组件添加一个 [reducer](/learn/extracting-state-logic-into-a-reducer)。

```js
const [state, dispatch] = useReducer(reducer, initialArg, init?)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `useReducer(reducer, initialArg, init?)` {/*usereducer*/}

在组件顶层调用 `useReducer`，通过 [reducer](/learn/extracting-state-logic-into-a-reducer) 来管理其状态。

```js
import { useReducer } from 'react';

function reducer(state, action) {
  // ...
}

function MyComponent() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });
  // ...
```

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `reducer`：指定状态如何更新的 reducer 函数。它必须是纯函数，应将 state 和 action 作为参数，并返回下一个 state。state 和 action 可以是任意类型。
* `initialArg`：用于计算初始 state 的值。它可以是任意类型。如何从它计算初始 state 取决于下一个 `init` 参数。
* **可选** `init`：应返回初始 state 的初始化函数。如果未指定，则初始 state 设为 `initialArg`。否则，初始 state 设为调用 `init(initialArg)` 的结果。

#### 返回值 {/*returns*/}

`useReducer` 返回一个恰好包含两个值的数组：

1. 当前 state。在第一次渲染时，它被设为 `init(initialArg)` 或 `initialArg`（如果没有 `init`）。
2. [`dispatch` 函数](#dispatch)，用于将 state 更新为不同的值并触发重新渲染。

#### 注意事项 {/*caveats*/}

* `useReducer` 是一个 Hook，因此你只能在**组件顶层**或你自己的 Hooks 中调用它。你不能在循环或条件语句中调用它。如果你需要那样做，请提取一个新组件并将 state 移入其中。
* `dispatch` 函数具有稳定的身份，因此你经常会看到它被从 Effect 依赖项中省略，但把它包含进去也不会导致 Effect 触发。如果 lint 工具允许你在没有报错的情况下省略某个依赖项，那就是安全的。 [了解更多关于移除 Effect 依赖项的信息。](/learn/removing-effect-dependencies#move-dynamic-objects-and-functions-inside-your-effect)
* 在严格模式下，React 会**调用你的 reducer 和 initializer 两次**，以便 [帮助你发现意外的副作用。](#my-reducer-or-initializer-function-runs-twice) 这是仅限开发环境的行为，不会影响生产环境。如果你的 reducer 和 initializer 是纯的（它们应该如此），这不会影响你的逻辑。两次调用中的一次结果会被忽略。

---

### `dispatch` 函数 {/*dispatch*/}

`useReducer` 返回的 `dispatch` 函数可让你将 state 更新为不同的值并触发重新渲染。你需要将 action 作为 `dispatch` 函数的唯一参数传入：

```js
const [state, dispatch] = useReducer(reducer, { age: 42 });

function handleClick() {
  dispatch({ type: 'incremented_age' });
  // ...
```

React 会将你提供的 `reducer` 函数以及当前 `state` 和你传给 `dispatch` 的 action 一起调用，并将下一次 state 设置为其结果。

#### 参数 {/*dispatch-parameters*/}

* `action`：用户执行的动作。它可以是任意类型的值。按惯例，action 通常是一个对象，包含一个用于标识它的 `type` 属性，以及可选的其他附加信息属性。

#### 返回值 {/*dispatch-returns*/}

`dispatch` 函数没有返回值。

#### 注意事项 {/*setstate-caveats*/}

* `dispatch` 函数**只会更新下一次渲染的 state 变量**。如果你在调用 `dispatch` 函数后读取 state 变量， [你仍然会得到调用前屏幕上的旧值](#ive-dispatched-an-action-but-logging-gives-me-the-old-state-value)。

* 如果你提供的新值与当前 `state` 完全相同，按 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较结果判断，React 将**跳过重新渲染组件及其子组件。** 这是一个优化。React 仍可能需要在忽略结果之前调用你的组件，但这不应影响你的代码。

* React 会[批处理 state 更新。](/learn/queueing-a-series-of-state-updates) 它会在**所有事件处理函数执行完毕**并且调用了它们的 `set` 函数之后更新屏幕。这可以防止在单个事件中发生多次重新渲染。在少数情况下，如果你需要更早地强制 React 更新屏幕，例如为了访问 DOM，你可以使用 [`flushSync`。](/reference/react-dom/flushSync)

---

## 用法 {/*usage*/}

### 向组件添加 reducer {/*adding-a-reducer-to-a-component*/}

在组件顶层调用 `useReducer`，通过 [reducer](/learn/extracting-state-logic-into-a-reducer) 管理 state。

```js [[1, 8, "state"], [2, 8, "dispatch"], [4, 8, "reducer"], [3, 8, "{ age: 42 }"]]
import { useReducer } from 'react';

function reducer(state, action) {
  // ...
}

function MyComponent() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });
  // ...
```

`useReducer` 返回一个恰好包含两个项的数组：

1. 这个 state 变量的<CodeStep step={1}>当前 state</CodeStep>，初始时设置为你提供的<CodeStep step={3}>初始 state</CodeStep>。
2. <CodeStep step={2}>`dispatch` 函数</CodeStep>，用于根据交互来更改它。

要更新屏幕上显示的内容，请使用代表用户所做操作的对象调用<CodeStep step={2}>`dispatch`</CodeStep>，这类操作称为 *action*：

```js [[2, 2, "dispatch"]]
function handleClick() {
  dispatch({ type: 'incremented_age' });
}
```

React 会将当前 state 和 action 传递给你的<CodeStep step={4}>reducer 函数</CodeStep>。你的 reducer 会计算并返回下一次 state。React 会存储该下一次 state，使用它渲染你的组件，并更新 UI。

<Sandpack>

```js
import { useReducer } from 'react';

function reducer(state, action) {
  if (action.type === 'incremented_age') {
    return {
      age: state.age + 1
    };
  }
  throw Error('Unknown action.');
}

export default function Counter() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });

  return (
    <>
      <button onClick={() => {
        dispatch({ type: 'incremented_age' })
      }}>
        增加年龄
      </button>
      <p>你好！你现在 {state.age} 岁。</p>
    </>
  );
}
```

```css
button { display: block; margin-top: 10px; }
```

</Sandpack>

`useReducer` 和 [`useState`](/reference/react/useState) 非常相似，但它允许你将 state 更新逻辑从事件处理函数移到组件外部的单个函数中。阅读更多关于[在 `useState` 和 `useReducer` 之间进行选择。](/learn/extracting-state-logic-into-a-reducer#comparing-usestate-and-usereducer)

---

### 编写 reducer 函数 {/*writing-the-reducer-function*/}

reducer 函数的声明如下：

```js
function reducer(state, action) {
  // ...
}
```

然后你需要填入用于计算并返回下一次 state 的代码。按惯例，通常会把它写成一个 [`switch` 语句。](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/switch) 对于 `switch` 中的每个 `case`，计算并返回某个下一次 state。

```js {4-7,10-13}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      return {
        name: state.name,
        age: state.age + 1
      };
    }
    case 'changed_name': {
      return {
        name: action.nextName,
        age: state.age
      };
    }
  }
  throw Error('Unknown action: ' + action.type);
}
```

action 可以有任意结构。按惯例，通常会传入带有 `type` 属性的对象来标识 action。它应包含 reducer 计算下一次 state 所需的最少信息。

```js {5,9-12}
function Form() {
  const [state, dispatch] = useReducer(reducer, { name: 'Taylor', age: 42 });

  function handleButtonClick() {
    dispatch({ type: 'incremented_age' });
  }

  function handleInputChange(e) {
    dispatch({
      type: 'changed_name',
      nextName: e.target.value
    });
  }
  // ...
```

action 类型名称仅限于你的组件内部使用。[每个 action 描述一次单独的交互，即使这会导致数据发生多处变化。](/learn/extracting-state-logic-into-a-reducer#writing-reducers-well) state 的结构是任意的，但通常它会是一个对象或数组。

阅读[将 state 逻辑提取到 reducer 中](/learn/extracting-state-logic-into-a-reducer)以了解更多内容。

<Pitfall>

state 是只读的。不要修改 state 中的任何对象或数组：

```js {4,5}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // 🚩 不要像这样直接修改 state 中的对象：
      state.age = state.age + 1;
      return state;
    }
```

相反，请始终从 reducer 中返回新对象：

```js {4-8}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // ✅ 相反，返回一个新对象
      return {
        ...state,
        age: state.age + 1
      };
    }
```

阅读[在 state 中更新对象](/learn/updating-objects-in-state)和[在 state 中更新数组](/learn/updating-arrays-in-state)以了解更多内容。

</Pitfall>

<Recipes titleText="Basic useReducer examples" titleId="examples-basic">

#### 表单（对象） {/*form-object*/}

在这个示例中，reducer 管理一个包含两个字段的 state 对象：`name` 和 `age`。

<Sandpack>

```js
import { useReducer } from 'react';

function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      return {
        name: state.name,
        age: state.age + 1
      };
    }
    case 'changed_name': {
      return {
        name: action.nextName,
        age: state.age
      };
    }
  }
  throw Error('Unknown action: ' + action.type);
}

const initialState = { name: 'Taylor', age: 42 };

export default function Form() {
  const [state, dispatch] = useReducer(reducer, initialState);

  function handleButtonClick() {
    dispatch({ type: 'incremented_age' });
  }

  function handleInputChange(e) {
    dispatch({
      type: 'changed_name',
      nextName: e.target.value
    });
  }

  return (
    <>
      <input
        value={state.name}
        onChange={handleInputChange}
      />
      <button onClick={handleButtonClick}>
        增加年龄
      </button>
      <p>你好，{state.name}。你现在 {state.age} 岁。</p>
    </>
  );
}
```

```css
button { display: block; margin-top: 10px; }
```

</Sandpack>

<Solution />

#### 待办事项列表（数组） {/*todo-list-array*/}

在这个示例中，reducer 管理一个任务数组。数组需要在[不发生 mutation 的情况下更新。](/learn/updating-arrays-in-state)

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

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
      throw Error('Unknown action: ' + action.type);
    }
  }
}

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId
    });
  }

  return (
    <>
      <h1>布拉格行程</h1>
      <AddTask
        onAddTask={handleAddTask}
      />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: '参观 Kafka 博物馆', done: true },
  { id: 1, text: '观看木偶表演', done: false },
  { id: 2, text: '列侬墙照片', done: false }
];
```

```js src/AddTask.js hidden
import { useState } from 'react';

export default function AddTask({ onAddTask }) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="添加任务"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        onAddTask(text);
      }}>添加</button>
    </>
  )
}
```

```js src/TaskList.js hidden
import { useState } from 'react';

export default function TaskList({
  tasks,
  onChangeTask,
  onDeleteTask
}) {
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task
            task={task}
            onChange={onChangeTask}
            onDelete={onDeleteTask}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ task, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={e => {
            onChange({
              ...task,
              text: e.target.value
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
          onChange({
            ...task,
            done: e.target.checked
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(task.id)}>
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

#### 使用 Immer 编写简洁的更新逻辑 {/*writing-concise-update-logic-with-immer*/}

如果在不发生 mutation 的情况下更新数组和对象让你觉得很麻烦，你可以使用像 [Immer](https://github.com/immerjs/use-immer#useimmerreducer) 这样的库来减少重复代码。Immer 允许你像在直接修改对象一样编写简洁代码，但其底层会执行不可变更新：

<Sandpack>

```js src/App.js
import { useImmerReducer } from 'use-immer';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

function tasksReducer(draft, action) {
  switch (action.type) {
    case 'added': {
      draft.push({
        id: action.id,
        text: action.text,
        done: false
      });
      break;
    }
    case 'changed': {
      const index = draft.findIndex(t =>
        t.id === action.task.id
      );
      draft[index] = action.task;
      break;
    }
    case 'deleted': {
      return draft.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

export default function TaskApp() {
  const [tasks, dispatch] = useImmerReducer(
    tasksReducer,
    initialTasks
  );

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId
    });
  }

  return (
    <>
      <h1>布拉格行程</h1>
      <AddTask
        onAddTask={handleAddTask}
      />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: '参观 Kafka 博物馆', done: true },
  { id: 1, text: '观看木偶表演', done: false },
  { id: 2, text: '列侬墙照片', done: false },
];
```

```js src/AddTask.js hidden
import { useState } from 'react';

export default function AddTask({ onAddTask }) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="添加任务"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        onAddTask(text);
      }}>添加</button>
    </>
  )
}
```

```js src/TaskList.js hidden
import { useState } from 'react';

export default function TaskList({
  tasks,
  onChangeTask,
  onDeleteTask
}) {
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task
            task={task}
            onChange={onChangeTask}
            onDelete={onDeleteTask}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ task, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={e => {
            onChange({
              ...task,
              text: e.target.value
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
          onChange({
            ...task,
            done: e.target.checked
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(task.id)}>
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

```json package.json
{
  "dependencies": {
    "immer": "1.7.3",
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "use-immer": "0.5.1"
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

<Solution />

</Recipes>

---

### 避免重新创建初始 state {/*avoiding-recreating-the-initial-state*/}

React 会保存初始 state 一次，并在后续渲染中忽略它。

```js
function createInitialState(username) {
  // ...
}

function TodoList({ username }) {
  const [state, dispatch] = useReducer(reducer, createInitialState(username));
  // ...
```

虽然 `createInitialState(username)` 的结果只会用于初次渲染，但你仍然会在每次渲染时调用这个函数。如果它要创建大型数组或执行昂贵计算，这可能会造成浪费。

为了解决这个问题，你可以把它作为 **初始化函数** 作为第三个参数传给 `useReducer`：

```js {6}
function createInitialState(username) {
  // ...
}

function TodoList({ username }) {
  const [state, dispatch] = useReducer(reducer, username, createInitialState);
  // ...
```

注意，你传入的是 `createInitialState`，也就是 *函数本身*，而不是 `createInitialState()`，后者是调用该函数的结果。这样一来，初始 state 在初始化后就不会再次被创建。

在上面的示例中，`createInitialState` 接收一个 `username` 参数。如果你的初始化函数不需要任何信息来计算初始 state，那么你可以将 `null` 作为第二个参数传给 `useReducer`。

<Recipes titleText="传递初始化函数与直接传递初始 state 的区别" titleId="examples-initializer">

#### 传递初始化函数 {/*passing-the-initializer-function*/}

此示例传递了初始化函数，因此 `createInitialState` 函数只会在初始化期间运行。它不会在组件重新渲染时运行，例如当你在输入框中输入内容时。

<Sandpack>

```js src/App.js hidden
import TodoList from './TodoList.js';

export default function App() {
  return <TodoList username="Taylor" />;
}
```

```js src/TodoList.js active
import { useReducer } from 'react';

function createInitialState(username) {
  const initialTodos = [];
  for (let i = 0; i < 50; i++) {
    initialTodos.push({
      id: i,
      text: username + "'s task #" + (i + 1)
    });
  }
  return {
    draft: '',
    todos: initialTodos,
  };
}

function reducer(state, action) {
  switch (action.type) {
    case 'changed_draft': {
      return {
        draft: action.nextDraft,
        todos: state.todos,
      };
    };
    case 'added_todo': {
      return {
        draft: '',
        todos: [{
          id: state.todos.length,
          text: state.draft
        }, ...state.todos]
      }
    }
  }
  throw Error('Unknown action: ' + action.type);
}

export default function TodoList({ username }) {
  const [state, dispatch] = useReducer(
    reducer,
    username,
    createInitialState
  );
  return (
    <>
      <input
        value={state.draft}
        onChange={e => {
          dispatch({
            type: 'changed_draft',
            nextDraft: e.target.value
          })
        }}
      />
      <button onClick={() => {
        dispatch({ type: 'added_todo' });
      }}>添加</button>
      <ul>
        {state.todos.map(item => (
          <li key={item.id}>
            {item.text}
          </li>
        ))}
      </ul>
    </>
  );
}
```

</Sandpack>

<Solution />

#### 直接传递初始 state {/*passing-the-initial-state-directly*/}

此示例**没有**传递初始化函数，因此 `createInitialState` 函数会在每次渲染时运行，例如当你在输入框中输入内容时。行为上没有可观察到的差异，但这段代码效率较低。

<Sandpack>

```js src/App.js hidden
import TodoList from './TodoList.js';

export default function App() {
  return <TodoList username="Taylor" />;
}
```

```js src/TodoList.js active
import { useReducer } from 'react';

function createInitialState(username) {
  const initialTodos = [];
  for (let i = 0; i < 50; i++) {
    initialTodos.push({
      id: i,
      text: username + "'s task #" + (i + 1)
    });
  }
  return {
    draft: '',
    todos: initialTodos,
  };
}

function reducer(state, action) {
  switch (action.type) {
    case 'changed_draft': {
      return {
        draft: action.nextDraft,
        todos: state.todos,
      };
    };
    case 'added_todo': {
      return {
        draft: '',
        todos: [{
          id: state.todos.length,
          text: state.draft
        }, ...state.todos]
      }
    }
  }
  throw Error('Unknown action: ' + action.type);
}

export default function TodoList({ username }) {
  const [state, dispatch] = useReducer(
    reducer,
    createInitialState(username)
  );
  return (
    <>
      <input
        value={state.draft}
        onChange={e => {
          dispatch({
            type: 'changed_draft',
            nextDraft: e.target.value
          })
        }}
      />
      <button onClick={() => {
        dispatch({ type: 'added_todo' });
      }}>添加</button>
      <ul>
        {state.todos.map(item => (
          <li key={item.id}>
            {item.text}
          </li>
        ))}
      </ul>
    </>
  );
}
```

</Sandpack>

<Solution />

</Recipes>

---

## 故障排除 {/*troubleshooting*/}

### 我已经派发了一个 action，但日志里拿到的还是旧的 state 值 {/*ive-dispatched-an-action-but-logging-gives-me-the-old-state-value*/}

调用 `dispatch` 函数**不会改变正在运行的代码中的 state**：

```js {4,5,8}
function handleClick() {
  console.log(state.age);  // 42

  dispatch({ type: 'incremented_age' }); // 请求使用 43 重新渲染
  console.log(state.age);  // 仍然是 42！

  setTimeout(() => {
    console.log(state.age); // 还是 42！
  }, 5000);
}
```

这是因为[state 的行为像快照。](/learn/state-as-a-snapshot)更新 state 会请求使用新的 state 值进行另一次渲染，但不会影响你已经在运行中的事件处理函数里的 `state` JavaScript 变量。

如果你需要推测下一个 state 值，可以自己调用 reducer 手动计算：

```js
const action = { type: 'incremented_age' };
dispatch(action);

const nextState = reducer(state, action);
console.log(state);     // { age: 42 }
console.log(nextState); // { age: 43 }
```

---

### 我已经派发了一个 action，但屏幕没有更新 {/*ive-dispatched-an-action-but-the-screen-doesnt-update*/}

如果下一个 state 等于上一个 state，React 会**忽略你的更新**，这是通过 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较得出的结果。这通常发生在你直接修改 state 中的对象或数组时：

```js {4-5,9-10}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // 🚩 错误：修改已有对象
      state.age++;
      return state;
    }
    case 'changed_name': {
      // 🚩 错误：修改已有对象
      state.name = action.nextName;
      return state;
    }
    // ...
  }
}
```

你修改了已有的 `state` 对象并把它返回了，所以 React 忽略了这次更新。要修复这个问题，你需要确保自己始终是在[更新 state 中的对象](/learn/updating-objects-in-state)和[更新 state 中的数组](/learn/updating-arrays-in-state)，而不是修改它们：

```js {4-8,11-15}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // ✅ 正确：创建一个新对象
      return {
        ...state,
        age: state.age + 1
      };
    }
    case 'changed_name': {
      // ✅ 正确：创建一个新对象
      return {
        ...state,
        name: action.nextName
      };
    }
    // ...
  }
}
```

---

### 在派发后，我的 reducer state 的一部分变成了 undefined {/*a-part-of-my-reducer-state-becomes-undefined-after-dispatching*/}

确保每个 `case` 分支在返回新 state 时都**复制了所有已有字段**：

```js {5}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      return {
        ...state, // 别忘了这个！
        age: state.age + 1
      };
    }
    // ...
```

如果没有上面的 `...state`，返回的下一个 state 只会包含 `age` 字段，而不会有其他任何内容。

---

### 在派发后，我的整个 reducer state 都变成了 undefined {/*my-entire-reducer-state-becomes-undefined-after-dispatching*/}

如果你的 state 意外变成了 `undefined`，很可能是你忘了在某个 case 里 `return` state，或者你的 action 类型没有匹配到任何 `case` 语句。要找出原因，可以在 `switch` 外部抛出一个错误：

```js {10}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // ...
    }
    case 'edited_name': {
      // ...
    }
  }
  throw Error('Unknown action: ' + action.type);
}
```

你也可以使用像 TypeScript 这样的静态类型检查器来发现这类错误。

---

### 我遇到了一个错误：“Too many re-renders” {/*im-getting-an-error-too-many-re-renders*/}

你可能会看到这样的错误：`Too many re-renders. React limits the number of renders to prevent an infinite loop.` 通常这意味着你在*渲染期间*无条件地派发了一个 action，所以你的组件进入了一个循环：渲染、派发（导致渲染）、渲染、派发（导致渲染），如此反复。很多时候，这是因为在指定事件处理函数时写错了：

```js {1-2}
// 🚩 错误：在渲染期间调用处理函数
return <button onClick={handleClick()}>点击我</button>

// ✅ 正确：传递事件处理函数
return <button onClick={handleClick}>点击我</button>

// ✅ 正确：传递一个内联函数
return <button onClick={(e) => handleClick(e)}>点击我</button>
```

如果你找不到这个错误的原因，可以点击控制台中错误旁边的箭头，查看 JavaScript 调用栈，找到导致错误的具体 `dispatch` 函数调用。

---

### 我的 reducer 或 initializer 函数运行了两次 {/*my-reducer-or-initializer-function-runs-twice*/}

在 [Strict Mode](/reference/react/StrictMode) 中，React 会调用你的 reducer 和 initializer 函数两次。这不应该破坏你的代码。

这种**仅在开发环境中**的行为有助于你[保持组件纯粹。](/learn/keeping-components-pure)React 会使用其中一次调用的结果，并忽略另一次调用的结果。只要你的组件、initializer 和 reducer 函数是纯函数，这就不会影响你的逻辑。不过，如果它们不小心变成了不纯函数，这有助于你发现错误。

例如，这个不纯的 reducer 函数会修改 state 中的一个数组：

```js {4-6}
function reducer(state, action) {
  switch (action.type) {
    case 'added_todo': {
      // 🚩 错误：修改 state
      state.todos.push({ id: nextId++, text: action.text });
      return state;
    }
    // ...
  }
}
```

因为 React 会调用你的 reducer 函数两次，你会看到 todo 被添加了两次，这样你就知道这里有错误了。在这个例子中，你可以通过[替换数组而不是修改它](/learn/updating-arrays-in-state#adding-to-an-array)来修复这个错误：

```js {4-11}
function reducer(state, action) {
  switch (action.type) {
    case 'added_todo': {
      // ✅ 正确：用新的 state 替换
      return {
        ...state,
        todos: [
          ...state.todos,
          { id: nextId++, text: action.text }
        ]
      };
    }
    // ...
  }
}
```

现在这个 reducer 函数是纯的了，多调用一次也不会改变行为。这就是为什么 React 调用它两次有助于你发现错误。**只有组件、initializer 和 reducer 函数需要是纯的。**事件处理函数不需要是纯的，所以 React 绝不会调用你的事件处理函数两次。

阅读[保持组件纯粹](/learn/keeping-components-pure)以了解更多。
