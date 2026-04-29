---
title: 使用 Reducer 和 Context 扩展规模
---

<Intro>

Reducer 可以让你把组件的状态更新逻辑整合起来。Context 则允许你把信息深层传递给其他组件。你可以将 reducer 和 context 结合起来，管理复杂界面的状态。

</Intro>

<YouWillLearn>

* 如何将 reducer 与 context 结合
* 如何避免通过 props 传递 state 和 dispatch
* 如何将 context 和状态逻辑放在单独的文件中

</YouWillLearn>

## 将 reducer 与 context 结合 {/*combining-a-reducer-with-context*/}

在这个来自 [reducer 介绍](/learn/extracting-state-logic-into-a-reducer) 的示例中，状态由 reducer 管理。reducer 函数包含了所有的状态更新逻辑，并声明在这个文件的底部：

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

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
      <h1>京都的一天假期</h1>
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
      throw Error('未知的 action：' + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: '哲学之道', done: true },
  { id: 1, text: '参观寺庙', done: false },
  { id: 2, text: '喝抹茶', done: false }
];
```

```js src/AddTask.js
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

```js src/TaskList.js
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

reducer 有助于让事件处理函数保持简短而精炼。不过，随着应用增长，你可能会遇到另一个困难。**目前，`tasks` 状态和 `dispatch` 函数只在顶层的 `TaskApp` 组件中可用。** 要让其他组件读取任务列表或修改它，你必须显式地把当前状态以及修改它的事件处理函数作为 props [向下传递](/learn/passing-props-to-a-component)。

例如，`TaskApp` 把任务列表和事件处理函数传给 `TaskList`：

```js
<TaskList
  tasks={tasks}
  onChangeTask={handleChangeTask}
  onDeleteTask={handleDeleteTask}
/>
```

然后 `TaskList` 再把事件处理函数传给 `Task`：

```js
<Task
  task={task}
  onChange={onChangeTask}
  onDelete={onDeleteTask}
/>
```

在这样的小例子中，这种方式效果很好，但如果中间有几十个甚至几百个组件，把所有状态和函数一路向下传递会非常令人沮丧！

这就是为什么，作为通过 props 传递它们的替代方案，你可能希望把 `tasks` 状态和 `dispatch` 函数都 [放入 context。](/learn/passing-data-deeply-with-context) **这样，树中位于 `TaskApp` 下方的任何组件都可以读取任务并分发 actions，而无需重复的“属性钻取”。**

下面是将 reducer 和 context 结合的方法：

1. **创建** context。
2. **把** state 和 dispatch 放入 context。
3. **在**树中的任何位置使用 context。

### 第 1 步：创建 context {/*step-1-create-the-context*/}

`useReducer` Hook 会返回当前的 `tasks` 和用于更新它们的 `dispatch` 函数：

```js
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

为了把它们传递到组件树中，你需要[创建](/learn/passing-data-deeply-with-context#step-2-use-the-context)两个独立的 context：

- `TasksContext` 提供当前的任务列表。
- `TasksDispatchContext` 提供让组件分发 actions 的函数。

把它们从单独的文件中导出，这样之后就可以在其他文件中导入它们：

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

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
      <h1>京都的一天假期</h1>
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
      throw Error('未知的 action：' + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: '哲学之道', done: true },
  { id: 1, text: '参观寺庙', done: false },
  { id: 2, text: '喝抹茶', done: false }
];
```

```js src/TasksContext.js active
import { createContext } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```

```js src/AddTask.js
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

```js src/TaskList.js
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

这里，你把 `null` 作为两个 context 的默认值传入。实际值将由 `TaskApp` 组件提供。

### 第 2 步：把 state 和 dispatch 放入 context {/*step-2-put-state-and-dispatch-into-context*/}

现在你可以在 `TaskApp` 组件中导入这两个 context。拿到 `useReducer()` 返回的 `tasks` 和 `dispatch`，并把它们 [提供](/learn/passing-data-deeply-with-context#step-3-provide-the-context) 给其下方的整个树：

```js {4,7-8}
import { TasksContext, TasksDispatchContext } from './TasksContext.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
  // ...
  return (
    <TasksContext value={tasks}>
      <TasksDispatchContext value={dispatch}>
        ...
      </TasksDispatchContext>
    </TasksContext>
  );
}
```

目前，你既通过 props 也通过 context 传递信息：

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import { TasksContext, TasksDispatchContext } from './TasksContext.js';

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
    <TasksContext value={tasks}>
      <TasksDispatchContext value={dispatch}>
        <h1>京都的一天假期</h1>
        <AddTask
          onAddTask={handleAddTask}
        />
        <TaskList
          tasks={tasks}
          onChangeTask={handleChangeTask}
          onDeleteTask={handleDeleteTask}
        />
      </TasksDispatchContext>
    </TasksContext>
  );
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
      throw Error('未知的 action：' + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: '哲学之道', done: true },
  { id: 1, text: '参观寺庙', done: false },
  { id: 2, text: '喝抹茶', done: false }
];
```

