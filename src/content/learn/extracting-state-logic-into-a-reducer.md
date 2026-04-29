---
title: 将状态逻辑提取到 Reducer 中
---

<Intro>

包含许多状态更新、并分散在多个事件处理函数中的组件，可能会变得令人不知所措。对于这类情况，你可以把所有状态更新逻辑集中到组件外部的一个单独函数中，这个函数称为 _reducer_。

</Intro>

<YouWillLearn>

- 什么是 reducer 函数
- 如何将 `useState` 重构为 `useReducer`
- 何时使用 reducer
- 如何把它写好

</YouWillLearn>

## 使用 reducer 整合状态逻辑 {/*consolidate-state-logic-with-a-reducer*/}

随着组件变得越来越复杂，一眼看清组件状态有哪些不同更新方式会变得更难。例如，下面的 `TaskApp` 组件在 state 中保存了一个 `tasks` 数组，并使用三个不同的事件处理函数来添加、删除和编辑任务：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, setTasks] = useState(initialTasks);

  function handleAddTask(text) {
    setTasks([
      ...tasks,
      {
        id: nextId++,
        text: text,
        done: false,
      },
    ]);
  }

  function handleChangeTask(task) {
    setTasks(
      tasks.map((t) => {
        if (t.id === task.id) {
          return task;
        } else {
          return t;
        }
      })
    );
  }

  function handleDeleteTask(taskId) {
    setTasks(tasks.filter((t) => t.id !== taskId));
  }

  return (
    <>
      <h1>布拉格行程</h1>
      <AddTask onAddTask={handleAddTask} />
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
  {id: 0, text: '参观 Kafka 博物馆', done: true},
  {id: 1, text: '观看木偶戏', done: false},
  {id: 2, text: '列侬墙照片', done: false},
];
```

```js src/AddTask.js hidden
import { useState } from 'react';

export default function AddTask({onAddTask}) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="添加任务"
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <button
        onClick={() => {
          setText('');
          onAddTask(text);
        }}>
        添加
      </button>
    </>
  );
}
```

```js src/TaskList.js hidden
import { useState } from 'react';

export default function TaskList({tasks, onChangeTask, onDeleteTask}) {
  return (
    <ul>
      {tasks.map((task) => (
        <li key={task.id}>
          <Task task={task} onChange={onChangeTask} onDelete={onDeleteTask} />
        </li>
      ))}
    </ul>
  );
}

function Task({task, onChange, onDelete}) {
  const [isEditing, setIsEditing] = useState(false);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={(e) => {
            onChange({
              ...task,
              text: e.target.value,
            });
          }}
        />
        <button onClick={() => setIsEditing(false)}>保存</button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>编辑</button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={(e) => {
          onChange({
            ...task,
            done: e.target.checked,
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(task.id)}>删除</button>
    </label>
  );
}
```

```css
button {
  margin: 5px;
}
li {
  list-style-type: none;
}
ul,
li {
  margin: 0;
  padding: 0;
}
```

</Sandpack>

它的每个事件处理函数都会调用 `setTasks` 来更新状态。随着组件不断增长，分散在其中的状态逻辑也会越来越多。为了降低这种复杂性，并把所有逻辑放到一个容易访问的地方，你可以把这些状态逻辑移到组件外部的一个单独函数中，**这个函数称为“reducer”。**

Reducer 是处理 state 的另一种方式。你可以通过三个步骤从 `useState` 迁移到 `useReducer`：

1. **从设置 state 改为派发 actions。**
2. **编写 reducer 函数。**
3. **在组件中使用 reducer。**

### 第 1 步：从设置 state 改为派发 actions {/*step-1-move-from-setting-state-to-dispatching-actions*/}

你当前的事件处理函数通过设置 state 来指定 _要做什么_：

```js
function handleAddTask(text) {
  setTasks([
    ...tasks,
    {
      id: nextId++,
      text: text,
      done: false,
    },
  ]);
}

function handleChangeTask(task) {
  setTasks(
    tasks.map((t) => {
      if (t.id === task.id) {
        return task;
      } else {
        return t;
      }
    })
  );
}

function handleDeleteTask(taskId) {
  setTasks(tasks.filter((t) => t.id !== taskId));
}
```

移除所有设置 state 的逻辑后，剩下的就是三个事件处理函数：

- `handleAddTask(text)` 在用户按下“添加”时被调用。
- `handleChangeTask(task)` 在用户切换任务状态或按下“保存”时被调用。
- `handleDeleteTask(taskId)` 在用户按下“删除”时被调用。

使用 reducer 管理 state 和直接设置 state 略有不同。你不是通过事件处理函数告诉 React “要做什么”并设置 state，而是通过事件处理函数派发 “actions” 来说明“用户刚刚做了什么”。（状态更新逻辑会放在别处！）因此，与其通过事件处理函数“设置 `tasks`”，不如派发一个“添加/修改/删除任务”的 action。这更能描述用户的意图。

```js
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
    task: task,
  });
}

