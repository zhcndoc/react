---
title: Server Components
---

<Intro>

Server Components 是一种新的组件类型，它会在打包之前提前渲染，并且运行在与你的客户端应用或 SSR 服务器分离的环境中。

</Intro>

这个独立的环境就是 React Server Components 中的“服务器”。Server Components 可以在 CI 服务器上于构建时运行一次，也可以使用 Web 服务器在每个请求时运行。

<InlineToc />

<Note>

#### 如何为 Server Components 构建支持？ {/*how-do-i-build-support-for-server-components*/}

虽然 React 19 中的 React Server Components 已经稳定，并且不会在小版本之间发生破坏性变更，但用于实现 React Server Components 打包器或框架的底层 API 不遵循 semver，可能会在 React 19.x 的小版本之间发生变化。

为了在打包器或框架中支持 React Server Components，我们建议锁定到特定的 React 版本，或者使用 Canary 版本。我们将继续与打包器和框架合作，未来让用于实现 React Server Components 的 API 稳定下来。

</Note>

### 没有服务器的 Server Components {/*server-components-without-a-server*/}
Server components 可以在构建时运行，从文件系统读取内容或获取静态内容，因此并不需要 Web 服务器。例如，你可能希望从内容管理系统中读取静态数据。

如果没有 Server Components，通常会在客户端通过 Effect 获取静态数据：

```js
// bundle.js
import marked from 'marked'; // 35.9K（11.2K gzipped）
import sanitizeHtml from 'sanitize-html'; // 206K（63.3K gzipped）

function Page({page}) {
  const [content, setContent] = useState('');
  // 注意：在第一次页面渲染之后才加载。
  useEffect(() => {
    fetch(`/api/content/${page}`).then((data) => {
      setContent(data.content);
    });
  }, [page]);

  return <div>{sanitizeHtml(marked(content))}</div>;
}
```
```js
// api.js
app.get(`/api/content/:page`, async (req, res) => {
  const page = req.params.page;
  const content = await file.readFile(`${page}.md`);
  res.send({content});
});
```

这种模式意味着用户需要额外下载并解析 75K（gzipped）的库，并且在页面加载后还要等待第二次请求来获取数据，仅仅是为了渲染生命周期内不会变化的静态内容。

使用 Server Components，你可以在构建时一次性渲染这些组件：

```js
import marked from 'marked'; // 不包含在 bundle 中
import sanitizeHtml from 'sanitize-html'; // 不包含在 bundle 中

async function Page({page}) {
  // 注意：在应用构建时于渲染期间加载。
  const content = await file.readFile(`${page}.md`);

  return <div>{sanitizeHtml(marked(content))}</div>;
}
```

然后，渲染结果可以被服务端渲染（SSR）为 HTML，并上传到 CDN。当应用加载时，客户端不会看到原始的 `Page` 组件，也不会看到用于渲染 markdown 的昂贵库。客户端只会看到渲染后的输出：

```js
<div><!-- markdown 的 HTML --></div>
```

这意味着内容会在首次页面加载时就可见，而且 bundle 中不包含渲染静态内容所需的昂贵库。

<Note>

你可能注意到上面的 Server Component 是一个 async 函数：

```js
async function Page({page}) {
  //...
}
```

Async Components 是 Server Components 的一项新特性，它允许你在 render 中使用 `await`。

