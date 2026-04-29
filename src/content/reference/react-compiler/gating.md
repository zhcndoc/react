---
title: gating
---

<Intro>

`gating` 选项启用条件编译，允许你控制优化后的代码在运行时何时被使用。

</Intro>

```js
{
  gating: {
    source: 'my-feature-flags',
    importSpecifierName: 'shouldUseCompiler'
  }
}
```

<InlineToc />

---

## 参考 {/*reference*/}

### `gating` {/*gating*/}

为已编译函数配置运行时特性标志门控。

#### 类型 {/*type*/}

```
{
  source: string;
  importSpecifierName: string;
} | null
```

#### 默认值 {/*default-value*/}

`null`

#### 属性 {/*properties*/}

- **`source`**：要从中导入特性标志的模块路径
- **`importSpecifierName`**：要导入的已导出函数名称

#### 注意事项 {/*caveats*/}

- 门控函数必须返回布尔值
- 编译版本和原始版本都会增加包体积
- 每个包含已编译函数的文件都会添加该导入

---

## 用法 {/*usage*/}

### 基础特性标志设置 {/*basic-setup*/}

1. 创建一个特性标志模块：

```js
// src/utils/feature-flags.js
export function shouldUseCompiler() {
  // 你的逻辑在这里
  return getFeatureFlag('react-compiler-enabled');
}
```

2. 配置编译器：

```js
{
  gating: {
    source: './src/utils/feature-flags',
    importSpecifierName: 'shouldUseCompiler'
  }
}
```

3. 编译器生成带门控的代码：

```js
// 输入
function Button(props) {
  return <button>{props.label}</button>;
}

// 输出（简化）
import { shouldUseCompiler } from './src/utils/feature-flags';

const Button = shouldUseCompiler()
  ? function Button_optimized(props) { /* 编译版本 */ }
  : function Button_original(props) { /* 原始版本 */ };
```

请注意，门控函数会在模块加载时执行一次，因此一旦 JS 包被解析并执行，组件的选择在浏览器会话的剩余时间内都会保持静态。

---

## 故障排查 {/*troubleshooting*/}

### 特性标志不工作 {/*flag-not-working*/}

请验证你的标志模块导出了正确的函数：

```js
// ❌ 错误：默认导出
export default function shouldUseCompiler() {
  return true;
}

// ✅ 正确：与 importSpecifierName 匹配的命名导出
export function shouldUseCompiler() {
  return true;
}
```

### 导入错误 {/*import-errors*/}

确保 source 路径正确：

```js
// ❌ 错误：相对于 babel.config.js
{
  source: './src/flags',
  importSpecifierName: 'flag'
}

// ✅ 正确：模块解析路径
{
  source: '@myapp/feature-flags',
  importSpecifierName: 'flag'
}

// ✅ 也正确：从项目根目录开始的绝对路径
{
  source: './src/utils/flags',
  importSpecifierName: 'flag'
}
```
