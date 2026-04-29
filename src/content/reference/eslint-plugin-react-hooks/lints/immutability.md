---
title: 不可变性
---

<Intro>

验证是否存在对 props、state 以及其他值的变更，这些值[是不可变的](/reference/rules/components-and-hooks-must-be-pure#props-and-state-are-immutable)。

</Intro>

## 规则详情 {/*rule-details*/}

组件的 props 和 state 是不可变的快照。不要直接修改它们。应当传递新的 props，并使用 `useState` 中的 setter 函数。

## 常见违规 {/*common-violations*/}

### 无效 {/*invalid*/}

```js
// ❌ 数组 push 变更
function Component() {
  const [items, setItems] = useState([1, 2, 3]);

  const addItem = () => {
    items.push(4); // 正在变更！
    setItems(items); // 相同引用，不会重新渲染
  };
}

// ❌ 对象属性赋值
function Component() {
  const [user, setUser] = useState({name: 'Alice'});

  const updateName = () => {
    user.name = 'Bob'; // 正在变更！
    setUser(user); // 相同引用
  };
}

// ❌ 不使用展开而直接排序
function Component() {
  const [items, setItems] = useState([3, 1, 2]);

  const sortItems = () => {
    setItems(items.sort()); // sort 会修改原数组！
  };
}
```

### 有效 {/*valid*/}

```js
// ✅ 创建新数组
function Component() {
  const [items, setItems] = useState([1, 2, 3]);

  const addItem = () => {
    setItems([...items, 4]); // 新数组
  };
}

// ✅ 创建新对象
function Component() {
  const [user, setUser] = useState({name: 'Alice'});

  const updateName = () => {
    setUser({...user, name: 'Bob'}); // 新对象
  };
}
```

## 故障排查 {/*troubleshooting*/}

### 我需要向数组中添加项目 {/*add-items-array*/}

使用 `push()` 之类的方法变更数组不会触发重新渲染：

```js
// ❌ 错误：变更数组
function TodoList() {
  const [todos, setTodos] = useState([]);

  const addTodo = (id, text) => {
    todos.push({id, text});
    setTodos(todos); // 相同的数组引用！
  };

  return (
    <ul>
      {todos.map(todo => <li key={todo.id}>{todo.text}</li>)}
    </ul>
  );
}
```

改为创建一个新数组：

```js
// ✅ 更好：创建一个新数组
function TodoList() {
  const [todos, setTodos] = useState([]);

  const addTodo = (id, text) => {
    setTodos([...todos, {id, text}]);
    // 或者：setTodos(todos => [...todos, {id: Date.now(), text}])
  };

  return (
    <ul>
      {todos.map(todo => <li key={todo.id}>{todo.text}</li>)}
    </ul>
  );
}
```

### 我需要更新嵌套对象 {/*update-nested-objects*/}

变更嵌套属性不会触发重新渲染：

```js
// ❌ 错误：变更嵌套对象
function UserProfile() {
  const [user, setUser] = useState({
    name: 'Alice',
    settings: {
      theme: 'light',
      notifications: true
    }
  });

  const toggleTheme = () => {
    user.settings.theme = 'dark'; // 变更！
    setUser(user); // 相同的对象引用
  };
}
```

在需要更新的每一层都进行展开：

```js
// ✅ 更好：在每一层都创建新对象
function UserProfile() {
  const [user, setUser] = useState({
    name: 'Alice',
    settings: {
      theme: 'light',
      notifications: true
    }
  });

  const toggleTheme = () => {
    setUser({
      ...user,
      settings: {
        ...user.settings,
        theme: 'dark'
      }
    });
  };
}
```