请参见下方的 [Server Components 中的异步组件](#async-components-with-server-components)。

</Note>

### 带有服务器的 Server Components {/*server-components-with-a-server*/}
Server Components 也可以在请求页面时运行在 Web 服务器上，让你无需构建 API 就能访问数据层。它们会在你的应用被打包之前渲染，并且可以将数据和 JSX 作为 props 传递给 Client Components。

如果没有 Server Components，通常会在客户端通过 Effect 获取动态数据：

```js
// bundle.js
function Note({id}) {
  const [note, setNote] = useState('');
  // 注意：在第一次渲染之后才加载。
  useEffect(() => {
    fetch(`/api/notes/${id}`).then(data => {
      setNote(data.note);
    });
  }, [id]);

  return (
    <div>
      <Author id={note.authorId} />
      <p>{note}</p>
    </div>
  );
}

function Author({id}) {
  const [author, setAuthor] = useState('');
  // 注意：在 Note 渲染之后才加载。
  // 这会导致昂贵的客户端到服务器瀑布请求。
  useEffect(() => {
    fetch(`/api/authors/${id}`).then(data => {
      setAuthor(data.author);
    });
  }, [id]);

  return <span>作者：{author.name}</span>;
}
```
```js
// api
import db from './database';

app.get(`/api/notes/:id`, async (req, res) => {
  const note = await db.notes.get(id);
  res.send({note});
});

app.get(`/api/authors/:id`, async (req, res) => {
  const author = await db.authors.get(id);
  res.send({author});
});
```

使用 Server Components，你可以在组件中读取数据并进行渲染：

```js
import db from './database';

async function Note({id}) {
  // 注意：在渲染期间加载。
  const note = await db.notes.get(id);
  return (
    <div>
      <Author id={note.authorId} />
      <p>{note}</p>
    </div>
  );
}

async function Author({id}) {
  // 注意：在 Note 之后加载，
  // 但如果数据是同地存放的，则会很快。
  const author = await db.authors.get(id);
  return <span>作者：{author.name}</span>;
}
```

然后，打包器会将数据、渲染后的 Server Components 以及动态 Client Components 合并到一个 bundle 中。可选地，这个 bundle 还可以再进行服务端渲染（SSR），为页面生成初始 HTML。当页面加载时，浏览器不会看到原始的 `Note` 和 `Author` 组件；只有渲染后的输出会发送给客户端：

```js
<div>
  <span>作者：React 团队</span>
  <p>React 19 是...</p>
</div>
```

Server Components 可以通过从服务器重新获取它们来变为动态化，在服务器上它们可以再次访问数据并重新渲染。这个新的应用架构将服务端中心的多页应用简单的“请求/响应”心智模型，与客户端中心的单页应用无缝交互性结合在一起，为你提供两全其美的体验。

### 为 Server Components 添加交互性 {/*adding-interactivity-to-server-components*/}

Server Components 不会发送到浏览器，因此它们不能使用 `useState` 之类的交互式 API。要为 Server Components 添加交互性，你可以使用 `"use client"` 指令将它们与 Client Component 组合起来。

<Note>

#### 没有用于 Server Components 的指令。 {/*there-is-no-directive-for-server-components*/}

一种常见的误解是 Server Components 由 `"use server"` 标记，但实际上并没有用于 Server Components 的指令。`"use server"` 指令是用于 Server Functions 的。

更多信息请参见 [Directives](/reference/rsc/directives) 文档。

</Note>


在下面的示例中，`Notes` 这个 Server Component 导入了一个 `Expandable` Client Component，它使用 state 来切换其 `expanded` 状态：
```js
// Server Component
import Expandable from './Expandable';

async function Notes() {
  const notes = await db.notes.getAll();
  return (
    <div>
      {notes.map(note => (
        <Expandable key={note.id}>
          <p note={note} />
        </Expandable>
      ))}
    </div>
  )
}
```
```js
// Client Component
"use client"

export default function Expandable({children}) {
  const [expanded, setExpanded] = useState(false);
  return (
    <div>
      <button
        onClick={() => setExpanded(!expanded)}
      >
        切换
      </button>
      {expanded && children}
    </div>
  )
}
```

其工作方式是先将 `Notes` 作为 Server Component 渲染，然后指示打包器为 Client Component `Expandable` 创建一个 bundle。在浏览器中，Client Components 会接收到作为 props 传入的 Server Components 输出：

```js
<head>
  <!-- Client Components 的 bundle -->
  <script src="bundle.js" />
</head>
<body>
  <div>
    <Expandable key={1}>
      <p>这是第一条笔记</p>
    </Expandable>
    <Expandable key={2}>
      <p>这是第二条笔记</p>
    </Expandable>
    <!--...-->
  </div>
</body>
```

### 使用 Server Components 的异步组件 {/*async-components-with-server-components*/}

Server Components 引入了一种使用 async/await 编写组件的新方式。当你在异步组件中 `await` 时，React 会挂起并等待 promise 解析后再继续渲染。借助 Suspense 的流式支持，这种方式可以跨越 server/client 边界工作。

你甚至可以在服务端创建一个 promise，然后在客户端 await 它：

```js
// Server Component
import db from './database';

async function Page({id}) {
  // 将会挂起 Server Component。
  const note = await db.notes.get(id);

  // 注意：没有 await，会从这里开始，并在客户端 await。
  const commentsPromise = db.comments.get(note.id);
  return (
    <div>
      {note}
      <Suspense fallback={<p>正在加载评论...</p>}>
        <Comments commentsPromise={commentsPromise} />
      </Suspense>
    </div>
  );
}
```

```js
// Client Component
"use client";
import {use} from 'react';

function Comments({commentsPromise}) {
  // 注意：这会恢复来自服务器的 promise。
  // 它会一直挂起，直到数据可用。
  const comments = use(commentsPromise);
  return comments.map(comment => <p>{comment}</p>);
}
```

`note` 内容是页面渲染所需的重要数据，所以我们在服务端 `await` 它。评论位于首屏下方，优先级更低，所以我们在服务端启动这个 promise，然后在客户端使用 `use` API 等待它。这会在客户端触发 Suspense，而不会阻塞 `note` 内容的渲染。

由于客户端不支持异步组件，我们使用 `use` 来 await 这个 promise。
