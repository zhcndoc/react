---
title: 管理状态
---

<Intro>

随着应用的增长，你会更需要有意识地组织状态，以及让数据如何在组件之间流动。冗余或重复的状态是常见的 bug 来源。在这一章中，你将学习如何良好地组织状态，如何让状态更新逻辑更易维护，以及如何在相距较远的组件之间共享状态。

</Intro>

<YouWillLearn isChapter={true}>

* [如何将 UI 变化视为状态变化](/learn/reacting-to-input-with-state)
* [如何良好地组织状态](/learn/choosing-the-state-structure)
* [如何通过“状态提升”在组件之间共享状态](/learn/sharing-state-between-components)
* [如何控制状态是被保留还是被重置](/learn/preserving-and-resetting-state)
* [如何将复杂的状态逻辑整合到一个函数中](/learn/extracting-state-logic-into-a-reducer)
* [如何在不进行“属性传递”的情况下传递信息](/learn/passing-data-deeply-with-context)
* [当应用增长时如何扩展状态管理](/learn/scaling-up-with-reducer-and-context)

</YouWillLearn>

## 用状态响应输入 {/*reacting-to-input-with-state*/}

在 React 中，你不会直接通过代码修改 UI。例如，你不会写出诸如“禁用按钮”“启用按钮”“显示成功消息”之类的命令。相反，你会描述在组件的不同视觉状态下你想看到的 UI（“初始状态”“输入状态”“成功状态”），然后根据用户输入触发状态变化。这类似于设计师思考 UI 的方式。

下面是一个使用 React 构建的测验表单。注意它如何使用 `status` 状态变量来决定是否启用或禁用提交按钮，以及是否改为显示成功消息。

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [answer, setAnswer] = useState('');
  const [error, setError] = useState(null);
  const [status, setStatus] = useState('typing');

  if (status === 'success') {
    return <h1>答对了！</h1>
  }

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('submitting');
    try {
      await submitForm(answer);
      setStatus('success');
    } catch (err) {
      setStatus('typing');
      setError(err);
    }
  }

  function handleTextareaChange(e) {
    setAnswer(e.target.value);
  }

  return (
    <>
      <h2>城市测验</h2>
      <p>
        在哪个城市里有一个广告牌能把空气变成可饮用的水？
      </p>
      <form onSubmit={handleSubmit}>
        <textarea
          value={answer}
          onChange={handleTextareaChange}
          disabled={status === 'submitting'}
        />
        <br />
        <button disabled={
          answer.length === 0 ||
          status === 'submitting'
        }>
          提交
        </button>
        {error !== null &&
          <p className="Error">
            {error.message}
          </p>
        }
      </form>
    </>
  );
}

function submitForm(answer) {
  // 假装它正在访问网络。
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      let shouldError = answer.toLowerCase() !== 'lima'
      if (shouldError) {
        reject(new Error('猜得不错，但答案错误。再试一次！'));
      } else {
        resolve();
      }
    }, 1500);
  });
}
```

```css
.Error { color: red; }
```

</Sandpack>

<LearnMore path="/learn/reacting-to-input-with-state">

阅读 **[用状态响应输入](/learn/reacting-to-input-with-state)**，学习如何以状态驱动的思维方式来处理交互。

</LearnMore>

## 选择状态结构 {/*choosing-the-state-structure*/}

良好地组织状态，可以决定一个组件是易于修改和调试，还是持续成为 bug 来源。最重要的原则是：状态不应包含冗余或重复的信息。如果有不必要的状态，就很容易忘记更新它，从而引入 bug！

例如，这个表单有一个**冗余的** `fullName` 状态变量：

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [fullName, setFullName] = useState('');

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
    setFullName(e.target.value + ' ' + lastName);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
    setFullName(firstName + ' ' + e.target.value);
  }

  return (
    <>
      <h2>让我们为你办理登记</h2>
      <label>
        名字：{' '}
        <input
          value={firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        姓氏：{' '}
        <input
          value={lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <p>
        你的票将签发给：<b>{fullName}</b>
      </p>
    </>
  );
}
```

```css
label { display: block; margin-bottom: 5px; }
```

</Sandpack>

