---
title: unsupported-syntax
---

<Intro>

验证 React Compiler 不支持的语法。如果需要，你仍然可以在 React 之外使用这种语法，例如在独立的工具函数中。

</Intro>

## Rule Details {/*rule-details*/}

React Compiler 需要静态分析你的代码以应用优化。像 `eval` 和 `with` 这样的特性会使编译时不可能静态理解代码在做什么，因此编译器无法优化使用它们的组件。

### Invalid {/*invalid*/}

此规则的错误代码示例：

```js
// ❌ 在组件中使用 eval
function Component({ code }) {
  const result = eval(code); // 无法被分析
  return <div>{result}</div>;
}

// ❌ 使用 with 语句
function Component() {
  with (Math) { // 动态改变作用域
    return <div>{sin(PI / 2)}</div>;
  }
}

// ❌ 使用 eval 进行动态属性访问
function Component({propName}) {
  const value = eval(`props.${propName}`);
  return <div>{value}</div>;
}
```

### Valid {/*valid*/}

此规则的正确代码示例：

```js
// ✅ 使用普通属性访问
function Component({propName, props}) {
  const value = props[propName]; // 可分析
  return <div>{value}</div>;
}

// ✅ 使用标准的 Math 方法
function Component() {
  return <div>{Math.sin(Math.PI / 2)}</div>;
}
```

## Troubleshooting {/*troubleshooting*/}

### I need to evaluate dynamic code {/*evaluate-dynamic-code*/}

你可能需要执行用户提供的代码：

```js {expectedErrors: {'react-compiler': [3]}}
// ❌ 错误：在组件中使用 eval
function Calculator({expression}) {
  const result = eval(expression); // 不安全且无法优化
  return <div>结果：{result}</div>;
}
```

改用安全的表达式解析器：

```js
// ✅ 更好的方式：使用安全的解析器
import {evaluate} from 'mathjs'; // 或类似的库

function Calculator({expression}) {
  const [result, setResult] = useState(null);

  const calculate = () => {
    try {
      // 安全的数学表达式求值
      setResult(evaluate(expression));
    } catch (error) {
      setResult('无效表达式');
    }
  };

  return (
    <div>
      <button onClick={calculate}>计算</button>
      {result && <div>结果：{result}</div>}
    </div>
  );
}
```

<Note>

绝不要将 `eval` 与用户输入一起使用——这有安全风险。针对特定用例请使用专门的解析库，例如数学表达式、JSON 解析或模板求值。

</Note>