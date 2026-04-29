---
title: experimental_taintObjectReference
version: experimental
---

<Experimental>

**此 API 处于实验阶段，React 的稳定版本中尚不可用。**

你可以通过升级 React 包到最新的实验版本来尝试它：

- `react@experimental`
- `react-dom@experimental`
- `eslint-plugin-react-hooks@experimental`

React 的实验版本可能包含 bug。不要在生产环境中使用它们。

此 API 仅可在 React Server Components 内部使用。

</Experimental>


<Intro>

`taintObjectReference` 允许你阻止某个特定对象实例被传递给像 `user` 对象这样的 Client Component。

```js
experimental_taintObjectReference(message, object);
```

要阻止传递 key、hash 或 token，请参阅 [`taintUniqueValue`](/reference/react/experimental_taintUniqueValue)。

</Intro>

<InlineToc />

---

## Reference {/*reference*/}

### `taintObjectReference(message, object)` {/*taintobjectreference*/}

使用对象调用 `taintObjectReference`，将其注册到 React 中，表示它不应原样传递给 Client：

```js
import {experimental_taintObjectReference} from 'react';

experimental_taintObjectReference(
  '不要将所有环境变量传递给客户端。',
  process.env
);
```

[查看更多示例。](#usage)

#### Parameters {/*parameters*/}

* `message`: 如果该对象被传递给 Client Component 时，你希望显示的消息。若该对象被传递给 Client Component，这条消息将作为将要抛出的 Error 的一部分显示出来。

* `object`: 要被 taint 的对象。函数和类实例可以作为 `object` 传递给 `taintObjectReference`。函数和类本来就已被禁止传递给 Client Components，但 React 的默认错误消息会被你在 `message` 中定义的内容替换。当某个 Typed Array 的特定实例作为 `object` 传递给 `taintObjectReference` 时，该 Typed Array 的任何其他副本都不会被 taint。

#### Returns {/*returns*/}

`experimental_taintObjectReference` 返回 `undefined`。

#### Caveats {/*caveats*/}

- 重新创建或克隆一个被 taint 的对象会创建一个新的未被 taint 的对象，其中可能包含敏感数据。例如，如果你有一个被 taint 的 `user` 对象，`const userInfo = {name: user.name, ssn: user.ssn}` 或 `{...user}` 会创建新的、未被 taint 的对象。`taintObjectReference` 只能防止对象在原样传递给 Client Component 时出现的简单错误。

<Pitfall>

**不要只依赖 taint 作为安全措施。** 对对象进行 taint 不能阻止泄漏所有可能的派生值。例如，被 taint 对象的克隆会创建一个新的未被 taint 对象。使用被 taint 对象中的数据（例如 `{secret: taintedObj.secret}`）会创建一个新的、未被 taint 的值或对象。taint 只是保护层之一；安全的应用会有多层防护、设计良好的 API，以及隔离模式。

</Pitfall>

---

## Usage {/*usage*/}

### 防止用户数据无意中到达客户端 {/*prevent-user-data-from-unintentionally-reaching-the-client*/}

Client Component 不应接受携带敏感数据的对象。理想情况下，数据获取函数不应暴露当前用户无权访问的数据。有时在重构过程中会发生错误。为了防止这类错误在后续发生，我们可以在数据 API 中对用户对象进行 “taint”。

```js
import {experimental_taintObjectReference} from 'react';

export async function getUser(id) {
  const user = await db`SELECT * FROM users WHERE id = ${id}`;
  experimental_taintObjectReference(
    '不要将整个用户对象传递给客户端。' +
      '请改为选取此用例所需的特定属性。',
    user,
  );
  return user;
}
```

现在，每当有人试图将这个对象传递给 Client Component 时，都会抛出一个错误，并显示你传入的错误消息。

<DeepDive>

#### 防止数据获取中的泄漏 {/*protecting-against-leaks-in-data-fetching*/}

如果你运行的是一个可以访问敏感数据的 Server Components 环境，就必须小心不要直接将对象透传出去：

```js
// api.js
export async function getUser(id) {
  const user = await db`SELECT * FROM users WHERE id = ${id}`;
  return user;
}
```

```js
import { getUser } from 'api.js';
import { InfoCard } from 'components.js';

export async function Profile(props) {
  const user = await getUser(props.userId);
  // 不要这样做
  return <InfoCard user={user} />;
}
```

```js
// components.js
"use client";

export async function InfoCard({ user }) {
  return <div>{user.name}</div>;
}
```

理想情况下，`getUser` 不应暴露当前用户无权访问的数据。为了防止后续有人将 `user` 对象传递给 Client Component，我们可以对用户对象进行 “taint”：

```js
// api.js
import {experimental_taintObjectReference} from 'react';

export async function getUser(id) {
  const user = await db`SELECT * FROM users WHERE id = ${id}`;
  experimental_taintObjectReference(
    '不要将整个用户对象传递给客户端。' +
      '请改为选取此用例所需的特定属性。',
    user,
  );
  return user;
}
```

现在，如果有人试图将 `user` 对象传递给 Client Component，就会抛出一个错误，并显示你传入的错误消息。

</DeepDive>