```js src/TasksContext.js
import { createContext } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```

```js src/AddTask.js
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

```js src/TaskList.js
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

下一步，你将移除 props 传递。

### 第 3 步：在树中的任何位置使用 context {/*step-3-use-context-anywhere-in-the-tree*/}

现在你不需要再把任务列表或事件处理函数一路向下传递了：

```js {4-5}
<TasksContext value={tasks}>
  <TasksDispatchContext value={dispatch}>
    <h1>京都的一天假期</h1>
    <AddTask />
    <TaskList />
  </TasksDispatchContext>
</TasksContext>
```

相反，任何需要任务列表的组件都可以从 `TasksContext` 中读取它：

```js {2}
export default function TaskList() {
  const tasks = useContext(TasksContext);
  // ...
```

要更新任务列表，任何组件都可以从 context 中读取 `dispatch` 函数并调用它：

```js {3,9-13}
export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useContext(TasksDispatchContext);
  // ...
  return (
    // ...
    <button onClick={() => {
      setText('');
      dispatch({
        type: 'added',
        id: nextId++,
        text: text,
      });
    }}>添加</button>
    // ...
```

**`TaskApp` 组件不再向下传递任何事件处理函数，`TaskList` 也不再向 `Task` 组件传递任何事件处理函数。** 每个组件都会读取它所需要的 context：

<Sandpack>

```js src/App.js
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import { TasksContext, TasksDispatchContext } from './TasksContext.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  return (
    <TasksContext value={tasks}>
      <TasksDispatchContext value={dispatch}>
        <h1>京都的一天假期</h1>
        <AddTask />
        <TaskList />
      </TasksDispatchContext>
    </TasksContext>
  );
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
      throw Error('未知的 action：' + action.type);
    }
  }
}

const initialTasks = [
  { id: 0, text: '哲学之道', done: true },
  { id: 1, text: '参观寺庙', done: false },
  { id: 2, text: '喝抹茶', done: false }
];
```

```js src/TasksContext.js
import { createContext } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```

```js src/AddTask.js
import { useState, useContext } from 'react';
import { TasksDispatchContext } from './TasksContext.js';

export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useContext(TasksDispatchContext);
  return (
    <>
      <input
        placeholder="添加任务"
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

```js src/TaskList.js active
import { useState, useContext } from 'react';
import { TasksContext, TasksDispatchContext } from './TasksContext.js';

