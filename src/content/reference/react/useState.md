---
title: useState
---

<Intro>

`useState` 是一个 React Hook，可让你向组件添加一个 [state 变量](/learn/state-a-components-memory)。

```js
const [state, setState] = useState(initialState)
```

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `useState(initialState)` {/*usestate*/}

在组件顶层调用 `useState` 来声明一个 [state 变量。](/learn/state-a-components-memory)

```js
import { useState } from 'react';

function MyComponent() {
  const [age, setAge] = useState(28);
  const [name, setName] = useState('Taylor');
  const [todos, setTodos] = useState(() => createTodos());
  // ...
```

按照约定，state 变量通常命名为 `[something, setSomething]`，并使用 [数组解构。](https://javascript.info/destructuring-assignment)

[查看更多示例。](#usage)

#### 参数 {/*parameters*/}

* `initialState`：你希望 state 的初始值。它可以是任何类型的值，但对于函数有特殊行为。这个参数在初次渲染后会被忽略。
  * 如果你把一个函数作为 `initialState` 传入，它会被当作一个 _initializer function_。它应该是纯函数，不应接受任何参数，并且应返回任意类型的值。React 会在初始化组件时调用你的 initializer function，并将其返回值作为初始 state 存储起来。[查看下面的示例。](#avoiding-recreating-the-initial-state)

#### 返回值 {/*returns*/}

`useState` 返回一个恰好包含两个值的数组：

1. 当前的 state。在第一次渲染时，它将与你传入的 `initialState` 相匹配。
2. 允许你将 state 更新为其他值并触发重新渲染的 [`set` 函数](#setstate)。

#### 注意事项 {/*caveats*/}

* `useState` 是一个 Hook，因此你只能在**组件顶层**或你自己的 Hooks 中调用它。你不能在循环或条件中调用它。如果你需要这样做，请提取一个新组件并把 state 移到其中。
* 在 Strict Mode 下，React 会**调用你的 initializer function 两次**，以[帮助你发现意外的副作用。](#my-initializer-or-updater-function-runs-twice) 这是仅开发环境下的行为，不会影响生产环境。如果你的 initializer function 是纯函数（它本来就应该是），这不会影响行为。其中一次调用的结果会被忽略。

---

### `set` 函数，例如 `setSomething(nextState)` {/*setstate*/}

`useState` 返回的 `set` 函数允许你将 state 更新为不同的值并触发重新渲染。你可以直接传入下一个 state，或者传入一个根据前一个 state 计算出它的函数：

```js
const [name, setName] = useState('Edward');

function handleClick() {
  setName('Taylor');
  setAge(a => a + 1);
  // ...
```

#### 参数 {/*setstate-parameters*/}

* `nextState`：你希望 state 变成的值。它可以是任何类型的值，但对于函数有特殊行为。
  * 如果你把一个函数作为 `nextState` 传入，它会被当作一个 _updater function_。它必须是纯函数，只应接收待处理的 state 作为唯一参数，并应返回下一个 state。React 会把你的 updater function 放入队列并重新渲染你的组件。在下一次渲染时，React 会通过把队列中的所有 updater 依次应用到前一个 state 上来计算下一个 state。[查看下面的示例。](#updating-state-based-on-the-previous-state)

#### 返回值 {/*setstate-returns*/}

`set` 函数没有返回值。

#### 注意事项 {/*setstate-caveats*/}

* `set` 函数**只会更新下一次渲染的 state 变量**。如果你在调用 `set` 函数之后读取 state 变量，[你仍然会得到旧值](#ive-updated-the-state-but-logging-gives-me-the-old-value)，也就是你调用之前屏幕上显示的值。

* 如果你提供的新值与当前 `state` 完全相同（根据 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较判断），React 会**跳过重新渲染该组件及其子组件。** 这是一个优化。虽然在某些情况下，React 可能仍然需要在跳过子组件之前调用你的组件，但这不应该影响你的代码。

* React 会[批量处理 state 更新。](/learn/queueing-a-series-of-state-updates) 它会在**所有事件处理函数运行完并调用了它们的 `set` 函数之后**再更新屏幕。这可以防止在单个事件中发生多次重新渲染。在极少数情况下，如果你需要强制 React 更早地更新屏幕，例如为了访问 DOM，你可以使用 [`flushSync`。](/reference/react-dom/flushSync)

* `set` 函数具有稳定的身份，因此你经常会看到它被省略在 Effect 依赖项之外，但即使包含它也不会导致 Effect 触发。如果 linter 允许你在不报错的情况下省略某个依赖项，那么这样做是安全的。[了解更多关于移除 Effect 依赖项的内容。](/learn/removing-effect-dependencies#move-dynamic-objects-and-functions-inside-your-effect)

* 在**渲染期间**调用 `set` 函数只允许在当前正在渲染的组件内部进行。React 会丢弃其输出，并立即尝试使用新 state 重新渲染它。这种模式很少需要，但你可以用它来**存储上一次渲染中的信息**。[查看下面的示例。](#storing-information-from-previous-renders)

* 在 Strict Mode 下，React 会**调用你的 updater function 两次**，以[帮助你发现意外的副作用。](#my-initializer-or-updater-function-runs-twice) 这是仅开发环境下的行为，不会影响生产环境。如果你的 updater function 是纯函数（它本来就应该是），这不会影响行为。其中一次调用的结果会被忽略。

---

## 使用 {/*usage*/}

### 向组件添加 state {/*adding-state-to-a-component*/}

在组件顶层调用 `useState`，以声明一个或多个 [state 变量。](/learn/state-a-components-memory)

```js [[1, 4, "age"], [2, 4, "setAge"], [3, 4, "42"], [1, 5, "name"], [2, 5, "setName"], [3, 5, "'Taylor'"]]
import { useState } from 'react';

function MyComponent() {
  const [age, setAge] = useState(42);
  const [name, setName] = useState('Taylor');
  // ...
```

按照约定，state 变量通常命名为 `[something, setSomething]`，并使用 [数组解构。](https://javascript.info/destructuring-assignment)

`useState` 返回一个恰好包含两个项的数组：

1. 这个 state 变量的 <CodeStep step={1}>当前 state</CodeStep>，初始值为你提供的 <CodeStep step={3}>初始 state</CodeStep>。
2. 允许你根据交互将其更改为其他值的 <CodeStep step={2}>`set` 函数</CodeStep>。

要更新屏幕上的内容，请用某个下一个 state 调用 `set` 函数：

```js [[2, 2, "setName"]]
function handleClick() {
  setName('Robin');
}
```

React 会保存下一个 state，再次使用新值渲染你的组件，并更新 UI。

<Pitfall>

调用 `set` 函数[**不会**改变已在执行中的代码里的当前 state](#ive-updated-the-state-but-logging-gives-me-the-old-value)：

```js {3}
function handleClick() {
  setName('Robin');
  console.log(name); // 仍然是 "Taylor"！
}
```

它只会影响从*下一次*渲染开始 `useState` 返回的内容。

</Pitfall>

<Recipes titleText="useState 基础示例" titleId="examples-basic">

#### 计数器（数字） {/*counter-number*/}

在这个示例中，`count` state 变量保存一个数字。点击按钮会使其递增。

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      You pressed me {count} times
    </button>
  );
}
```

</Sandpack>

<Solution />

#### 文本输入框（字符串） {/*text-field-string*/}

在这个示例中，`text` state 变量保存一个字符串。输入时，`handleChange` 会从浏览器输入 DOM 元素中读取最新的输入值，并调用 `setText` 来更新 state。这样你就可以在下方显示当前的 `text`。

<Sandpack>

```js
import { useState } from 'react';

export default function MyInput() {
  const [text, setText] = useState('hello');

  function handleChange(e) {
    setText(e.target.value);
  }

  return (
    <>
      <input value={text} onChange={handleChange} />
      <p>You typed: {text}</p>
      <button onClick={() => setText('hello')}>
        Reset
      </button>
    </>
  );
}
```

</Sandpack>

<Solution />

#### 复选框（布尔值） {/*checkbox-boolean*/}

在这个示例中，`liked` state 变量保存一个布尔值。当你点击输入框时，`setLiked` 会根据浏览器复选框输入是否被勾选来更新 `liked` state 变量。`liked` 变量用于渲染复选框下方的文本。

<Sandpack>

```js
import { useState } from 'react';

export default function MyCheckbox() {
  const [liked, setLiked] = useState(true);

  function handleChange(e) {
    setLiked(e.target.checked);
  }

  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={liked}
          onChange={handleChange}
        />
        I liked this
      </label>
      <p>You {liked ? 'liked' : 'did not like'} this.</p>
    </>
  );
}
```

</Sandpack>

<Solution />

#### 表单（两个变量） {/*form-two-variables*/}

你可以在同一个组件中声明多个 state 变量。每个 state 变量都是完全独立的。

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [name, setName] = useState('Taylor');
  const [age, setAge] = useState(42);

  return (
    <>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <button onClick={() => setAge(age + 1)}>
        Increment age
      </button>
      <p>Hello, {name}. You are {age}.</p>
    </>
  );
}
```

```css
button { display: block; margin-top: 10px; }
```

</Sandpack>

<Solution />

</Recipes>

---

### 基于前一个 state 更新 state {/*updating-state-based-on-the-previous-state*/}

假设 `age` 为 `42`。这个处理函数会三次调用 `setAge(age + 1)`：

```js
function handleClick() {
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
}
```

然而，点击一次之后，`age` 只会变成 `43`，而不是 `45`！这是因为调用 `set` 函数[不会更新](/learn/state-as-a-snapshot)已在运行中的代码里的 `age` state 变量。所以每次 `setAge(age + 1)` 调用都会变成 `setAge(43)`。

要解决这个问题，你可以向 `setAge` 传入一个 *updater function*，而不是下一个 state：

```js [[1, 2, "a", 0], [2, 2, "a + 1"], [1, 3, "a", 0], [2, 3, "a + 1"], [1, 4, "a", 0], [2, 4, "a + 1"]]
function handleClick() {
  setAge(a => a + 1); // setAge(42 => 43)
  setAge(a => a + 1); // setAge(43 => 44)
  setAge(a => a + 1); // setAge(44 => 45)
}
```

这里，`a => a + 1` 就是你的 updater function。它接收 <CodeStep step={1}>待处理的 state</CodeStep>，并基于它计算出 <CodeStep step={2}>下一个 state</CodeStep>。

React 会把你的 updater functions 放入[队列。](/learn/queueing-a-series-of-state-updates) 然后在下一次渲染时按相同顺序调用它们：

1. `a => a + 1` 会接收 `42` 作为待处理的 state，并返回 `43` 作为下一个 state。
1. `a => a + 1` 会接收 `43` 作为待处理的 state，并返回 `44` 作为下一个 state。
1. `a => a + 1` 会接收 `44` 作为待处理的 state，并返回 `45` 作为下一个 state。

队列中没有其他更新，因此 React 最终会将 `45` 保存为当前 state。

按照约定，通常会用 state 变量名称的第一个字母来命名待处理的 state 参数，例如用 `a` 表示 `age`。不过，你也可以把它命名为 `prevAge` 或其他你觉得更清晰的名字。

React 可能会在开发环境中[调用你的 updaters 两次](#my-initializer-or-updater-function-runs-twice)，以验证它们是否[纯函数。](/learn/keeping-components-pure)

<DeepDive>

#### 总是优先使用 updater 吗？ {/*is-using-an-updater-always-preferred*/}

你可能会听到这样的建议：如果你设置的 state 是从前一个 state 计算出来的，就总是写成 `setAge(a => a + 1)`。这样写没有坏处，但也并不总是必要。

在大多数情况下，这两种方式没有区别。React 总是会确保，对于点击这类有意的用户操作，`age` state 变量会在下一次点击之前更新。这意味着，点击处理函数在事件开始时看到“过期”的 `age` 并不会有风险。

不过，如果你在同一个事件中执行多次更新，updater 会很有帮助。如果直接访问 state 变量本身不方便，它们也很有用（在优化重新渲染时你可能会遇到这种情况）。

如果你更偏好一致性而不是稍微更啰嗦的语法，那么在 state 是由前一个 state 计算出来时，总是写 updater 是合理的。如果它是由某个*其他* state 变量的前一个 state 计算出来的，你可能会想把它们合并成一个对象，并[使用 reducer。](/learn/extracting-state-logic-into-a-reducer)

</DeepDive>

<Recipes titleText="传入 updater 和直接传入下一个 state 的区别" titleId="examples-updater">

#### 传入 updater function {/*passing-the-updater-function*/}

这个示例传入了 updater function，因此 “+3” 按钮可以正常工作。

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [age, setAge] = useState(42);

  function increment() {
    setAge(a => a + 1);
  }

  return (
    <>
      <h1>Your age: {age}</h1>
      <button onClick={() => {
        increment();
        increment();
        increment();
      }}>+3</button>
      <button onClick={() => {
        increment();
      }}>+1</button>
    </>
  );
}
```

```css
button { display: block; margin: 10px; font-size: 20px; }
h1 { display: block; margin: 10px; }
```

</Sandpack>

<Solution />

#### 直接传入下一个 state {/*passing-the-next-state-directly*/}

这个示例**没有**传入 updater function，因此 “+3” 按钮**不能按预期工作**。

<Sandpack>

```js
import { useState } from 'react';

export default function Counter() {
  const [age, setAge] = useState(42);

  function increment() {
    setAge(age + 1);
  }

  return (
    <>
      <h1>Your age: {age}</h1>
      <button onClick={() => {
        increment();
        increment();
        increment();
      }}>+3</button>
      <button onClick={() => {
        increment();
      }}>+1</button>
    </>
  );
}
```

```css
button { display: block; margin: 10px; font-size: 20px; }
h1 { display: block; margin: 10px; }
```

</Sandpack>

<Solution />

</Recipes>

---

### 更新 state 中的对象和数组 {/*updating-objects-and-arrays-in-state*/}

你可以把对象和数组放入 state 中。在 React 中，state 被视为只读，因此**你应该*替换*它，而不是*修改*现有对象**。例如，如果你在 state 中有一个 `form` 对象，不要直接修改它：

```js
// 🚩 不要像这样修改 state 中的对象：
form.firstName = 'Taylor';
```

相反，应该通过创建一个新对象来替换整个对象：

```js
// ✅ 用一个新对象替换 state
setForm({
  ...form,
  firstName: 'Taylor'
});
```

阅读 [更新 state 中的对象](/learn/updating-objects-in-state) 和 [更新 state 中的数组](/learn/updating-arrays-in-state) 以了解更多。

<Recipes titleText="state 中对象和数组的示例" titleId="examples-objects">

#### 表单（对象） {/*form-object*/}

在这个示例中，`form` state 变量保存一个对象。每个输入框都有一个变更处理函数，它会用整个表单的下一个 state 调用 `setForm`。`{ ...form }` 扩展语法确保 state 对象被替换而不是被修改。

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [form, setForm] = useState({
    firstName: 'Barbara',
    lastName: 'Hepworth',
    email: 'bhepworth@sculpture.com',
  });

  return (
    <>
      <label>
        名字：
        <input
          value={form.firstName}
          onChange={e => {
            setForm({
              ...form,
              firstName: e.target.value
            });
          }}
        />
      </label>
      <label>
        姓氏：
        <input
          value={form.lastName}
          onChange={e => {
            setForm({
              ...form,
              lastName: e.target.value
            });
          }}
        />
      </label>
      <label>
        邮箱：
        <input
          value={form.email}
          onChange={e => {
            setForm({
              ...form,
              email: e.target.value
            });
          }}
        />
      </label>
      <p>
        {form.firstName}{' '}
        {form.lastName}{' '}
        ({form.email})
      </p>
    </>
  );
}
```

```css
label { display: block; }
input { margin-left: 5px; }
```

</Sandpack>

<Solution />

#### 表单（嵌套对象） {/*form-nested-object*/}

在这个示例中，state 更加嵌套。当你更新嵌套 state 时，你需要创建你正在更新的对象的副本，以及沿着向上路径中“包含”它的任何对象。阅读 [更新嵌套对象](/learn/updating-objects-in-state#updating-a-nested-object) 以了解更多。

<Sandpack>

```js
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    name: 'Niki de Saint Phalle',
    artwork: {
      title: 'Blue Nana',
      city: 'Hamburg',
      image: 'https://react.dev/images/docs/scientists/Sd1AgUOm.jpg',
    }
  });

  function handleNameChange(e) {
    setPerson({
      ...person,
      name: e.target.value
    });
  }

  function handleTitleChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        title: e.target.value
      }
    });
  }

  function handleCityChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        city: e.target.value
      }
    });
  }

  function handleImageChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        image: e.target.value
      }
    });
  }

  return (
    <>
      <label>
        名字：
        <input
          value={person.name}
          onChange={handleNameChange}
        />
      </label>
      <label>
        标题：
        <input
          value={person.artwork.title}
          onChange={handleTitleChange}
        />
      </label>
      <label>
        城市：
        <input
          value={person.artwork.city}
          onChange={handleCityChange}
        />
      </label>
      <label>
        图片：
        <input
          value={person.artwork.image}
          onChange={handleImageChange}
        />
      </label>
      <p>
        <i>{person.artwork.title}</i>
        {' 作者 '}
        {person.name}
        <br />
        （位于 {person.artwork.city}）
      </p>
      <img
        src={person.artwork.image}
        alt={person.artwork.title}
      />
    </>
  );
}
```

```css
label { display: block; }
input { margin-left: 5px; margin-bottom: 5px; }
img { width: 200px; height: 200px; }
```

</Sandpack>

<Solution />

#### 列表（数组） {/*list-array*/}

在这个示例中，`todos` state 变量保存一个数组。每个按钮处理函数都会用该数组的下一个版本调用 `setTodos`。`[...todos]` 扩展语法、`todos.map()` 和 `todos.filter()` 确保 state 数组被替换而不是被修改。

<Sandpack>

```js src/App.js
import { useState } from 'react';
import AddTodo from './AddTodo.js';
import TaskList from './TaskList.js';

let nextId = 3;
const initialTodos = [
  { id: 0, title: 'Buy milk', done: true },
  { id: 1, title: 'Eat tacos', done: false },
  { id: 2, title: 'Brew tea', done: false },
];

export default function TaskApp() {
  const [todos, setTodos] = useState(initialTodos);

  function handleAddTodo(title) {
    setTodos([
      ...todos,
      {
        id: nextId++,
        title: title,
        done: false
      }
    ]);
  }

  function handleChangeTodo(nextTodo) {
    setTodos(todos.map(t => {
      if (t.id === nextTodo.id) {
        return nextTodo;
      } else {
        return t;
      }
    }));
  }

  function handleDeleteTodo(todoId) {
    setTodos(
      todos.filter(t => t.id !== todoId)
    );
  }

  return (
    <>
      <AddTodo
        onAddTodo={handleAddTodo}
      />
      <TaskList
        todos={todos}
        onChangeTodo={handleChangeTodo}
        onDeleteTodo={handleDeleteTodo}
      />
    </>
  );
}
```

```js src/AddTodo.js
import { useState } from 'react';

export default function AddTodo({ onAddTodo }) {
  const [title, setTitle] = useState('');
  return (
    <>
      <input
        placeholder="添加待办事项"
        value={title}
        onChange={e => setTitle(e.target.value)}
      />
      <button onClick={() => {
        setTitle('');
        onAddTodo(title);
      }}>添加</button>
    </>
  )
}
```

```js src/TaskList.js
import { useState } from 'react';

export default function TaskList({
  todos,
  onChangeTodo,
  onDeleteTodo
}) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <Task
            todo={todo}
            onChange={onChangeTodo}
            onDelete={onDeleteTodo}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ todo, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let todoContent;
  if (isEditing) {
    todoContent = (
      <>
        <input
          value={todo.title}
          onChange={e => {
            onChange({
              ...todo,
              title: e.target.value
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          保存
        </button>
      </>
    );
  } else {
    todoContent = (
      <>
        {todo.title}
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
        checked={todo.done}
        onChange={e => {
          onChange({
            ...todo,
            done: e.target.checked
          });
        }}
      />
      {todoContent}
      <button onClick={() => onDelete(todo.id)}>
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

如果不通过修改来更新数组和对象让你觉得繁琐，你可以使用像 [Immer](https://github.com/immerjs/use-immer) 这样的库来减少重复代码。Immer 允许你像修改对象一样编写简洁的代码，但在底层它会执行不可变更新：

<Sandpack>

```js
import { useState } from 'react';
import { useImmer } from 'use-immer';

let nextId = 3;
const initialList = [
  { id: 0, title: 'Big Bellies', seen: false },
  { id: 1, title: 'Lunar Landscape', seen: false },
  { id: 2, title: 'Terracotta Army', seen: true },
];

export default function BucketList() {
  const [list, updateList] = useImmer(initialList);

  function handleToggle(artworkId, nextSeen) {
    updateList(draft => {
      const artwork = draft.find(a =>
        a.id === artworkId
      );
      artwork.seen = nextSeen;
    });
  }

  return (
    <>
      <h1>Art Bucket List</h1>
      <h2>My list of art to see:</h2>
      <ItemList
        artworks={list}
        onToggle={handleToggle} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map(artwork => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={e => {
                onToggle(
                  artwork.id,
                  e.target.checked
                );
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
  );
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

<Solution />

</Recipes>

---

### 避免重新创建初始 state {/*avoiding-recreating-the-initial-state*/}

React 只会保存一次初始 state，并在后续渲染中忽略它。

```js
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos());
  // ...
```

虽然 `createInitialTodos()` 的结果只会用于初次渲染，但你仍然在每次渲染时调用了这个函数。如果它创建的是大型数组或执行了昂贵的计算，这会很浪费。

为了解决这个问题，你可以改为把它作为一个 _initializer_ 函数传给 `useState`：

```js
function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos);
  // ...
```

注意，你传入的是 `createInitialTodos`，也就是*函数本身*，而不是 `createInitialTodos()`，即调用它的结果。如果你向 `useState` 传入一个函数，React 只会在初始化时调用它。

React 可能会在开发环境中[调用你的 initializers 两次](#my-initializer-or-updater-function-runs-twice)，以验证它们是否[纯函数。](/learn/keeping-components-pure)

<Recipes titleText="传入 initializer 与直接传入初始 state 的区别" titleId="examples-initializer">

#### 传入 initializer function {/*passing-the-initializer-function*/}

这个示例传入了 initializer function，因此 `createInitialTodos` 函数只会在初始化期间运行。它不会在组件重新渲染时运行，例如你在输入框中输入时。

<Sandpack>

```js
import { useState } from 'react';

function createInitialTodos() {
  const initialTodos = [];
  for (let i = 0; i < 50; i++) {
    initialTodos.push({
      id: i,
      text: 'Item ' + (i + 1)
    });
  }
  return initialTodos;
}

export default function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos);
  const [text, setText] = useState('');

  return (
    <>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        setTodos([{
          id: todos.length,
          text: text
        }, ...todos]);
      }}>Add</button>
      <ul>
        {todos.map(item => (
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

#### 直接传入初始 state {/*passing-the-initial-state-directly*/}

这个示例**没有**传入 initializer function，因此 `createInitialTodos` 函数会在每次渲染时运行，例如你在输入框中输入时。行为上没有可观察到的差异，但这段代码效率较低。

<Sandpack>

```js
import { useState } from 'react';

function createInitialTodos() {
  const initialTodos = [];
  for (let i = 0; i < 50; i++) {
    initialTodos.push({
      id: i,
      text: 'Item ' + (i + 1)
    });
  }
  return initialTodos;
}

export default function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos());
  const [text, setText] = useState('');

  return (
    <>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        setTodos([{
          id: todos.length,
          text: text
        }, ...todos]);
      }}>Add</button>
      <ul>
        {todos.map(item => (
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

### 使用 key 重置 state {/*resetting-state-with-a-key*/}

你经常会在[渲染列表](/learn/rendering-lists)时遇到 `key` 属性。不过，它还有另一个用途。

你可以通过向组件传递不同的 `key` 来**重置组件的 state。** 在这个示例中，Reset 按钮会更改 `version` state 变量，我们将它作为 `key` 传给 `Form`。当 `key` 改变时，React 会从头重新创建 `Form` 组件（以及它的所有子组件），因此它的 state 会被重置。

阅读 [保留和重置 state](/learn/preserving-and-resetting-state) 以了解更多。

<Sandpack>

```js src/App.js
import { useState } from 'react';

export default function App() {
  const [version, setVersion] = useState(0);

  function handleReset() {
    setVersion(version + 1);
  }

  return (
    <>
      <button onClick={handleReset}>重置</button>
      <Form key={version} />
    </>
  );
}

function Form() {
  const [name, setName] = useState('Taylor');

  return (
    <>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <p>Hello, {name}.</p>
    </>
  );
}
```

```css
button { display: block; margin-bottom: 20px; }
```

</Sandpack>

---

### 存储上一次渲染中的信息 {/*storing-information-from-previous-renders*/}

通常，你会在事件处理函数中更新 state。不过，在极少数情况下，你可能希望根据渲染来调整 state——例如，当 prop 改变时你可能想修改某个 state 变量。

在大多数情况下，你并不需要这样做：

* **如果你需要的值可以完全根据当前 props 或其他 state 计算出来，[就把那个冗余 state 完全移除。](/learn/choosing-the-state-structure#avoid-redundant-state)** 如果你担心重新计算太频繁，[`useMemo` Hook](/reference/react/useMemo) 可以帮忙。
* 如果你想重置整个组件树的 state，[给你的组件传入不同的 `key`。](#resetting-state-with-a-key)
* 如果可以的话，在事件处理函数中更新所有相关的 state。

在极少数这些都不适用的情况下，你可以使用一种模式：在组件渲染时调用 `set` 函数，根据到目前为止已经渲染出来的值来更新 state。

下面是一个示例。这个 `CountLabel` 组件显示传入的 `count` prop：

```js src/CountLabel.js
export default function CountLabel({ count }) {
  return <h1>{count}</h1>
}
```

假设你想显示计数器相较于上一次变化是*增加还是减少*。`count` prop 并不能告诉你这一点——你需要跟踪它的前一个值。添加 `prevCount` state 变量来跟踪它。再添加一个名为 `trend` 的 state 变量来保存计数是增加还是减少。将 `prevCount` 与 `count` 比较，如果它们不相等，就同时更新 `prevCount` 和 `trend`。现在你就可以同时显示当前的 count prop 以及*它自上一次渲染以来发生了怎样的变化*。

<Sandpack>

```js src/App.js
import { useState } from 'react';
import CountLabel from './CountLabel.js';

export default function App() {
  const [count, setCount] = useState(0);
  return (
    <>
      <button onClick={() => setCount(count + 1)}>
        增加
      </button>
      <button onClick={() => setCount(count - 1)}>
        减少
      </button>
      <CountLabel count={count} />
    </>
  );
}
```

```js src/CountLabel.js active
import { useState } from 'react';

export default function CountLabel({ count }) {
  const [prevCount, setPrevCount] = useState(count);
  const [trend, setTrend] = useState(null);
  if (prevCount !== count) {
    setPrevCount(count);
    setTrend(count > prevCount ? 'increasing' : 'decreasing');
  }
  return (
    <>
      <h1>{count}</h1>
      {trend && <p>The count is {trend}</p>}
    </>
  );
}
```

```css
button { margin-bottom: 10px; }
```

</Sandpack>

请注意，如果你在渲染时调用 `set` 函数，它必须位于类似 `prevCount !== count` 的条件中，并且条件内部必须有类似 `setPrevCount(count)` 的调用。否则，你的组件会陷入循环重新渲染直到崩溃。另外，你只能像这样更新*当前正在渲染*的组件的 state。在渲染期间调用*另一个*组件的 `set` 函数是错误的。最后，你的 `set` 调用仍然应该[在不修改的情况下更新 state](#updating-objects-and-arrays-in-state)——这并不意味着你可以违反 [纯函数。](/learn/keeping-components-pure) 的其他规则。

这种模式可能很难理解，通常最好避免。不过，它比在 effect 中更新 state 更好。当你在渲染期间调用 `set` 函数时，React 会在你的组件通过 `return` 语句退出之后、渲染子组件之前立即重新渲染该组件。这样，子组件就不需要渲染两次。你组件函数的其余部分仍然会执行（结果会被丢弃）。如果你的条件位于所有 Hook 调用之后，你可以添加一个提前的 `return;` 来更早地重新开始渲染。

---

## 故障排除 {/*troubleshooting*/}

### 我已经更新了 state，但日志里拿到的还是旧值 {/*ive-updated-the-state-but-logging-gives-me-the-old-value*/}

调用 `set` 函数**不会改变正在运行代码中的状态**：

```js {4,5,8}
function handleClick() {
  console.log(count);  // 0

  setCount(count + 1); // 请求用 1 重新渲染
  console.log(count);  // 仍然是 0！

  setTimeout(() => {
    console.log(count); // 还是 0！
  }, 5000);
}
```

这是因为 [state 的行为就像一个快照。](/learn/state-as-a-snapshot) 更新 state 会请求使用新的 state 值进行另一次渲染，但不会影响你已经在运行中的事件处理函数里的 `count` JavaScript 变量。

如果你需要使用下一个 state，可以在传给 `set` 函数之前把它保存到一个变量里：

```js
const nextCount = count + 1;
setCount(nextCount);

console.log(count);     // 0
console.log(nextCount); // 1
```

---

### 我已经更新了 state，但屏幕没有更新 {/*ive-updated-the-state-but-the-screen-doesnt-update*/}

如果下一个 state 和上一个 state 相等，React 会根据 [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 比较**忽略你的更新**。这通常发生在你直接修改 state 中的对象或数组时：

```js
obj.x = 10;  // 🚩 错误：直接修改现有对象
setObj(obj); // 🚩 什么也不会发生
```

你修改了现有的 `obj` 对象，并把它传回给 `setObj`，所以 React 忽略了这次更新。要修复这个问题，你需要确保自己总是[_替换_] state 中的对象和数组，而不是[_修改_]它们：([#updating-objects-and-arrays-in-state](#updating-objects-and-arrays-in-state))

```js
// ✅ 正确：创建一个新对象
setObj({
  ...obj,
  x: 10
});
```

---

### 我遇到了一个错误：“Too many re-renders” {/*im-getting-an-error-too-many-re-renders*/}

你可能会看到这样的错误：`Too many re-renders. React limits the number of renders to prevent an infinite loop.` 通常这意味着你在 *渲染期间* 无条件地设置了 state，于是你的组件进入了一个循环：渲染、设置 state（这会导致渲染）、渲染、设置 state（这会导致渲染），如此反复。很多时候，这是因为事件处理函数的写法有误：

```js {1-2}
// 🚩 错误：在渲染期间调用处理函数
return <button onClick={handleClick()}>Click me</button>

// ✅ 正确：传递事件处理函数
return <button onClick={handleClick}>Click me</button>

// ✅ 正确：传递内联函数
return <button onClick={(e) => handleClick(e)}>Click me</button>
```

如果你找不到这个错误的原因，点击控制台中错误旁边的箭头，查看 JavaScript 调用栈，找到导致该错误的具体 `set` 函数调用。

---

### 我的初始化器或更新器函数运行了两次 {/*my-initializer-or-updater-function-runs-twice*/}

在 [严格模式](/reference/react/StrictMode) 下，React 会把你的一些函数调用两次，而不是一次：

```js {2,5-6,11-12}
function TodoList() {
  // 这个组件函数在每次渲染时都会运行两次。

  const [todos, setTodos] = useState(() => {
    // 这个初始化器函数在初始化期间会运行两次。
    return createTodos();
  });

  function handleClick() {
    setTodos(prevTodos => {
      // 这个更新器函数在每次点击时都会运行两次。
      return [...prevTodos, createTodo()];
    });
  }
  // ...
```

这是预期行为，不会破坏你的代码。

这种**仅限开发环境**的行为有助于你[保持组件纯净。](/learn/keeping-components-pure) React 会使用其中一次调用的结果，并忽略另一次调用的结果。只要你的组件、初始化器和更新器函数是纯函数，这就不会影响你的逻辑。不过，如果它们不小心变成了非纯函数，这能帮助你发现错误。

例如，这个非纯更新器函数会修改 state 中的数组：

```js {2,3}
setTodos(prevTodos => {
  // 🚩 错误：修改 state
  prevTodos.push(createTodo());
});
```

因为 React 会把你的更新器函数调用两次，所以你会看到待办事项被添加了两次，这样你就知道这里有错误。在这个例子中，你可以通过[替换数组而不是修改它](#updating-objects-and-arrays-in-state)来修复这个错误：

```js {2,3}
setTodos(prevTodos => {
  // ✅ 正确：替换为新 state
  return [...prevTodos, createTodo()];
});
```

现在这个更新器函数是纯的了，额外调用一次也不会改变行为。这就是为什么 React 调用它两次能帮助你发现错误。**只有组件、初始化器和更新器函数需要是纯的。** 事件处理函数不需要是纯的，所以 React 绝不会把你的事件处理函数调用两次。

阅读[保持组件纯净](/learn/keeping-components-pure)了解更多。

---

### 我想把 state 设置为一个函数，但它却被调用了 {/*im-trying-to-set-state-to-a-function-but-it-gets-called-instead*/}

你不能像这样把一个函数放进 state 里：

```js
const [fn, setFn] = useState(someFunction);

function handleClick() {
  setFn(someOtherFunction);
}
```

因为你传递的是一个函数，React 会认为 `someFunction` 是一个[初始化器函数](#avoiding-recreating-the-initial-state)，而 `someOtherFunction` 是一个[更新器函数](#updating-state-based-on-the-previous-state)，所以它会尝试调用它们并存储结果。要真正*存储*一个函数，你必须在这两种情况下都在它们前面加上 `() =>`。这样 React 就会存储你传入的函数。

```js {1,4}
const [fn, setFn] = useState(() => someFunction);

function handleClick() {
  setFn(() => someOtherFunction);
}
```
