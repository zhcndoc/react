---
title: logger
---

<Intro>

`logger` 选项在编译期间为 React Compiler 事件提供自定义日志记录。

</Intro>

```js
{
  logger: {
    logEvent(filename, event) {
      console.log(`[Compiler] ${event.kind}: ${filename}`);
    }
  }
}
```

<InlineToc />

---

## 参考 {/*reference*/}

### `logger` {/*logger*/}

配置自定义日志记录以跟踪编译器行为并调试问题。

#### 类型 {/*type*/}

```
{
  logEvent: (filename: string | null, event: LoggerEvent) => void;
} | null
```

#### 默认值 {/*default-value*/}

`null`

#### 方法 {/*methods*/}

- **`logEvent`**：针对每个编译器事件调用，传入文件名和事件详情

#### 事件类型 {/*event-types*/}

- **`CompileSuccess`**：函数成功编译
- **`CompileError`**：由于错误而跳过函数
- **`CompileDiagnostic`**：非致命诊断信息
- **`CompileSkip`**：由于其他原因跳过函数
- **`PipelineError`**：意外的编译错误
- **`Timing`**：性能计时信息

#### 注意事项 {/*caveats*/}

- 事件结构在不同版本之间可能会发生变化
- 大型代码库会生成许多日志条目

---

## 用法 {/*usage*/}

### 基本日志记录 {/*basic-logging*/}

跟踪编译成功和失败：

```js
{
  logger: {
    logEvent(filename, event) {
      switch (event.kind) {
        case 'CompileSuccess': {
          console.log(`✅ 已编译：${filename}`);
          break;
        }
        case 'CompileError': {
          console.log(`❌ 已跳过：${filename}`);
          break;
        }
        default: {}
      }
    }
  }
}
```

### 详细错误日志记录 {/*detailed-error-logging*/}

获取有关编译失败的具体信息：

```js
{
  logger: {
    logEvent(filename, event) {
      if (event.kind === 'CompileError') {
        console.error(`\n编译失败：${filename}`);
        console.error(`原因：${event.detail.reason}`);

        if (event.detail.description) {
          console.error(`详情：${event.detail.description}`);
        }

        if (event.detail.loc) {
          const { line, column } = event.detail.loc.start;
          console.error(`位置：第 ${line} 行，第 ${column} 列`);
        }

        if (event.detail.suggestions) {
          console.error('建议：', event.detail.suggestions);
        }
      }
    }
  }
}
```