export default function TaskList() {
  const tasks = useContext(TasksContext);
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
  const dispatch = useContext(TasksDispatchContext);
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

**状态仍然“存在于”顶层的 `TaskApp` 组件中，并由 `useReducer` 管理。** 但它的 `tasks` 和 `dispatch` 现在可以通过导入并使用这些 context，被树中下方的每个组件访问。

## 将所有连接逻辑移动到单个文件中 {/*moving-all-wiring-into-a-single-file*/}

你不必这样做，但你可以通过把 reducer 和 context 都移动到一个文件里，进一步让组件更简洁。目前，`TasksContext.js` 里只有两个 context 声明：

```js
import { createContext } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```

这个文件马上就要变得拥挤了！你会把 reducer 也移到同一个文件中。然后你会在同一个文件里声明一个新的 `TasksProvider` 组件。这个组件会把所有部分串联起来：

1. 它会使用 reducer 管理状态。
2. 它会把这两个 context 提供给下方的组件。
3. 它会将 [children 作为 prop 传入](/learn/passing-props-to-a-component#passing-jsx-as-children)，这样你就可以向它传递 JSX。

```js
export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksContext value={tasks}>
      <TasksDispatchContext value={dispatch}>
        {children}
      </TasksDispatchContext>
    </TasksContext>
  );
}
```

**这会从你的 `TaskApp` 组件中移除所有复杂性和连接逻辑：**

<Sandpack>

```js src/App.js
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import { TasksProvider } from './TasksContext.js';

export default function TaskApp() {
  return (
    <TasksProvider>
      <h1>Day off in Kyoto</h1>
      <AddTask />
      <TaskList />
    </TasksProvider>
  );
}
```

```js src/TasksContext.js
import { createContext, useReducer } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);

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
      throw Error('未知的操作：' + action.type);
    }
  }
}

const initialTasks = [
  { id: 0, text: '哲学家之路', done: true },
  { id: 1, text: '参观寺庙', done: false },
  { id: 2, text: '喝抹茶', done: false }
];
```

```js src/AddTask.js
import { useState, useContext } from 'react';
import { TasksDispatchContext } from './TasksContext.js';

export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useContext(TasksDispatchContext);
  return (
    <>
      <input
        placeholder="添加任务"
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
import { TasksContext, TasksDispatchContext } from './TasksContext.js';

export default function TaskList() {
  const tasks = useContext(TasksContext);
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
  const dispatch = useContext(TasksDispatchContext);
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

你也可以导出从 `TasksContext.js` 中 _使用_ context 的函数：

```js
export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}
```

当某个组件需要读取 context 时，它可以通过这些函数来完成：

```js
const tasks = useTasks();
const dispatch = useTasksDispatch();
```

这不会改变任何行为，但它让你以后可以进一步拆分这些 context，或者在这些函数中添加一些逻辑。**现在所有的 context 和 reducer 连接逻辑都在 `TasksContext.js` 中。这让组件保持干净、简洁，并专注于它们展示什么，而不是它们从哪里获取数据：**

<Sandpack>

```js src/App.js
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import { TasksProvider } from './TasksContext.js';

export default function TaskApp() {
  return (
    <TasksProvider>
      <h1>Day off in Kyoto</h1>
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
      throw Error('未知的操作：' + action.type);
    }
  }
}

const initialTasks = [
  { id: 0, text: '哲学家之路', done: true },
  { id: 1, text: '参观寺庙', done: false },
  { id: 2, text: '喝抹茶', done: false }
];
```

```js src/AddTask.js
import { useState } from 'react';
import { useTasksDispatch } from './TasksContext.js';

export default function AddTask() {
  const [text, setText] = useState('');
  const dispatch = useTasksDispatch();
  return (
    <>
      <input
        placeholder="添加任务"
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

```js src/TaskList.js active
import { useState } from 'react';
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

你可以把 `TasksProvider` 看作是屏幕中负责处理任务的部分，把 `useTasks` 看作读取任务的方式，把 `useTasksDispatch` 看作从树中任何下方组件更新任务的方式。

<Note>

像 `useTasks` 和 `useTasksDispatch` 这样的函数被称为 *[自定义 Hook。](/learn/reusing-logic-with-custom-hooks)* 如果你的函数名以 `use` 开头，它就会被认为是自定义 Hook。这使你可以在其中使用其他 Hook，例如 `useContext`。

</Note>

随着应用不断增长，你可能会有很多像这样的 context-reducer 对。这是一个强大的方式，可以在几乎不费力的情况下扩展你的应用，并在你想要访问树深处的数据时 [提升状态](/learn/sharing-state-between-components)。

<Recap>

- 你可以将 reducer 与 context 结合起来，让任何组件都能读取和更新它上方的状态。
- 要将状态和 dispatch 函数提供给下方组件：
  1. 创建两个 context（分别用于状态和 dispatch 函数）。
  2. 从使用 reducer 的组件中提供这两个 context。
  3. 在需要读取它们的组件中使用任一 context。
- 你可以通过把所有连接逻辑移动到一个文件中，进一步让组件更简洁。
  - 你可以导出像 `TasksProvider` 这样的组件来提供 context。
  - 你也可以导出像 `useTasks` 和 `useTasksDispatch` 这样的自定义 Hook 来读取它。
- 你的应用中可以有很多像这样的 context-reducer 对。

</Recap>
