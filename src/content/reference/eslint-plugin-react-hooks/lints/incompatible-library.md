---
title: incompatible-library
---

<Intro>

验证与与 memoization（手动或自动）不兼容的库使用情况。

</Intro>

<Note>

这些库是在 React 的 memoization 规则尚未被充分记录之前设计的。当时它们做出了正确的选择，以优化在应用状态变化时让组件保持“恰到好处”的响应性这一更符合开发体验的方式。虽然这些旧模式曾经有效，但我们后来发现它们与 React 的编程模型不兼容。我们将继续与库作者合作，将这些库迁移到遵循 React 规则的模式。

</Note>

## Rule Details {/*rule-details*/}

某些库使用了 React 不支持的模式。当 linter 检测到来自[已知列表](https://github.com/facebook/react/blob/main/compiler/packages/babel-plugin-react-compiler/src/HIR/DefaultModuleTypeProvider.ts)中的这些 API 的使用时，会根据这条规则将其标记出来。这意味着 React Compiler 可以自动跳过使用这些不兼容 API 的组件，从而避免破坏你的应用。

```js
// 这些库中 memoization 失效的示例
function Form() {
  const { watch } = useForm();

  // ❌ 即使 'name' 字段变化，这个值也永远不会更新
  const name = useMemo(() => watch('name'), [watch]);

  return <div>Name: {name}</div>; // UI 看起来“冻结”了
}
```

React Compiler 会根据 React 规则自动对值进行 memoization。如果手动使用 `useMemo` 会出错，那么编译器的自动优化也会出错。这条规则有助于识别这些有问题的模式。

<DeepDive>

#### 设计遵循 React 规则的 API {/*designing-apis-that-follow-the-rules-of-react*/}

在设计库的 API 或 hook 时，需要思考的一个问题是：调用该 API 是否可以安全地用 `useMemo` 进行 memoization。如果不行，那么无论是手动 memoization 还是 React Compiler 的 memoization，都会破坏用户的代码。

例如，一种不兼容的模式是“内部可变性”（interior mutability）。内部可变性指的是：一个对象或函数会保留自己的隐藏状态，并且这个状态会随着时间发生变化，尽管它的引用保持不变。可以把它想象成一个外表看起来一样、但内部内容会悄悄重排的盒子。React 无法判断任何东西变了，因为它只会检查你是否给了它一个不同的盒子，而不会看盒子里面是什么。这会破坏 memoization，因为 React 依赖外层对象（或函数）在其值的某部分发生变化时也随之变化。

通常来说，在设计 React API 时，要思考 `useMemo` 是否会把它弄坏：

```js
function Component() {
  const { someFunction } = useLibrary();
  // 像这样的函数，应该始终可以安全地进行 memoization
  const result = useMemo(() => someFunction(), [someFunction]);
}
```

相反，应当设计返回不可变状态并使用显式更新函数的 API：

```js
// ✅ 好：返回不可变状态，在更新时引用会变化
function Component() {
  const { field, updateField } = useLibrary();
  // 这样做始终可以安全地进行 memo
  const greeting = useMemo(() => `Hello, ${field.name}!`, [field.name]);

  return (
    <div>
      <input
        value={field.name}
        onChange={(e) => updateField('name', e.target.value)}
      />
      <p>{greeting}</p>
    </div>
  );
}
```

</DeepDive>

### Invalid {/*invalid*/}

以下是此规则的错误代码示例：

```js
// ❌ react-hook-form `watch`
function Component() {
  const {watch} = useForm();
  const value = watch('field'); // 内部可变性
  return <div>{value}</div>;
}

// ❌ TanStack Table `useReactTable`
function Component({data}) {
  const table = useReactTable({
    data,
    columns,
    getCoreRowModel: getCoreRowModel(),
  });
  // table 实例使用了内部可变性
  return <Table table={table} />;
}
```

<Pitfall>

#### MobX {/*mobx*/}

像 `observer` 这样的 MobX 模式也会破坏 memoization 的假设，但 linter 目前还不能检测到它们。如果你依赖 MobX，并且发现你的应用与 React Compiler 不兼容，你可能需要使用 `"use no memo" 指令`。

```js
// ❌ MobX `observer`
const Component = observer(() => {
  const [timer] = useState(() => new Timer());
  return <span>已经过去的秒数：{timer.secondsPassed}</span>;
});
```

</Pitfall>

### Valid {/*valid*/}

以下是此规则的正确代码示例：

```js
// ✅ 对于 react-hook-form，使用 `useWatch`：
function Component() {
  const {register, control} = useForm();
  const watchedValue = useWatch({
    control,
    name: 'field'
  });

  return (
    <>
      <input {...register('field')} />
      <div>当前值：{watchedValue}</div>
    </>
  );
}
```

还有一些其他库目前还没有与 React 的 memoization 模型兼容的替代 API。如果 linter 不能自动跳过调用这些 API 的组件或 hooks，请[提交 issue](https://github.com/facebook/react/issues)，这样我们就可以把它添加到 linter 中。
