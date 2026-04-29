---
title: gating
---

<Intro>

验证 [gating mode](/reference/react-compiler/gating) 的配置。

</Intro>

## 规则详情 {/*rule-details*/}

Gating mode 允许你通过标记特定组件进行优化，逐步采用 React Compiler。此规则可确保你的 gating 配置有效，从而让编译器知道要处理哪些组件。

### 无效 {/*invalid*/}

此规则的错误代码示例：

```js
// ❌ 缺少必需字段
module.exports = {
  plugins: [
    ['babel-plugin-react-compiler', {
      gating: {
        importSpecifierName: '__experimental_useCompiler'
        // 缺少 'source' 字段
      }
    }]
  ]
};

// ❌ 无效的 gating 类型
module.exports = {
  plugins: [
    ['babel-plugin-react-compiler', {
      gating: '__experimental_useCompiler' // 应为对象
    }]
  ]
};
```

### 有效 {/*valid*/}

此规则的正确代码示例：

```js
// ✅ 完整的 gating 配置
module.exports = {
  plugins: [
    ['babel-plugin-react-compiler', {
      gating: {
        importSpecifierName: 'isCompilerEnabled', // 导出的函数名
        source: 'featureFlags' // 模块名
      }
    }]
  ]
};

// featureFlags.js
export function isCompilerEnabled() {
  // ...
}

// ✅ 不使用 gating（编译所有内容）
module.exports = {
  plugins: [
    ['babel-plugin-react-compiler', {
      // 没有 gating 字段 - 编译所有组件
    }]
  ]
};
```