function handleDeleteTask(taskId) {
  dispatch({
    type: 'deleted',
    id: taskId,
  });
}
```

传给 `dispatch` 的对象称为一个 “action”：

```js {3-7}
function handleDeleteTask(taskId) {
  dispatch(
    // “action” 对象：
    {
      type: 'deleted',
      id: taskId,
    }
  );
}
```

它就是一个普通的 JavaScript 对象。你可以决定往里面放什么，但通常它应该包含关于 _发生了什么_ 的最少信息。（你会在后面的步骤中添加 `dispatch` 函数本身。）

<Note>

action 对象可以有任意形状。

按照惯例，通常会给它一个描述发生了什么的字符串 `type`，并在其他字段中传递额外信息。`type` 是组件特定的，所以在这个示例中，`'added'` 或 `'added_task'` 都可以。选择一个能说明发生了什么的名称！

```js
dispatch({
  // 组件特定
  type: 'what_happened',
  // 其他字段写在这里
});
```

</Note>

### 第 2 步：编写 reducer 函数 {/*step-2-write-a-reducer-function*/}

reducer 函数就是你放置 state 逻辑的地方。它接收两个参数：当前 state 和 action 对象，然后返回下一个 state：

```js
function yourReducer(state, action) {
  // 返回 React 要设置的下一个 state
}
```

React 会把 state 设置为 reducer 返回的值。

要把这个例子中的状态设置逻辑从事件处理函数迁移到 reducer 函数中，你需要：

1. 将当前 state（`tasks`）声明为第一个参数。
2. 将 `action` 对象声明为第二个参数。
3. 从 reducer 返回 _下一个_ state（React 会把它设置为 state）。

下面是所有状态设置逻辑迁移到 reducer 函数后的样子：

```js
function tasksReducer(tasks, action) {
  if (action.type === 'added') {
    return [
      ...tasks,
      {
        id: action.id,
        text: action.text,
        done: false,
      },
    ];
  } else if (action.type === 'changed') {
    return tasks.map((t) => {
      if (t.id === action.task.id) {
        return action.task;
      } else {
        return t;
      }
    });
  } else if (action.type === 'deleted') {
    return tasks.filter((t) => t.id !== action.id);
  } else {
    throw Error('Unknown action: ' + action.type);
  }
}
```

因为 reducer 函数将 state（`tasks`）作为参数，所以你可以**把它声明在组件外部。** 这样可以减少缩进层级，并让代码更易读。

<Note>

上面的代码使用了 if/else 语句，但在 reducer 内部惯例上会使用 [switch 语句](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/switch)。结果是一样的，不过 switch 语句通常一眼更容易看懂。

在本文件余下部分，我们会像这样使用它们：

```js
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}
```

我们建议把每个 `case` 块都用 `{` 和 `}` 花括号包起来，这样在不同 `case` 中声明的变量就不会互相冲突。并且，`case` 通常应该以 `return` 结尾。如果你忘记 `return`，代码就会“贯穿”到下一个 `case`，这可能导致错误！

如果你还不太熟悉 switch 语句，使用 if/else 也完全没问题。

</Note>

<DeepDive>

#### 为什么 reducer 叫这个名字？ {/*why-are-reducers-called-this-way*/}

虽然 reducer 可以“减少”组件内的代码量，但它实际上是以你可以对数组执行的 [`reduce()`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce) 操作命名的。

`reduce()` 操作可以让你拿到一个数组，并从多个值中“累积”出一个单一值：

```
const arr = [1, 2, 3, 4, 5];
const sum = arr.reduce(
  (result, number) => result + number
); // 1 + 2 + 3 + 4 + 5
```

你传给 `reduce` 的函数称为 “reducer”。它接收 _当前累计结果_ 和 _当前项_，然后返回 _下一个结果_。React 中的 reducer 也是同样的思路：它接收 _当前 state_ 和 _action_，并返回 _下一个 state_。这样一来，它们会随着时间把 action 累积为 state。

你甚至可以使用带有 `initialState` 和 `actions` 数组的 `reduce()` 方法，把 reducer 函数传给它来计算最终 state：

<Sandpack>

```js src/index.js active
import tasksReducer from './tasksReducer.js';

let initialState = [];
let actions = [
  {type: 'added', id: 1, text: '参观 Kafka 博物馆'},
  {type: 'added', id: 2, text: '观看木偶戏'},
  {type: 'deleted', id: 1},
  {type: 'added', id: 3, text: '列侬墙照片'},
];

let finalState = actions.reduce(tasksReducer, initialState);

const output = document.getElementById('output');
output.textContent = JSON.stringify(finalState, null, 2);
```

```js src/tasksReducer.js
export default function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}
```

```html public/index.html
<pre id="output"></pre>
```

</Sandpack>

你大概不需要自己这样做，但这和 React 所做的事情很相似！

</DeepDive>

### 第 3 步：在组件中使用 reducer {/*step-3-use-the-reducer-from-your-component*/}

最后，你需要将 `tasksReducer` 接到你的组件上。从 React 导入 `useReducer` Hook：

```js
import { useReducer } from 'react';
```

然后你就可以用 `useReducer` 替换 `useState`：

```js
const [tasks, setTasks] = useState(initialTasks);
```

像这样改成 `useReducer`：

```js
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

`useReducer` Hook 和 `useState` 类似——你必须传入一个初始 state，它会返回一个有状态的值和一种设置 state 的方式（在这里就是 dispatch 函数）。但它又有一点不同。

`useReducer` Hook 接收两个参数：

1. 一个 reducer 函数
2. 一个初始 state

并返回：

1. 一个有状态的值
2. 一个 dispatch 函数（用于把用户 action “派发”给 reducer）

现在它已经完全连接好了！下面示例中，reducer 声明在组件文件的底部：

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

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
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>布拉格行程</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  {id: 0, text: '参观 Kafka 博物馆', done: true},
  {id: 1, text: '观看木偶戏', done: false},
  {id: 2, text: '列侬墙照片', done: false},
];
```

```js src/AddTask.js hidden
import { useState } from 'react';

export default function AddTask({onAddTask}) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="添加任务"
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <button
        onClick={() => {
          setText('');
          onAddTask(text);
        }}>
        添加
      </button>
    </>
  );
}
```

```js src/TaskList.js hidden
import { useState } from 'react';

export default function TaskList({tasks, onChangeTask, onDeleteTask}) {
  return (
    <ul>
      {tasks.map((task) => (
        <li key={task.id}>
          <Task task={task} onChange={onChangeTask} onDelete={onDeleteTask} />
        </li>
      ))}
    </ul>
  );
}

