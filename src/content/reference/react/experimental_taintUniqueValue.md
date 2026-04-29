---
title: experimental_taintUniqueValue
version: experimental
---

<Experimental>

**此 API 仍处于实验阶段，尚未在稳定版 React 中提供。**

你可以通过将 React 包升级到最新的实验版本来试用它：

- `react@experimental`
- `react-dom@experimental`
- `eslint-plugin-react-hooks@experimental`

React 的实验版本可能包含 bug。不要在生产环境中使用它们。

此 API 仅可在 [React Server Components](/reference/rsc/use-client) 内部使用。

</Experimental>


<Intro>

`taintUniqueValue` 允许你阻止唯一值传递给 Client Components，例如密码、密钥或令牌。

```js
taintUniqueValue(errMessage, lifetime, value)
```

要防止传递包含敏感数据的对象，请参见 [`taintObjectReference`](/reference/react/experimental_taintObjectReference)。

</Intro>

<InlineToc />

---

## 参考 {/*reference*/}

### `taintUniqueValue(message, lifetime, value)` {/*taintuniquevalue*/}

使用密码、令牌、密钥或哈希调用 `taintUniqueValue`，将其注册到 React，表示它不应被原样传递给客户端：

```js
import {experimental_taintUniqueValue} from 'react';

experimental_taintUniqueValue(
  'Do not pass secret keys to the client.',
  process,
  process.env.SECRET_KEY
);
```

[查看下面的更多示例。](#usage)

#### 参数 {/*parameters*/}

* `message`：当 `value` 传递给 Client Component 时你想要显示的消息。如果 `value` 被传递给 Client Component，这条消息会作为将要抛出的错误的一部分显示。

* `lifetime`：任意对象，表示 `value` 应该被标记多长时间。只要这个对象仍然存在，`value` 就会被阻止发送到任何 Client Component。例如，传入 `globalThis` 会在应用的整个生命周期内阻止该值。`lifetime` 通常是一个其属性包含 `value` 的对象。

* `value`：一个 string、bigint 或 TypedArray。`value` 必须是具有高熵的唯一字符或字节序列，例如加密令牌、私钥、哈希，或长密码。`value` 将被阻止发送到任何 Client Component。

#### 返回值 {/*returns*/}

`experimental_taintUniqueValue` 返回 `undefined`。

#### 注意事项 {/*caveats*/}

* 从被标记的值派生新值可能会破坏标记保护。将被标记的值转换为大写、将被标记的字符串值拼接成更长的字符串、将被标记的值转换为 base64、截取被标记的值子串，以及其他类似转换，除非你显式地对这些新创建的值调用 `taintUniqueValue`，否则它们都不会被标记。
* 不要使用 `taintUniqueValue` 来保护低熵值，例如 PIN 码或电话号码。如果请求中的任意值都可能被攻击者控制，他们可以通过枚举所有可能的密钥值来推断出哪个值被标记了。

---

## 用法 {/*usage*/}

### 防止令牌被传递给 Client Components {/*prevent-a-token-from-being-passed-to-client-components*/}

为了确保密码、会话令牌或其他唯一值等敏感信息不会无意中传递给 Client Components，`taintUniqueValue` 函数提供了一层保护。当一个值被标记后，任何试图将其传递给 Client Component 的行为都会导致错误。

`lifetime` 参数定义了该值保持被标记的持续时间。对于应当无限期保持被标记的值，可以使用诸如 [`globalThis`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/globalThis) 或 `process` 这样的对象作为 `lifetime` 参数。这些对象的生命周期贯穿你的应用整个运行期间。

```js
import {experimental_taintUniqueValue} from 'react';

experimental_taintUniqueValue(
  'Do not pass a user password to the client.',
  globalThis,
  process.env.SECRET_KEY
);
```

如果被标记值的生命周期与某个对象绑定，那么 `lifetime` 应该是封装该值的对象。这样可以确保该被标记值在封装对象的整个生命周期内都受到保护。

```js
import {experimental_taintUniqueValue} from 'react';

export async function getUser(id) {
  const user = await db`SELECT * FROM users WHERE id = ${id}`;
  experimental_taintUniqueValue(
    'Do not pass a user session token to the client.',
    user,
    user.session.token
  );
  return user;
}
```

在这个示例中，`user` 对象充当 `lifetime` 参数。如果该对象被存储到全局缓存中，或者可被另一个请求访问，那么会话令牌仍然保持被标记状态。

<Pitfall>

**不要仅仅依赖标记机制来保证安全。** 对值进行标记并不能阻止所有可能的派生值。例如，通过将一个被标记的字符串转换为大写来创建新值，并不会让新值也被标记。


```js
import {experimental_taintUniqueValue} from 'react';

const password = 'correct horse battery staple';

experimental_taintUniqueValue(
  'Do not pass the password to the client.',
  globalThis,
  password
);

const uppercasePassword = password.toUpperCase() // `uppercasePassword` is not tainted
```

在这个示例中，常量 `password` 被标记了。然后通过对 `password` 调用 `toUpperCase` 方法，创建了新值 `uppercasePassword`。新创建的 `uppercasePassword` 并未被标记。

从被标记值派生新值的其他类似方式，比如将其拼接成更长的字符串、将其转换为 base64，或返回子字符串，都会创建未被标记的值。

标记机制只能防止诸如显式将机密值传递给客户端这类简单错误。在调用 `taintUniqueValue` 时如果出现错误，例如在 React 之外使用全局存储而没有对应的生命周期对象，就可能导致被标记的值变为未标记。标记只是保护层之一；安全的应用需要多层保护、设计良好的 API 以及隔离模式。

</Pitfall>

<DeepDive>

#### 使用 `server-only` 和 `taintUniqueValue` 防止秘密泄露 {/*using-server-only-and-taintuniquevalue-to-prevent-leaking-secrets*/}

如果你运行的是一个可访问私钥或密码（例如数据库密码）的 Server Components 环境，就必须小心不要把它传递给 Client Component。

```js
export async function Dashboard(props) {
  // 不要这样做
  return <Overview password={process.env.API_PASSWORD} />;
}
```

```js
"use client";

import {useEffect} from '...'

export async function Overview({ password }) {
  useEffect(() => {
    const headers = { Authorization: password };
    fetch(url, { headers }).then(...);
  }, [password]);
  ...
}
```

这个示例会将机密 API 令牌泄露给客户端。如果这个 API 令牌可用于访问该特定用户不应有权访问的数据，就可能导致数据泄露。

[comment]: <> (TODO: 在 `server-only` 文档编写完成后链接到它)

理想情况下，这类机密应被抽象到单独的辅助文件中，并且该文件只能被服务器端受信任的数据工具导入。该辅助文件甚至可以标记为 [`server-only`](https://www.npmjs.com/package/server-only)，以确保此文件不会在客户端被导入。

```js
import "server-only";

export function fetchAPI(url) {
  const headers = { Authorization: process.env.API_PASSWORD };
  return fetch(url, { headers });
}
```

有时在重构过程中会发生错误，而且并非所有同事都知道这一点。
为了防止这类错误在后续发生，我们可以把实际密码“标记”起来：

```js
import "server-only";
import {experimental_taintUniqueValue} from 'react';

experimental_taintUniqueValue(
  'Do not pass the API token password to the client. ' +
    'Instead do all fetches on the server.'
  process,
  process.env.API_PASSWORD
);
```

现在，只要有人试图将这个密码传递给 Client Component，或者通过 Server Function 将密码发送给 Client Component，就会抛出一个错误，错误信息就是你在调用 `taintUniqueValue` 时定义的消息。

</DeepDive>

---
