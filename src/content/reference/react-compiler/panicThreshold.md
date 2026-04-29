---
title: panicThreshold
---

<Intro>

`panicThreshold` 选项控制 React Compiler 在编译期间如何处理错误。

</Intro>

```js
{
  panicThreshold: 'none' // 推荐
}
```

<InlineToc />

---

## Reference {/*reference*/}

### `panicThreshold` {/*panicthreshold*/}

决定编译错误是应导致构建失败，还是跳过优化。

#### Type {/*type*/}

```
'none' | 'critical_errors' | 'all_errors'
```

#### Default value {/*default-value*/}

`'none'`

#### Options {/*options*/}

- **`'none'`**（默认，推荐）：跳过无法编译的组件并继续构建
- **`'critical_errors'`**：仅在出现严重编译器错误时使构建失败
- **`'all_errors'`**：在出现任何编译器诊断时使构建失败

#### Caveats {/*caveats*/}

- 生产构建应始终使用 `'none'`
- 构建失败会阻止你的应用程序构建完成
- 在使用 `'none'` 时，编译器会自动检测并跳过有问题的代码
- 更高的阈值仅在开发期间用于调试

---

## Usage {/*usage*/}

### Production configuration (recommended) {/*production-configuration*/}

对于生产构建，请始终使用 `'none'`。这是默认值：

```js
{
  panicThreshold: 'none'
}
```

这可以确保：
- 你的构建不会因为编译器问题而失败
- 无法优化的组件会正常运行
- 尽可能多的组件得到优化
- 生产环境部署稳定

### Development debugging {/*development-debugging*/}

临时使用更严格的阈值来查找问题：

```js
const isDevelopment = process.env.NODE_ENV === 'development';

{
  panicThreshold: isDevelopment ? 'critical_errors' : 'none',
  logger: {
    logEvent(filename, event) {
      if (isDevelopment && event.kind === 'CompileError') {
        // ...
      }
    }
  }
}
```