function Task({task, onChange, onDelete}) {
  const [isEditing, setIsEditing] = useState(false);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={(e) => {
            onChange({
              ...task,
              text: e.target.value,
            });
          }}
        />
        <button onClick={() => setIsEditing(false)}>保存</button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>编辑</button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={(e) => {
          onChange({
            ...task,
            done: e.target.checked,
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(task.id)}>删除</button>
    </label>
  );
}
```

```css
button {
  margin: 5px;
}
li {
  list-style-type: none;
}
ul,
li {
  margin: 0;
  padding: 0;
}
```

</Sandpack>

如果你愿意，甚至可以把 reducer 移到另一个文件中：

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import tasksReducer from './tasksReducer.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

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
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>布拉格行程</h1>
      <AddTask onAddTask={handleAddTask} />
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
  {id: 0, text: '参观 Kafka 博物馆', done: true},
  {id: 1, text: '观看木偶戏', done: false},
  {id: 2, text: '列侬墙照片', done: false},
];
```

```js src/tasksReducer.js
export default function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}
```

```js src/AddTask.js hidden
import { useState } from 'react';

export default function AddTask({onAddTask}) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="添加任务"
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <button
        onClick={() => {
          setText('');
          onAddTask(text);
        }}>
        添加
      </button>
    </>
  );
}
```

```js src/TaskList.js hidden
import { useState } from 'react';

export default function TaskList({tasks, onChangeTask, onDeleteTask}) {
  return (
    <ul>
      {tasks.map((task) => (
        <li key={task.id}>
          <Task task={task} onChange={onChangeTask} onDelete={onDeleteTask} />
        </li>
      ))}
    </ul>
  );
}

function Task({task, onChange, onDelete}) {
  const [isEditing, setIsEditing] = useState(false);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={(e) => {
            onChange({
              ...task,
              text: e.target.value,
            });
          }}
        />
        <button onClick={() => setIsEditing(false)}>保存</button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>编辑</button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={(e) => {
          onChange({
            ...task,
            done: e.target.checked,
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(task.id)}>删除</button>
    </label>
  );
}
```

```css
button {
  margin: 5px;
}
li {
  list-style-type: none;
}
ul,
li {
  margin: 0;
  padding: 0;
}
```

</Sandpack>

把关注点这样分离后，组件逻辑会更容易阅读。现在，事件处理函数只通过派发 actions 来说明 _发生了什么_，而 reducer 函数则决定如何根据这些 action 更新 state。

## 比较 `useState` 和 `useReducer` {/*comparing-usestate-and-usereducer*/}

reducers 也不是没有缺点！下面是一些可以比较的方面：

- **代码量：** 通常来说，使用 `useState` 时你一开始需要写的代码更少。使用 `useReducer` 时，你需要同时编写 reducer 函数和分发 action。不过，如果很多事件处理器以相似的方式修改状态，`useReducer` 可以帮助减少代码量。
- **可读性：** 当状态更新很简单时，`useState` 非常容易阅读。当它们变得更复杂时，它们可能会让组件代码膨胀，并且难以浏览。在这种情况下，`useReducer` 可以让你清晰地把更新逻辑的 _如何做_ 与事件处理器中的 _发生了什么_ 分开。
- **调试：** 当你使用 `useState` 遇到 bug 时，可能很难判断状态是 _在哪里_ 被错误地设置的，以及 _为什么_ 会这样。使用 `useReducer` 时，你可以在 reducer 中添加 console log，查看每一次状态更新，以及它 _为什么_ 发生（是由于哪个 `action`）。如果每个 `action` 都是正确的，那么你就知道错误出在 reducer 逻辑本身。不过，与 `useState` 相比，你需要跟踪更多代码。
- **测试：** reducer 是一个不依赖组件的纯函数。这意味着你可以把它导出，并在隔离环境中单独测试。虽然通常最好是在更真实的环境中测试组件，但对于复杂的状态更新逻辑，断言 reducer 在某个初始状态和 action 下返回特定状态会很有用。
- **个人偏好：** 有些人喜欢 reducer，有些人不喜欢。没关系，这只是偏好问题。你总是可以在 `useState` 和 `useReducer` 之间来回转换：它们是等价的！

如果你经常在某个组件中遇到由于状态更新不正确而导致的 bug，并且希望给代码引入更多结构，我们建议使用 reducer。你不必把 reducer 用在所有地方：可以自由混用！你甚至可以在同一个组件中同时使用 `useState` 和 `useReducer`。

## 编写良好的 reducer {/*writing-reducers-well*/}

在编写 reducer 时，请记住这两个提示：

- **reducer 必须是纯函数。** 与 [state updater 函数](/learn/queueing-a-series-of-state-updates) 类似，reducer 会在渲染期间运行！（action 会被排队，直到下一次渲染。）这意味着 reducer [必须是纯函数](/learn/keeping-components-pure)——相同的输入总是得到相同的输出。它们不应该发送请求、设置超时，或执行任何副作用（即影响组件外部内容的操作）。它们应该在不变异的情况下更新 [对象](/learn/updating-objects-in-state) 和 [数组](/learn/updating-arrays-in-state)。
- **每个 action 描述一次单独的用户交互，即使这会导致数据发生多处变化。** 例如，如果用户在一个由 reducer 管理的、包含五个字段的表单中点击“重置”，分发一个 `reset_form` action 比分发五个独立的 `set_field` action 更合理。如果你在 reducer 中记录每个 action，这些日志应该足够清晰，让你能够重建交互或响应按什么顺序发生。这有助于调试！

## 使用 Immer 编写更简洁的 reducer {/*writing-concise-reducers-with-immer*/}

就像在普通状态中[更新对象](/learn/updating-objects-in-state#write-concise-update-logic-with-immer)和[更新数组](/learn/updating-arrays-in-state#write-concise-update-logic-with-immer)一样，你也可以使用 Immer 库让 reducer 更简洁。这里，[`useImmerReducer`](https://github.com/immerjs/use-immer#useimmerreducer) 让你可以通过 `push` 或 `arr[i] =` 赋值来变异状态：

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
        done: false,
      });
      break;
    }
    case 'changed': {
      const index = draft.findIndex((t) => t.id === action.task.id);
      draft[index] = action.task;
      break;
    }
    case 'deleted': {
      return draft.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('未知的 action: ' + action.type);
    }
  }
}

