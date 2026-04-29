---
title: target
---

<Intro>

`target` 选项指定编译器应为哪个 React 版本生成代码。

</Intro>

```js
{
  target: '19' // 或 '18'、'17'
}
```

<InlineToc />

---

## 参考 {/*reference*/}

### `target` {/*target*/}

配置编译后输出的 React 版本兼容性。

#### 类型 {/*type*/}

```
'17' | '18' | '19'
```

#### 默认值 {/*default-value*/}

`'19'`

#### 有效值 {/*valid-values*/}

- **`'19'`**：目标 React 19（默认）。不需要额外的运行时。
- **`'18'`**：目标 React 18。需要 `react-compiler-runtime` 包。
- **`'17'`**：目标 React 17。需要 `react-compiler-runtime` 包。

#### 注意事项 {/*caveats*/}

- 始终使用字符串值，而不是数字（例如，`'17'` 而不是 `17`）
- 不要包含补丁版本号（例如，使用 `'18'` 而不是 `'18.2.0'`）
- React 19 包含内置的编译器运行时 API
- React 17 和 18 需要安装 `react-compiler-runtime@latest`

---

## 用法 {/*usage*/}

### 以 React 19 为目标（默认） {/*targeting-react-19*/}

对于 React 19，无需特殊配置：

```js
{
  // 默认为 target: '19'
}
```

编译器将使用 React 19 内置的运行时 API：

```js
// 编译后的输出使用 React 19 的原生 API
import { c as _c } from 'react/compiler-runtime';
```

### 以 React 17 或 18 为目标 {/*targeting-react-17-or-18*/}

对于 React 17 和 React 18 项目，你需要两个步骤：

1. 安装运行时包：

```bash
npm install react-compiler-runtime@latest
```

2. 配置目标版本：

```js
// 对于 React 18
{
  target: '18'
}

// 对于 React 17
{
  target: '17'
}
```

编译器将为这两个版本使用 polyfill 运行时：

```js
// 编译后的输出使用 polyfill
import { c as _c } from 'react-compiler-runtime';
```

---

## 故障排查 {/*troubleshooting*/}

### 关于缺少编译器运行时的运行时错误 {/*missing-runtime*/}

如果你看到类似 “Cannot find module 'react/compiler-runtime'” 的错误：

1. 检查你的 React 版本：
   ```bash
   npm why react
   ```

2. 如果使用 React 17 或 18，请安装运行时：
   ```bash
   npm install react-compiler-runtime@latest
   ```

3. 确保你的 target 与 React 版本匹配：
   ```js
   {
     target: '18' // 必须与你的 React 主版本匹配
   }
   ```

### 运行时包不起作用 {/*runtime-not-working*/}

请确保运行时包：

1. 安装在你的项目中（而不是全局）
2. 列在你的 `package.json` 依赖项中
3. 使用正确的版本（`@latest` 标签）
4. 不在 `devDependencies` 中（它在运行时需要）

### 检查编译后的输出 {/*checking-output*/}

要验证是否使用了正确的运行时，请注意不同的导入（内置使用 `react/compiler-runtime`，17/18 的独立包使用 `react-compiler-runtime`）：

```js
// 对于 React 19（内置运行时）
import { c } from 'react/compiler-runtime'
//                      ^

// 对于 React 17/18（polyfill 运行时）
import { c } from 'react-compiler-runtime'
//                      ^
```