你可以删除它，并在组件渲染时计算 `fullName`，从而简化代码：

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  const fullName = firstName + ' ' + lastName;

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
  }

  return (
    <>
      <h2>让我们为你办理登记</h2>
      <label>
        名字：{' '}
        <input
          value={firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        姓氏：{' '}
        <input
          value={lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <p>
        你的票将签发给：<b>{fullName}</b>
      </p>
    </>
  );
}
```

```css
label { display: block; margin-bottom: 5px; }
```

</Sandpack>

这看起来可能只是一个小改动，但 React 应用中很多 bug 都是这样修复的。

<LearnMore path="/learn/choosing-the-state-structure">

阅读 **[选择状态结构](/learn/choosing-the-state-structure)**，学习如何设计状态形状以避免 bug。

</LearnMore>

## 在组件之间共享状态 {/*sharing-state-between-components*/}

有时你希望两个组件的状态始终一起变化。要做到这一点，就把它们各自的状态删除，将状态提升到它们最近的共同父组件中，然后通过 props 传递给它们。这被称为“状态提升”，也是你编写 React 代码时最常做的事情之一。

在这个例子中，一次只能有一个面板处于活动状态。为实现这一点，不再把活动状态保存在每个单独的面板内部，而是由父组件持有该状态，并为其子组件指定 props。

<Sandpack>

```js
import { useState } from 'react';

export default function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);
  return (
    <>
      <h2>哈萨克斯坦，阿拉木图</h2>
      <Panel
        title="关于"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        阿拉木图人口约 200 万，是哈萨克斯坦最大的城市。从 1929 年到 1997 年，它曾是该国首都。
      </Panel>
      <Panel
        title="词源"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        这个名字来自 <span lang="kk-KZ">алма</span>，即哈萨克语中“苹果”的意思，通常被翻译为“满是苹果”。事实上，阿拉木图周边地区被认为是苹果的原生家园，而野生的 <i lang="la">Malus sieversii</i> 被认为很可能是现代家养苹果的祖先。
      </Panel>
    </>
  );
}

function Panel({
  title,
  children,
  isActive,
  onShow
}) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}>
          显示
        </button>
      )}
    </section>
  );
}
```

```css
h3, p { margin: 5px 0px; }
.panel {
  padding: 10px;
  border: 1px solid #aaa;
}
```

</Sandpack>

<LearnMore path="/learn/sharing-state-between-components">

阅读 **[在组件之间共享状态](/learn/sharing-state-between-components)**，学习如何提升状态以及让组件保持同步。

</LearnMore>

## 保留和重置状态 {/*preserving-and-resetting-state*/}

当你重新渲染一个组件时，React 需要决定保留（并更新）树中的哪些部分，以及丢弃或从头重新创建哪些部分。在大多数情况下，React 的自动行为已经足够好。默认情况下，React 会保留与之前渲染的组件树“匹配”的那部分树。

然而，有时这并不是你想要的。在这个聊天应用中，输入一条消息后再切换收件人，并不会重置输入框。这可能会导致用户不小心把消息发送给错误的人：

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat contact={to} />
    </div>
  )
}

const contacts = [
  { name: 'Taylor', email: 'taylor@mail.com' },
  { name: 'Alice', email: 'alice@mail.com' },
  { name: 'Bob', email: 'bob@mail.com' }
];
```

```js src/ContactList.js
export default function ContactList({
  selectedContact,
  contacts,
  onSelect
}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map(contact =>
          <li key={contact.email}>
            <button onClick={() => {
              onSelect(contact);
            }}>
              {contact.name}
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js src/Chat.js
import { useState } from 'react';

export default function Chat({ contact }) {
  const [text, setText] = useState('');
  return (
    <section className="chat">
      <textarea
        value={text}
        placeholder={'与 ' + contact.name + ' 聊天'}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button>发送给 {contact.email}</button>
    </section>
  );
}
```

```css
.chat, .contact-list {
  float: left;
  margin-bottom: 20px;
}
ul, li {
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

React 允许你覆盖默认行为，并通过传递不同的 `key` 强制组件重置其状态，就像 `<Chat key={email} />` 一样。这告诉 React：如果收件人不同，就应将其视为一个**不同的** `Chat` 组件，需要用新数据（以及输入框等 UI）从头重新创建。现在，在收件人之间切换时会重置输入框——即使你渲染的是同一个组件。

<Sandpack>

```js src/App.js
import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat key={to.email} contact={to} />
    </div>
  )
}