export default function TaskApp() {
  const [tasks, dispatch] = useImmerReducer(tasksReducer, initialTasks);

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
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>布拉格行程</h1>
      <AddTask onAddTask={handleAddTask} />
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
  {id: 0, text: '参观 Kafka 博物馆', done: true},
  {id: 1, text: '看木偶表演', done: false},
  {id: 2, text: 'Lennon 墙照片', done: false},
];
```

```js src/AddTask.js hidden
import { useState } from 'react';

export default function AddTask({onAddTask}) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="添加任务"
        value={text}
        onChange={(e) => setText(e.target.value)}
      />
      <button
        onClick={() => {
          setText('');
          onAddTask(text);
        }}>
        添加
      </button>
    </>
  );
}
```

```js src/TaskList.js hidden
import { useState } from 'react';

export default function TaskList({tasks, onChangeTask, onDeleteTask}) {
  return (
    <ul>
      {tasks.map((task) => (
        <li key={task.id}>
          <Task task={task} onChange={onChangeTask} onDelete={onDeleteTask} />
        </li>
      ))}
    </ul>
  );
}

function Task({task, onChange, onDelete}) {
  const [isEditing, setIsEditing] = useState(false);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={(e) => {
            onChange({
              ...task,
              text: e.target.value,
            });
          }}
        />
        <button onClick={() => setIsEditing(false)}>保存</button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>编辑</button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={(e) => {
          onChange({
            ...task,
            done: e.target.checked,
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(task.id)}>删除</button>
    </label>
  );
}
```

```css
button {
  margin: 5px;
}
li {
  list-style-type: none;
}
ul,
li {
  margin: 0;
  padding: 0;
}
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

reducer 必须是纯函数，所以它们不应该变异状态。但 Immer 提供了一个特殊的 `draft` 对象，可以安全地对其进行变异。在底层，Immer 会根据你对 `draft` 所做的更改创建一份状态副本。这就是为什么由 `useImmerReducer` 管理的 reducer 可以变异第一个参数，并且不需要返回 state。

<Recap>

- 要从 `useState` 转换到 `useReducer`：
  1. 在事件处理器中分发 action。
  2. 编写一个 reducer 函数，它根据给定的 state 和 action 返回下一个 state。
  3. 用 `useReducer` 替换 `useState`。
- reducer 需要你多写一点代码，但它们有助于调试和测试。
- reducer 必须是纯函数。
- 每个 action 描述一次单独的用户交互。
- 如果你想用变异式风格编写 reducer，就使用 Immer。

</Recap>

<Challenges>

#### 从事件处理器中分发 action {/*dispatch-actions-from-event-handlers*/}

目前，`ContactList.js` 和 `Chat.js` 中的事件处理器都有 `// TODO` 注释。这就是为什么在输入框中输入不起作用，以及点击按钮不会改变所选收件人。

用用于 `dispatch` 对应 action 的代码替换这两个 `// TODO`。要查看 action 的预期形状和类型，请检查 `messengerReducer.js` 中的 reducer。reducer 已经写好了，所以你不需要修改它。你只需要在 `ContactList.js` 和 `Chat.js` 中分发这些 action。

<Hint>

这两个组件中已经可以使用 `dispatch` 函数，因为它是作为 prop 传入的。所以你需要用对应的 action 对象调用 `dispatch`。

要检查 action 对象的形状，你可以查看 reducer，看看它期望看到哪些 `action` 字段。例如，reducer 中的 `changed_selection` 分支是这样的：

```js
case 'changed_selection': {
  return {
    ...state,
    selectedId: action.contactId
  };
}
```

这意味着你的 action 对象应该有 `type: 'changed_selection'`。你还能看到使用了 `action.contactId`，所以你需要在 action 中包含一个 `contactId` 属性。

</Hint>

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';
import { initialState, messengerReducer } from './messengerReducer';

export default function Messenger() {
  const [state, dispatch] = useReducer(messengerReducer, initialState);
  const message = state.message;
  const contact = contacts.find((c) => c.id === state.selectedId);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={state.selectedId}
        dispatch={dispatch}
      />
      <Chat
        key={contact.id}
        message={message}
        contact={contact}
        dispatch={dispatch}
      />
    </div>
  );
}

const contacts = [
  {id: 0, name: 'Taylor', email: 'taylor@mail.com'},
  {id: 1, name: 'Alice', email: 'alice@mail.com'},
  {id: 2, name: 'Bob', email: 'bob@mail.com'},
];
```

```js src/messengerReducer.js
export const initialState = {
  selectedId: 0,
  message: 'Hello',
};

export function messengerReducer(state, action) {
  switch (action.type) {
    case 'changed_selection': {
      return {
        ...state,
        selectedId: action.contactId,
        message: '',
      };
    }
    case 'edited_message': {
      return {
        ...state,
        message: action.message,
      };
    }
    default: {
      throw Error('未知的 action: ' + action.type);
    }
  }
}
```

```js src/ContactList.js
export default function ContactList({contacts, selectedId, dispatch}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map((contact) => (
          <li key={contact.id}>
            <button
              onClick={() => {
                // TODO: 分发 changed_selection
              }}>
              {selectedId === contact.id ? <b>{contact.name}</b> : contact.name}
            </button>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

```js src/Chat.js
import { useState } from 'react';

export default function Chat({contact, message, dispatch}) {
  return (
    <section className="chat">
      <textarea
        value={message}
        placeholder={'与 ' + contact.name 聊天}
        onChange={(e) => {
          // TODO: 分发 edited_message
          // （从 e.target.value 中读取输入值）
        }}
      />
      <br />
      <button>发送给 {contact.email}</button>
    </section>
  );
}
```

```css
.chat,
.contact-list {
  float: left;
  margin-bottom: 20px;
}
ul,
li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

<Solution>

从 reducer 代码中，你可以推断 action 需要长这样：

```js
// 当用户点击 "Alice"
dispatch({
  type: 'changed_selection',
  contactId: 1,
});

// 当用户输入 "Hello!"
dispatch({
  type: 'edited_message',
  message: 'Hello!',
});
```

下面是更新后分发对应消息的示例：

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';
import { initialState, messengerReducer } from './messengerReducer';

export default function Messenger() {
  const [state, dispatch] = useReducer(messengerReducer, initialState);
  const message = state.message;
  const contact = contacts.find((c) => c.id === state.selectedId);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={state.selectedId}
        dispatch={dispatch}
      />
      <Chat
        key={contact.id}
        message={message}
        contact={contact}
        dispatch={dispatch}
      />
    </div>
  );
}

const contacts = [
  {id: 0, name: 'Taylor', email: 'taylor@mail.com'},
  {id: 1, name: 'Alice', email: 'alice@mail.com'},
  {id: 2, name: 'Bob', email: 'bob@mail.com'},
];
```

```js src/messengerReducer.js
export const initialState = {
  selectedId: 0,
  message: 'Hello',
};

export function messengerReducer(state, action) {
  switch (action.type) {
    case 'changed_selection': {
      return {
        ...state,
        selectedId: action.contactId,
        message: '',
      };
    }
    case 'edited_message': {
      return {
        ...state,
        message: action.message,
      };
    }
    default: {
      throw Error('未知的 action: ' + action.type);
    }
  }
}
```

```js src/ContactList.js
export default function ContactList({contacts, selectedId, dispatch}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map((contact) => (
          <li key={contact.id}>
            <button
              onClick={() => {
                dispatch({
                  type: 'changed_selection',
                  contactId: contact.id,
                });
              }}>
              {selectedId === contact.id ? <b>{contact.name}</b> : contact.name}
            </button>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

```js src/Chat.js
import { useState } from 'react';

export default function Chat({contact, message, dispatch}) {
  return (
    <section className="chat">
      <textarea
        value={message}
        placeholder={'与 ' + contact.name 聊天}
        onChange={(e) => {
          dispatch({
            type: 'edited_message',
            message: e.target.value,
          });
        }}
      />
      <br />
      <button>发送给 {contact.email}</button>
    </section>
  );
}
```

```css
.chat,
.contact-list {
  float: left;
  margin-bottom: 20px;
}
ul,
li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

</Solution>

#### 发送消息时清空输入框 {/*clear-the-input-on-sending-a-message*/}

目前，点击“发送”没有任何作用。给“发送”按钮添加一个事件处理器，让它能够：

1. 弹出一个 `alert`，显示收件人的邮箱和消息。
2. 清空消息输入框。

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';
import { initialState, messengerReducer } from './messengerReducer';

export default function Messenger() {
  const [state, dispatch] = useReducer(messengerReducer, initialState);
  const message = state.message;
  const contact = contacts.find((c) => c.id === state.selectedId);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={state.selectedId}
        dispatch={dispatch}
      />
      <Chat
        key={contact.id}
        message={message}
        contact={contact}
        dispatch={dispatch}
      />
    </div>
  );
}

const contacts = [
  {id: 0, name: 'Taylor', email: 'taylor@mail.com'},
  {id: 1, name: 'Alice', email: 'alice@mail.com'},
  {id: 2, name: 'Bob', email: 'bob@mail.com'},
];
```

```js src/messengerReducer.js
export const initialState = {
  selectedId: 0,
  message: 'Hello',
};

export function messengerReducer(state, action) {
  switch (action.type) {
    case 'changed_selection': {
      return {
        ...state,
        selectedId: action.contactId,
        message: '',
      };
    }
    case 'edited_message': {
      return {
        ...state,
        message: action.message,
      };
    }
    default: {
      throw Error('未知的 action: ' + action.type);
    }
  }
}
```

```js src/ContactList.js
export default function ContactList({contacts, selectedId, dispatch}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map((contact) => (
          <li key={contact.id}>
            <button
              onClick={() => {
                dispatch({
                  type: 'changed_selection',
                  contactId: contact.id,
                });
              }}>
              {selectedId === contact.id ? <b>{contact.name}</b> : contact.name}
            </button>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

```js src/Chat.js active
import { useState } from 'react';

export default function Chat({contact, message, dispatch}) {
  return (
    <section className="chat">
      <textarea
        value={message}
        placeholder={'与 ' + contact.name 聊天}
        onChange={(e) => {
          dispatch({
            type: 'edited_message',
            message: e.target.value,
          });
        }}
      />
      <br />
      <button>发送给 {contact.email}</button>
    </section>
  );
}
```

```css
.chat,
.contact-list {
  float: left;
  margin-bottom: 20px;
}
ul,
li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

<Solution>

你可以在“发送”按钮的事件处理器中用几种方式来实现它。一种方法是先显示 alert，然后分发一个 `edited_message` action，把 `message` 设为空：

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';
import { initialState, messengerReducer } from './messengerReducer';

export default function Messenger() {
  const [state, dispatch] = useReducer(messengerReducer, initialState);
  const message = state.message;
  const contact = contacts.find((c) => c.id === state.selectedId);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={state.selectedId}
        dispatch={dispatch}
      />
      <Chat
        key={contact.id}
        message={message}
        contact={contact}
        dispatch={dispatch}
      />
    </div>
  );
}

const contacts = [
  {id: 0, name: 'Taylor', email: 'taylor@mail.com'},
  {id: 1, name: 'Alice', email: 'alice@mail.com'},
  {id: 2, name: 'Bob', email: 'bob@mail.com'},
];
```

```js src/messengerReducer.js
export const initialState = {
  selectedId: 0,
  message: 'Hello',
};

export function messengerReducer(state, action) {
  switch (action.type) {
    case 'changed_selection': {
      return {
        ...state,
        selectedId: action.contactId,
        message: '',
      };
    }
    case 'edited_message': {
      return {
        ...state,
        message: action.message,
      };
    }
    default: {
      throw Error('未知的 action: ' + action.type);
    }
  }
}
```

```js src/ContactList.js
export default function ContactList({contacts, selectedId, dispatch}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map((contact) => (
          <li key={contact.id}>
            <button
              onClick={() => {
                dispatch({
                  type: 'changed_selection',
                  contactId: contact.id,
                });
              }}>
              {selectedId === contact.id ? <b>{contact.name}</b> : contact.name}
            </button>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

```js src/Chat.js active
import { useState } from 'react';

export default function Chat({contact, message, dispatch}) {
  return (
    <section className="chat">
      <textarea
        value={message}
        placeholder={'与 ' + contact.name 聊天}
        onChange={(e) => {
          dispatch({
            type: 'edited_message',
            message: e.target.value,
          });
        }}
      />
      <br />
      <button
        onClick={() => {
          alert(`发送 "${message}" 到 ${contact.email}`);
          dispatch({
            type: 'edited_message',
            message: '',
          });
        }}>
        发送给 {contact.email}
      </button>
    </section>
  );
}
```

```css
.chat,
.contact-list {
  float: left;
  margin-bottom: 20px;
}
ul,
li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

这可以正常工作，并且在你点击“发送”时会清空输入框。

不过，_从用户的角度来看_，发送消息和编辑输入框是两个不同的操作。为了体现这一点，你也可以创建一个新的 action，叫做 `sent_message`，并在 reducer 中单独处理它：

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';
import { initialState, messengerReducer } from './messengerReducer';

export default function Messenger() {
  const [state, dispatch] = useReducer(messengerReducer, initialState);
  const message = state.message;
  const contact = contacts.find((c) => c.id === state.selectedId);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={state.selectedId}
        dispatch={dispatch}
      />
      <Chat
        key={contact.id}
        message={message}
        contact={contact}
        dispatch={dispatch}
      />
    </div>
  );
}

const contacts = [
  {id: 0, name: 'Taylor', email: 'taylor@mail.com'},
  {id: 1, name: 'Alice', email: 'alice@mail.com'},
  {id: 2, name: 'Bob', email: 'bob@mail.com'},
];
```

```js src/messengerReducer.js active
export const initialState = {
  selectedId: 0,
  message: 'Hello',
};

export function messengerReducer(state, action) {
  switch (action.type) {
    case 'changed_selection': {
      return {
        ...state,
        selectedId: action.contactId,
        message: '',
      };
    }
    case 'edited_message': {
      return {
        ...state,
        message: action.message,
      };
    }
    case 'sent_message': {
      return {
        ...state,
        message: '',
      };
    }
    default: {
      throw Error('未知的 action: ' + action.type);
    }
  }
}
```

```js src/ContactList.js
export default function ContactList({contacts, selectedId, dispatch}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map((contact) => (
          <li key={contact.id}>
            <button
              onClick={() => {
                dispatch({
                  type: 'changed_selection',
                  contactId: contact.id,
                });
              }}>
              {selectedId === contact.id ? <b>{contact.name}</b> : contact.name}
            </button>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

```js src/Chat.js active
import { useState } from 'react';

export default function Chat({contact, message, dispatch}) {
  return (
    <section className="chat">
      <textarea
        value={message}
        placeholder={'与 ' + contact.name 聊天}
        onChange={(e) => {
          dispatch({
            type: 'edited_message',
            message: e.target.value,
          });
        }}
      />
      <br />
      <button
        onClick={() => {
          alert(`发送 "${message}" 到 ${contact.email}`);
          dispatch({
            type: 'sent_message',
          });
        }}>
        发送给 {contact.email}
      </button>
    </section>
  );
}
```

```css
.chat,
.contact-list {
  float: left;
  margin-bottom: 20px;
}
ul,
li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

最终行为是一样的。但请记住，action 类型最好描述“用户做了什么”，而不是“你希望状态如何改变”。这样以后添加更多功能会更容易。

无论哪种解决方案，都重要的是你**不要**把 `alert` 放在 reducer 里。reducer 应该是纯函数——它只应计算下一个状态。它不应该“做”任何事情，包括向用户显示消息。这应该发生在事件处理器中。（为了帮助发现这类错误，React 会在 Strict Mode 中多次调用你的 reducer。这就是为什么如果你把 alert 放进 reducer，它会弹出两次。）

</Solution>

#### 在不同标签页之间切换时恢复输入值 {/*restore-input-values-when-switching-between-tabs*/}

在这个例子中，在不同收件人之间切换时总是会清空文本输入框：

```js
case 'changed_selection': {
  return {
    ...state,
    selectedId: action.contactId,
    message: '' // 清空输入框
  };
```

这是因为你不希望多个收件人共享同一个消息草稿。但如果应用能分别“记住”每个联系人的草稿，并在切换联系人时恢复它们，那就更好了。

你的任务是更改 state 的结构，让你能为 _每个联系人_ 单独记住一份消息草稿。你需要对 reducer、初始 state 和组件做一些修改。

<Hint>

你可以把 state 结构设计成这样：

```js
export const initialState = {
  selectedId: 0,
  messages: {
    0: 'Hello, Taylor', // contactId = 0 的草稿
    1: 'Hello, Alice', // contactId = 1 的草稿
  },
};
```

`[key]: value` 的[计算属性](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#computed_property_names)语法可以帮助你更新 `messages` 对象：

```js
{
  ...state.messages,
  [id]: message
}
```

</Hint>

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';
import { initialState, messengerReducer } from './messengerReducer';

export default function Messenger() {
  const [state, dispatch] = useReducer(messengerReducer, initialState);
  const message = state.message;
  const contact = contacts.find((c) => c.id === state.selectedId);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={state.selectedId}
        dispatch={dispatch}
      />
      <Chat
        key={contact.id}
        message={message}
        contact={contact}
        dispatch={dispatch}
      />
    </div>
  );
}

const contacts = [
  {id: 0, name: 'Taylor', email: 'taylor@mail.com'},
  {id: 1, name: 'Alice', email: 'alice@mail.com'},
  {id: 2, name: 'Bob', email: 'bob@mail.com'},
];
```

```js src/messengerReducer.js
export const initialState = {
  selectedId: 0,
  message: 'Hello',
};

export function messengerReducer(state, action) {
  switch (action.type) {
    case 'changed_selection': {
      return {
        ...state,
        selectedId: action.contactId,
        message: '',
      };
    }
    case 'edited_message': {
      return {
        ...state,
        message: action.message,
      };
    }
    case 'sent_message': {
      return {
        ...state,
        message: '',
      };
    }
    default: {
      throw Error('未知的 action: ' + action.type);
    }
  }
}
```

```js src/ContactList.js
export default function ContactList({contacts, selectedId, dispatch}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map((contact) => (
          <li key={contact.id}>
            <button
              onClick={() => {
                dispatch({
                  type: 'changed_selection',
                  contactId: contact.id,
                });
              }}>
              {selectedId === contact.id ? <b>{contact.name}</b> : contact.name}
            </button>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

```js src/Chat.js
import { useState } from 'react';

export default function Chat({contact, message, dispatch}) {
  return (
    <section className="chat">
      <textarea
        value={message}
        placeholder={'与 ' + contact.name 聊天}
        onChange={(e) => {
          dispatch({
            type: 'edited_message',
            message: e.target.value,
          });
        }}
      />
      <br />
      <button
        onClick={() => {
          alert(`发送 "${message}" 到 ${contact.email}`);
          dispatch({
            type: 'sent_message',
          });
        }}>
        发送给 {contact.email}
      </button>
    </section>
  );
}
```

```css
.chat,
.contact-list {
  float: left;
  margin-bottom: 20px;
}
ul,
li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

<Solution>

你需要更新 reducer 来存储并更新每个联系人的独立消息草稿：

```js
// 当输入框被编辑时
case 'edited_message': {
  return {
    // 保留其他 state，比如选中状态
    ...state,
    messages: {
      // 保留其他联系人的消息
      ...state.messages,
      // 但修改当前选中联系人的消息
      [state.selectedId]: action.message
    }
  };
}
```

你还需要更新 `Messenger` 组件，让它读取当前所选联系人的消息：

```js
const message = state.messages[state.selectedId];
```

下面是完整解法：

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';
import { initialState, messengerReducer } from './messengerReducer';

export default function Messenger() {
  const [state, dispatch] = useReducer(messengerReducer, initialState);
  const message = state.messages[state.selectedId];
  const contact = contacts.find((c) => c.id === state.selectedId);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={state.selectedId}
        dispatch={dispatch}
      />
      <Chat
        key={contact.id}
        message={message}
        contact={contact}
        dispatch={dispatch}
      />
    </div>
  );
}

const contacts = [
  {id: 0, name: 'Taylor', email: 'taylor@mail.com'},
  {id: 1, name: 'Alice', email: 'alice@mail.com'},
  {id: 2, name: 'Bob', email: 'bob@mail.com'},
];
```

```js src/messengerReducer.js
export const initialState = {
  selectedId: 0,
  messages: {
    0: 'Hello, Taylor',
    1: 'Hello, Alice',
    2: 'Hello, Bob',
  },
};

export function messengerReducer(state, action) {
  switch (action.type) {
    case 'changed_selection': {
      return {
        ...state,
        selectedId: action.contactId,
      };
    }
    case 'edited_message': {
      return {
        ...state,
        messages: {
          ...state.messages,
          [state.selectedId]: action.message,
        },
      };
    }
    case 'sent_message': {
      return {
        ...state,
        messages: {
          ...state.messages,
          [state.selectedId]: '',
        },
      };
    }
    default: {
      throw Error('未知的 action: ' + action.type);
    }
  }
}
```

```js src/ContactList.js
export default function ContactList({contacts, selectedId, dispatch}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map((contact) => (
          <li key={contact.id}>
            <button
              onClick={() => {
                dispatch({
                  type: 'changed_selection',
                  contactId: contact.id,
                });
              }}>
              {selectedId === contact.id ? <b>{contact.name}</b> : contact.name}
            </button>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

```js src/Chat.js
import { useState } from 'react';

export default function Chat({contact, message, dispatch}) {
  return (
    <section className="chat">
      <textarea
        value={message}
        placeholder={'与 ' + contact.name 聊天}
        onChange={(e) => {
          dispatch({
            type: 'edited_message',
            message: e.target.value,
          });
        }}
      />
      <br />
      <button
        onClick={() => {
          alert(`发送 "${message}" 到 ${contact.email}`);
          dispatch({
            type: 'sent_message',
          });
        }}>
        发送给 {contact.email}
      </button>
    </section>
  );
}
```

```css
.chat,
.contact-list {
  float: left;
  margin-bottom: 20px;
}
ul,
li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

值得注意的是，你并不需要修改任何事件处理器就能实现这种不同的行为。如果没有 reducer，你就必须修改所有会更新状态的事件处理器。

</Solution>

#### 从零实现 `useReducer` {/*implement-usereducer-from-scratch*/}

在前面的示例中，你从 React 导入了 `useReducer` Hook。这次，你要自己实现 _`useReducer` Hook 本身！_ 下面有一个起始模板。代码不应该超过 10 行。

要测试你的修改，可以尝试在输入框中输入或者选择一个联系人。

<Hint>

下面是实现的更详细草图：

```js
export function useReducer(reducer, initialState) {
  const [state, setState] = useState(initialState);

  function dispatch(action) {
    // ???
  }

  return [state, dispatch];
}
```

回忆一下，reducer 函数接受两个参数——当前 state 和 action 对象——然后返回下一个 state。你的 `dispatch` 实现应该如何处理它？

</Hint>

<Sandpack>

```js src/App.js
import { useReducer } from './MyReact.js';
import Chat from './Chat.js';
import ContactList from './ContactList.js';
import { initialState, messengerReducer } from './messengerReducer';

export default function Messenger() {
  const [state, dispatch] = useReducer(messengerReducer, initialState);
  const message = state.messages[state.selectedId];
  const contact = contacts.find((c) => c.id === state.selectedId);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={state.selectedId}
        dispatch={dispatch}
      />
      <Chat
        key={contact.id}
        message={message}
        contact={contact}
        dispatch={dispatch}
      />
    </div>
  );
}

const contacts = [
  {id: 0, name: 'Taylor', email: 'taylor@mail.com'},
  {id: 1, name: 'Alice', email: 'alice@mail.com'},
  {id: 2, name: 'Bob', email: 'bob@mail.com'},
];
```

```js src/messengerReducer.js
export const initialState = {
  selectedId: 0,
  messages: {
    0: 'Hello, Taylor',
    1: 'Hello, Alice',
    2: 'Hello, Bob',
  },
};

export function messengerReducer(state, action) {
  switch (action.type) {
    case 'changed_selection': {
      return {
        ...state,
        selectedId: action.contactId,
      };
    }
    case 'edited_message': {
      return {
        ...state,
        messages: {
          ...state.messages,
          [state.selectedId]: action.message,
        },
      };
    }
    case 'sent_message': {
      return {
        ...state,
        messages: {
          ...state.messages,
          [state.selectedId]: '',
        },
      };
    }
    default: {
      throw Error('未知的 action: ' + action.type);
    }
  }
}
```

```js src/MyReact.js active
import { useState } from 'react';

export function useReducer(reducer, initialState) {
  const [state, setState] = useState(initialState);

  // ???

  return [state, dispatch];
}
```

```js src/ContactList.js hidden
export default function ContactList({contacts, selectedId, dispatch}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map((contact) => (
          <li key={contact.id}>
            <button
              onClick={() => {
                dispatch({
                  type: 'changed_selection',
                  contactId: contact.id,
                });
              }}>
              {selectedId === contact.id ? <b>{contact.name}</b> : contact.name}
            </button>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

```js src/Chat.js hidden
import { useState } from 'react';

export default function Chat({contact, message, dispatch}) {
  return (
    <section className="chat">
      <textarea
        value={message}
        placeholder={'与 ' + contact.name 聊天}
        onChange={(e) => {
          dispatch({
            type: 'edited_message',
            message: e.target.value,
          });
        }}
      />
      <br />
      <button
        onClick={() => {
          alert(`发送 "${message}" 到 ${contact.email}`);
          dispatch({
            type: 'sent_message',
          });
        }}>
        发送给 {contact.email}
      </button>
    </section>
  );
}
```

```css
.chat,
.contact-list {
  float: left;
  margin-bottom: 20px;
}
ul,
li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

<Solution>

分发一个 action 会用当前 state 和 action 调用 reducer，并将结果存储为下一个 state。代码大致如下：

<Sandpack>

```js src/App.js
import { useReducer } from './MyReact.js';
import Chat from './Chat.js';
import ContactList from './ContactList.js';
import { initialState, messengerReducer } from './messengerReducer';

export default function Messenger() {
  const [state, dispatch] = useReducer(messengerReducer, initialState);
  const message = state.messages[state.selectedId];
  const contact = contacts.find((c) => c.id === state.selectedId);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={state.selectedId}
        dispatch={dispatch}
      />
      <Chat
        key={contact.id}
        message={message}
        contact={contact}
        dispatch={dispatch}
      />
    </div>
  );
}

const contacts = [
  {id: 0, name: 'Taylor', email: 'taylor@mail.com'},
  {id: 1, name: 'Alice', email: 'alice@mail.com'},
  {id: 2, name: 'Bob', email: 'bob@mail.com'},
];
```

```js src/messengerReducer.js
export const initialState = {
  selectedId: 0,
  messages: {
    0: 'Hello, Taylor',
    1: 'Hello, Alice',
    2: 'Hello, Bob',
  },
};

export function messengerReducer(state, action) {
  switch (action.type) {
    case 'changed_selection': {
      return {
        ...state,
        selectedId: action.contactId,
      };
    }
    case 'edited_message': {
      return {
        ...state,
        messages: {
          ...state.messages,
          [state.selectedId]: action.message,
        },
      };
    }
    case 'sent_message': {
      return {
        ...state,
        messages: {
          ...state.messages,
          [state.selectedId]: '',
        },
      };
    }
    default: {
      throw Error('未知的 action: ' + action.type);
    }
  }
}
```

```js src/MyReact.js active
import { useState } from 'react';

export function useReducer(reducer, initialState) {
  const [state, setState] = useState(initialState);

  function dispatch(action) {
    const nextState = reducer(state, action);
    setState(nextState);
  }

  return [state, dispatch];
}
```

```js src/ContactList.js hidden
export default function ContactList({contacts, selectedId, dispatch}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map((contact) => (
          <li key={contact.id}>
            <button
              onClick={() => {
                dispatch({
                  type: 'changed_selection',
                  contactId: contact.id,
                });
              }}>
              {selectedId === contact.id ? <b>{contact.name}</b> : contact.name}
            </button>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

```js src/Chat.js hidden
import { useState } from 'react';

export default function Chat({contact, message, dispatch}) {
  return (
    <section className="chat">
      <textarea
        value={message}
        placeholder={'与 ' + contact.name 聊天}
        onChange={(e) => {
          dispatch({
            type: 'edited_message',
            message: e.target.value,
          });
        }}
      />
      <br />
      <button
        onClick={() => {
          alert(`发送 "${message}" 到 ${contact.email}`);
          dispatch({
            type: 'sent_message',
          });
        }}>
        发送给 {contact.email}
      </button>
    </section>
  );
}
```

```css
.chat,
.contact-list {
  float: left;
  margin-bottom: 20px;
}
ul,
li {
  list-style: none;
  margin: 0;
  padding: 0;
}
li button {
  width: 100px;
  padding: 10px;
  margin-right: 10px;
}
textarea {
  height: 150px;
}
```

</Sandpack>

不过，虽然在大多数情况下都没关系，但一个稍微更准确的实现是这样的：

```js
function dispatch(action) {
  setState((s) => reducer(s, action));
}
```

这是因为分发的 action 会被排队，直到下一次渲染，[类似于 updater 函数。](/learn/queueing-a-series-of-state-updates)

</Solution>

</Challenges>
