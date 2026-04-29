---
title: react-dom/test-utils 弃用警告
---

## ReactDOMTestUtils.act() 警告 {/*reactdomtestutilsact-warning*/}

来自 `react-dom/test-utils` 的 `act` 已被弃用，建议改用来自 `react` 的 `act`。

之前：

```js
import {act} from 'react-dom/test-utils';
```

之后：

```js
import {act} from 'react';
```

## ReactDOMTestUtils 的其余 API {/*rest-of-reactdomtestutils-apis*/}

除 `act` 外，所有 API 都已移除。

React 团队建议将你的测试迁移到 [@testing-library/react](https://testing-library.com/docs/react-testing-library/intro/)，以获得现代且支持良好的测试体验。

### ReactDOMTestUtils.renderIntoDocument {/*reactdomtestutilsrenderintodocument*/}

`renderIntoDocument` 可以用 `@testing-library/react` 中的 `render` 替代。

之前：

```js
import {renderIntoDocument} from 'react-dom/test-utils';

renderIntoDocument(<Component />);
```

之后：

```js
import {render} from '@testing-library/react';

render(<Component />);
```

### ReactDOMTestUtils.Simulate {/*reactdomtestutilssimulate*/}

`Simulate` 可以用 `@testing-library/react` 中的 `fireEvent` 替代。

之前：

```js
import {Simulate} from 'react-dom/test-utils';

const element = document.querySelector('button');
Simulate.click(element);
```

之后：

```js
import {fireEvent} from '@testing-library/react';

const element = document.querySelector('button');
fireEvent.click(element);
```

请注意，`fireEvent` 会在元素上分发一个真实事件，而不只是以合成方式调用事件处理函数。

### 所有已移除 API 的列表 {/*list-of-all-removed-apis-list-of-all-removed-apis*/}

- `mockComponent()`
- `isElement()`
- `isElementOfType()`
- `isDOMComponent()`
- `isCompositeComponent()`
- `isCompositeComponentWithType()`
- `findAllInRenderedTree()`
- `scryRenderedDOMComponentsWithClass()`
- `findRenderedDOMComponentWithClass()`
- `scryRenderedDOMComponentsWithTag()`
- `findRenderedDOMComponentWithTag()`
- `scryRenderedComponentsWithType()`
- `findRenderedComponentWithType()`
- `renderIntoDocument`
- `Simulate`