const contacts = [
  { name: 'Taylor', email: 'taylor@mail.com' },
  { name: 'Alice', email: 'alice@mail.com' },
  { name: 'Bob', email: 'bob@mail.com' }
];
```

```js src/ContactList.js
export default function ContactList({
  selectedContact,
  contacts,
  onSelect
}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map(contact =>
          <li key={contact.email}>
            <button onClick={() => {
              onSelect(contact);
            }}>
              {contact.name}
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

```js src/Chat.js
import { useState } from 'react';

export default function Chat({ contact }) {
  const [text, setText] = useState('');
  return (
    <section className="chat">
      <textarea
        value={text}
        placeholder={'与 ' + contact.name + ' 聊天'}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button>发送给 {contact.email}</button>
    </section>
  );
}
```

```css
.chat, .contact-list {
  float: left;
  margin-bottom: 20px;
}
ul, li {
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

<LearnMore path="/learn/preserving-and-resetting-state">

阅读 **[保留和重置状态](/learn/preserving-and-resetting-state)**，学习状态的生命周期以及如何控制它。

</LearnMore>

## 将状态逻辑提取到 reducer 中 {/*extracting-state-logic-into-a-reducer*/}

在许多事件处理函数中分散着大量状态更新的组件，可能会变得难以管理。对于这种情况，你可以把所有状态更新逻辑集中到组件外面的一个单独函数中，这个函数称为 “reducer”。这样你的事件处理函数就会变得简洁，因为它们只需要说明用户的 “动作”。在文件底部，reducer 函数会指定状态应该如何响应每个动作进行更新！

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
      throw Error('未知的动作：' + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: '参观卡夫卡博物馆', done: true },
  { id: 1, text: '看木偶戏', done: false },
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

<LearnMore path="/learn/extracting-state-logic-into-a-reducer">

阅读 **[将状态逻辑提取到 reducer 中](/learn/extracting-state-logic-into-a-reducer)**，了解如何在 reducer 函数中整合逻辑。

</LearnMore>

## 使用 context 深层传递数据 {/*passing-data-deeply-with-context*/}

通常，你会通过 props 将信息从父组件传递给子组件。但如果你需要把某个 prop 经过许多组件层层传递下去，或者许多组件都需要同样的信息，传递 props 可能会变得不方便。Context 允许父组件把某些信息提供给它下面树中的任意组件——不管层级有多深——而无需显式地通过 props 传递。

这里，`Heading` 组件通过“询问”最近的 `Section` 来确定自己的标题级别。每个 `Section` 通过询问父 `Section` 并在其基础上加一来跟踪自己的级别。每个 `Section` 都会通过 context 向其下方的所有组件提供信息，而不需要传递 props。

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
          <Heading>副标题</Heading>
          <Heading>副标题</Heading>
          <Heading>副标题</Heading>
          <Section>
            <Heading>二级副标题</Heading>
            <Heading>二级副标题</Heading>
            <Heading>二级副标题</Heading>
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
      throw Error('标题必须位于 Section 内部！');
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
      throw Error('未知级别：' + level);
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

<LearnMore path="/learn/passing-data-deeply-with-context">

阅读 **[使用 Context 深层传递数据](/learn/passing-data-deeply-with-context)**，了解如何使用 context 作为传递 props 的替代方案。

</LearnMore>

## 使用 reducer 和 context 扩展规模 {/*scaling-up-with-reducer-and-context*/}

Reducer 可以让你将组件的状态更新逻辑集中起来。Context 可以让你将信息向下传递到更深层的其他组件。你可以将 reducer 和 context 结合起来管理复杂界面的状态。

使用这种方法时，具有复杂状态的父组件会通过 reducer 来管理它。树中任意深处的其他组件都可以通过 context 读取它的状态。它们也可以分发动作来更新该状态。

<Sandpack>

```js src/App.js
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import { TasksProvider } from './TasksContext.js';

export default function TaskApp() {
  return (
    <TasksProvider>
      <h1>京都休息日</h1>
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
      throw Error('未知的动作：' + action.type);
    }
  }
}

const initialTasks = [
  { id: 0, text: '哲人之路', done: true },
  { id: 1, text: '参观寺庙', done: false },
  { id: 2, text: '喝抹茶', done: false }
];
```

```js src/AddTask.js
import { useState, useContext } from 'react';
import { useTasksDispatch } from './TasksContext.js';

export default function AddTask({ onAddTask }) {
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

<LearnMore path="/learn/scaling-up-with-reducer-and-context">

阅读 **[使用 reducer 和 context 扩展规模](/learn/scaling-up-with-reducer-and-context)**，了解状态管理如何在不断增长的应用中扩展。

</LearnMore>

## 接下来是什么？ {/*whats-next*/}

前往 [使用 State 响应输入](/learn/reacting-to-input-with-state) 开始逐页阅读本章！

或者，如果你已经熟悉这些主题，为什么不去看看 [逃生舱口](/learn/escape-hatches) 呢？
