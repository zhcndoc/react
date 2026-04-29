---
title: "React Labs: 视图过渡、Activity 及更多内容"
author: Ricky Hanlon
date: 2025/04/23
description: 在 React Labs 的文章中，我们会介绍处于积极研究与开发中的项目。在这篇文章里，我们将分享两个今天即可尝试的新实验性功能，以及我们当前正在推进的其他领域的更新。
---

2025 年 4 月 23 日，作者 [Ricky Hanlon](https://twitter.com/rickhanlonii)

---

<Intro>

在 React Labs 的文章中，我们会介绍处于积极研究与开发中的项目。在这篇文章里，我们将分享两个今天即可尝试的新实验性功能，以及我们当前正在推进的其他领域的更新。

</Intro>


今天，我们很高兴发布两个可供测试的新实验性功能的文档：

- [视图过渡](#view-transitions)
- [Activity](#activity)

我们还会分享当前正在开发中的新功能更新：
- [React 性能轨道](#react-performance-tracks)
- [编译器 IDE 扩展](#compiler-ide-extension)
- [自动 Effect 依赖](#automatic-effect-dependencies)
- [Fragment Refs](#fragment-refs)
- [并发存储](#concurrent-stores)

---

# 新的实验性功能 {/*new-experimental-features*/}

<Note>

`<Activity />` 已在 `react@19.2` 中发布。

`<ViewTransition />` 和 `addTransitionType` 现已在 `react@canary` 中可用。

</Note>

View Transitions 和 Activity 现已可在 `react@experimental` 中进行测试。这些功能已经在生产环境中经过测试并且是稳定的，但随着我们吸收反馈，最终 API 仍可能发生变化。

你可以通过将 React 包升级到最新的实验版本来试用它们：

- `react@experimental`
- `react-dom@experimental`

继续阅读，了解如何在你的应用中使用这些功能，或者查看新发布的文档：

- [`<ViewTransition>`](/reference/react/ViewTransition)：一个让你为 Transition 启用动画的组件。
- [`addTransitionType`](/reference/react/addTransitionType)：一个允许你指定 Transition 原因的函数。
- [`<Activity>`](/reference/react/Activity)：一个让你隐藏和显示 UI 部分的组件。

## 视图过渡 {/*view-transitions*/}

React View Transitions 是一项新的实验性功能，它让你更容易为应用中的 UI 过渡添加动画。在底层，这些动画使用了大多数现代浏览器中可用的新 [`startViewTransition`](https://developer.mozilla.org/en-US/docs/Web/API/Document/startViewTransition) API。

要让某个元素参与动画，请将其包裹在新的 `<ViewTransition>` 组件中：

```js
// 要动画的“什么”。
<ViewTransition>
  <div>animate me</div>
</ViewTransition>
```

这个新组件让你可以以声明式方式定义在动画被激活时要动画的“什么”。

你可以通过使用以下三种触发器之一来定义 View Transition 的“何时”动画：

```js
// 何时动画。

// Transitions
startTransition(() => setState(...));

// Deferred Values
const deferred = useDeferredValue(value);

// Suspense
<Suspense fallback={<Fallback />}>
  <div>Loading...</div>
</Suspense>
```

默认情况下，这些动画会使用应用于 [View Transitions 的默认 CSS 动画](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API/Using#customizing_your_animations)（通常是平滑的交叉淡入淡出）。你可以使用 [view transition 伪选择器](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API/Using#the_view_transition_pseudo-element_tree) 来定义动画“如何”运行。例如，你可以使用 `*` 来修改所有过渡的默认动画：

```
// 何如动画。
::view-transition-old(*) {
  animation: 300ms ease-out fade-out;
}
::view-transition-new(*) {
  animation: 300ms ease-in fade-in;
}
```

当由于某个动画触发器——例如 `startTransition`、`useDeferredValue`，或者 `Suspense` 的 fallback 切换到内容——导致 DOM 更新时，React 会使用 [声明式启发式规则](/reference/react/ViewTransition#viewtransition) 自动判断哪些 `<ViewTransition>` 组件需要激活动画。然后浏览器会运行 CSS 中定义的动画。

如果你熟悉浏览器的 View Transition API，并想了解 React 如何支持它，请查看文档中的 [`<ViewTransition>` 如何工作](/reference/react/ViewTransition#how-does-viewtransition-work)。

在这篇文章中，让我们看几个如何使用 View Transitions 的示例。

我们先从这个应用开始，它目前不会为以下任何交互添加动画：
- 点击一个视频查看详情。
- 点击“back”返回信息流。
- 在列表中输入以过滤视频。

<Sandpack>

```js src/App.js active
import TalkDetails from './Details'; import Home from './Home'; import {useRouter} from './router';

export default function App() {
  const {url} = useRouter();

  // 🚩此版本还没有包含任何动画
  return url === '/' ? <Home /> : <TalkDetails />;
}
```

```js src/Details.js
import { fetchVideo, fetchVideoDetails } from "./data";
import { Thumbnail, VideoControls } from "./Videos";
import { useRouter } from "./router";
import Layout from "./Layout";
import { use, Suspense } from "react";
import { ChevronLeft } from "./Icons";

function VideoInfo({ id }) {
  const details = use(fetchVideoDetails(id));
  return (
    <>
      <p className="info-title">{details.title}</p>
      <p className="info-description">{details.description}</p>
    </>
  );
}

function VideoInfoFallback() {
  return (
    <>
      <div className="fallback title"></div>
      <div className="fallback description"></div>
    </>
  );
}

export default function Details() {
  const { url, navigateBack } = useRouter();
  const videoId = url.split("/").pop();
  const video = use(fetchVideo(videoId));

  return (
    <Layout
      heading={
        <div
          className="fit back"
          onClick={() => {
            navigateBack("/");
          }}
        >
          <ChevronLeft /> 返回
        </div>
      }
    >
      <div className="details">
        <Thumbnail video={video} large>
          <VideoControls />
        </Thumbnail>
        <Suspense fallback={<VideoInfoFallback />}>
          <VideoInfo id={video.id} />
        </Suspense>
      </div>
    </Layout>
  );
}

```

```js src/Home.js
import { Video } from "./Videos";
import Layout from "./Layout";
import { fetchVideos } from "./data";
import { useId, useState, use } from "react";
import { IconSearch } from "./Icons";

function SearchInput({ value, onChange }) {
  const id = useId();
  return (
    <form className="search" onSubmit={(e) => e.preventDefault()}>
      <label htmlFor={id} className="sr-only">
        搜索
      </label>
      <div className="search-input">
        <div className="search-icon">
          <IconSearch />
        </div>
        <input
          type="text"
          id={id}
          placeholder="搜索"
          value={value}
          onChange={(e) => onChange(e.target.value)}
        />
      </div>
    </form>
  );
}

function filterVideos(videos, query) {
  const keywords = query
    .toLowerCase()
    .split(" ")
    .filter((s) => s !== "");
  if (keywords.length === 0) {
    return videos;
  }
  return videos.filter((video) => {
    const words = (video.title + " " + video.description)
      .toLowerCase()
      .split(" ");
    return keywords.every((kw) => words.some((w) => w.includes(kw)));
  });
}

export default function Home() {
  const videos = use(fetchVideos());
  const count = videos.length;
  const [searchText, setSearchText] = useState("");
  const foundVideos = filterVideos(videos, searchText);
  return (
    <Layout heading={<div className="fit">{count} 个视频</div>}>
      <SearchInput value={searchText} onChange={setSearchText} />
      <div className="video-list">
        {foundVideos.length === 0 && (
          <div className="no-results">无结果</div>
        )}
        <div className="videos">
          {foundVideos.map((video) => (
            <Video key={video.id} video={video} />
          ))}
        </div>
      </div>
    </Layout>
  );
}

```

```js src/Icons.js
export function ChevronLeft() {
  return (
    <svg
      className="chevron-left"
      xmlns="http://www.w3.org/2000/svg"
      width="20"
      height="20"
      viewBox="0 0 20 20">
      <g fill="none" fillRule="evenodd" transform="translate(-446 -398)">
        <path
          fill="currentColor"
          fillRule="nonzero"
          d="M95.8838835,240.366117 C95.3957281,239.877961 94.6042719,239.877961 94.1161165,240.366117 C93.6279612,240.854272 93.6279612,241.645728 94.1161165,242.133883 L98.6161165,246.633883 C99.1042719,247.122039 99.8957281,247.122039 100.383883,246.633883 L104.883883,242.133883 C105.372039,241.645728 105.372039,240.854272 104.883883,240.366117 C104.395728,239.877961 103.604272,239.877961 103.116117,240.366117 L99.5,243.982233 L95.8838835,240.366117 Z"
          transform="translate(356.5 164.5)"
        />
        <polygon points="446 418 466 418 466 398 446 398" />
      </g>
    </svg>
  );
}

export function PauseIcon() {
  return (
    <svg
      className="control-icon"
      style={{padding: '4px'}}
      width="100"
      height="100"
      viewBox="0 0 512 512"
      fill="none"
      xmlns="http://www.w3.org/2000/svg">
      <path
        fillRule="evenodd"
        clipRule="evenodd"
        d="M256 0C114.617 0 0 114.615 0 256s114.617 256 256 256 256-114.615 256-256S397.383 0 256 0zm-32 320c0 8.836-7.164 16-16 16h-32c-8.836 0-16-7.164-16-16V192c0-8.836 7.164-16 16-16h32c8.836 0 16 7.164 16 16v128zm128 0c0 8.836-7.164 16-16 16h-32c-8.836 0-16-7.164-16-16V192c0-8.836 7.164-16 16-16h32c8.836 0 16 7.164 16 16v128z"
        fill="currentColor"
      />
    </svg>
  );
}

export function PlayIcon() {
  return (
    <svg
      className="control-icon"
      width="100"
      height="100"
      viewBox="0 0 72 72"
      fill="none"
      xmlns="http://www.w3.org/2000/svg">
      <path
        fillRule="evenodd"
        clipRule="evenodd"
        d="M36 69C54.2254 69 69 54.2254 69 36C69 17.7746 54.2254 3 36 3C17.7746 3 3 17.7746 3 36C3 54.2254 17.7746 69 36 69ZM52.1716 38.6337L28.4366 51.5801C26.4374 52.6705 24 51.2235 24 48.9464V23.0536C24 20.7764 26.4374 19.3295 28.4366 20.4199L52.1716 33.3663C54.2562 34.5034 54.2562 37.4966 52.1716 38.6337Z"
        fill="currentColor"
      />
    </svg>
  );
}
export function Heart({liked, animate}) {
  return (
    <>
      <svg
        className="absolute overflow-visible"
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg">
        <circle
          className={`circle ${liked ? 'liked' : ''} ${animate ? 'animate' : ''}`}
          cx="12"
          cy="12"
          r="11.5"
          fill="transparent"
          strokeWidth="0"
          stroke="currentColor"
        />
      </svg>

      <svg
        className={`heart ${liked ? 'liked' : ''} ${animate ? 'animate' : ''}`}
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg">
        {liked ? (
          <path
            d="M12 23a.496.496 0 0 1-.26-.074C7.023 19.973 0 13.743 0 8.68c0-4.12 2.322-6.677 6.058-6.677 2.572 0 5.108 2.387 5.134 2.41l.808.771.808-.771C12.834 4.387 15.367 2 17.935 2 21.678 2 24 4.558 24 8.677c0 5.06-7.022 11.293-11.74 14.246a.496.496 0 0 1-.26.074V23z"
            fill="currentColor"
          />
        ) : (
          <path
            fillRule="evenodd"
            clipRule="evenodd"
            d="m12 5.184-.808-.771-.004-.004C11.065 4.299 8.522 2.003 6 2.003c-3.736 0-6 2.558-6 6.677 0 4.47 5.471 9.848 10 13.079.602.43 1.187.82 1.74 1.167A.497.497 0 0 0 12 23v-.003c.09 0 .182-.026.26-.074C16.977 19.97 24 13.737 24 8.677 24 4.557 21.743 2 18 2c-2.569 0-5.166 2.387-5.192 2.413L12 5.184zm-.002 15.525c2.071-1.388 4.477-3.342 6.427-5.47C20.72 12.733 22 10.401 22 8.677c0-1.708-.466-2.855-1.087-3.55C20.316 4.459 19.392 4 18 4c-.726 0-1.63.364-2.5.9-.67.412-1.148.82-1.266.92-.03.025-.037.031-.019.014l-.013.013L12 7.949 9.832 5.88a10.08 10.08 0 0 0-1.33-.977C7.633 4.367 6.728 4.003 6 4.003c-1.388 0-2.312.459-2.91 1.128C2.466 5.826 2 6.974 2 8.68c0 1.726 1.28 4.058 3.575 6.563 1.948 2.127 4.352 4.078 6.423 5.466z"
            fill="currentColor"
          />
        )}
      </svg>
    </>
  );
}

export function IconSearch(props) {
  return (
    <svg width="1em" height="1em" viewBox="0 0 20 20">
      <path
        d="M14.386 14.386l4.0877 4.0877-4.0877-4.0877c-2.9418 2.9419-7.7115 2.9419-10.6533 0-2.9419-2.9418-2.9419-7.7115 0-10.6533 2.9418-2.9419 7.7115-2.9419 10.6533 0 2.9419 2.9418 2.9419 7.7115 0 10.6533z"
        stroke="currentColor"
        fill="none"
        strokeWidth="2"
        fillRule="evenodd"
        strokeLinecap="round"
        strokeLinejoin="round"></path>
    </svg>
  );
}
```

```js src/Layout.js
import {ViewTransition} from 'react'; import { useIsNavPending } from "./router";

export default function Page({ heading, children }) {
  const isPending = useIsNavPending();

  return (
    <div className="page">
      <div className="top">
        <div className="top-nav">
          {heading}
          {isPending && <span className="loader"></span>}
        </div>
      </div>
      {/* 对内容禁用 ViewTransition。 */}
      {/* 内容可以定义自己的 ViewTransition。 */}
      <ViewTransition default="none">
        <div className="bottom">
          <div className="content">{children}</div>
        </div>
      </ViewTransition>
    </div>
  );
}
```

```js src/LikeButton.js hidden
import {useState} from 'react';
import {Heart} from './Icons';

// 由于我们实际上没有后端，这是一个变通方案。
// 与本地状态不同，它可以在视频被过滤后仍然保留。
const likedVideos = new Set();

export default function LikeButton({video}) {
  const [isLiked, setIsLiked] = useState(() => likedVideos.has(video.id));
  const [animate, setAnimate] = useState(false);
  return (
    <button
      className={`like-button ${isLiked && 'liked'}`}
      aria-label={isLiked ? 'Unsave' : 'Save'}
      onClick={() => {
        const nextIsLiked = !isLiked;
        if (nextIsLiked) {
          likedVideos.add(video.id);
        } else {
          likedVideos.delete(video.id);
        }
        setAnimate(true);
        setIsLiked(nextIsLiked);
      }}>
      <Heart liked={isLiked} animate={animate} />
    </button>
  );
}
```

```js src/Videos.js hidden
import { useState } from "react";
import LikeButton from "./LikeButton";
import { useRouter } from "./router";
import { PauseIcon, PlayIcon } from "./Icons";
import { startTransition } from "react";

export function VideoControls() {
  const [isPlaying, setIsPlaying] = useState(false);

  return (
    <span
      className="controls"
      onClick={() =>
        startTransition(() => {
          setIsPlaying((p) => !p);
        })
      }
    >
      {isPlaying ? <PauseIcon /> : <PlayIcon />}
    </span>
  );
}

export function Thumbnail({ video, children }) {
  return (
    <div
      aria-hidden="true"
      tabIndex={-1}
      className={`thumbnail ${video.image}`}
    >
      {children}
    </div>
  );
}

export function Video({ video }) {
  const { navigate } = useRouter();

  return (
    <div className="video">
      <div
        className="link"
        onClick={(e) => {
          e.preventDefault();
          navigate(`/video/${video.id}`);
        }}
      >
        <Thumbnail video={video}></Thumbnail>

        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
      <LikeButton video={video} />
    </div>
  );
}
```


```js src/data.js hidden
const videos = [
  {
    id: '1',
    title: 'First video',
    description: 'Video description',
    image: 'blue',
  },
  {
    id: '2',
    title: 'Second video',
    description: 'Video description',
    image: 'red',
  },
  {
    id: '3',
    title: 'Third video',
    description: 'Video description',
    image: 'green',
  },
  {
    id: '4',
    title: 'Fourth video',
    description: 'Video description',
    image: 'purple',
  },
  {
    id: '5',
    title: 'Fifth video',
    description: 'Video description',
    image: 'yellow',
  },
  {
    id: '6',
    title: 'Sixth video',
    description: 'Video description',
    image: 'gray',
  },
];

let videosCache = new Map();
let videoCache = new Map();
let videoDetailsCache = new Map();
const VIDEO_DELAY = 1;
const VIDEO_DETAILS_DELAY = 1000;
export function fetchVideos() {
  if (videosCache.has(0)) {
    return videosCache.get(0);
  }
  const promise = new Promise((resolve) => {
    setTimeout(() => {
      resolve(videos);
    }, VIDEO_DELAY);
  });
  videosCache.set(0, promise);
  return promise;
}

export function fetchVideo(id) {
  if (videoCache.has(id)) {
    return videoCache.get(id);
  }
  const promise = new Promise((resolve) => {
    setTimeout(() => {
      resolve(videos.find((video) => video.id === id));
    }, VIDEO_DELAY);
  });
  videoCache.set(id, promise);
  return promise;
}

export function fetchVideoDetails(id) {
  if (videoDetailsCache.has(id)) {
    return videoDetailsCache.get(id);
  }
  const promise = new Promise((resolve) => {
    setTimeout(() => {
      resolve(videos.find((video) => video.id === id));
    }, VIDEO_DETAILS_DELAY);
  });
  videoDetailsCache.set(id, promise);
  return promise;
}
```

```js src/router.js
import {useState, createContext,use,useTransition,useLayoutEffect,useEffect} from "react";

export function Router({ children }) {
  const [isPending, startTransition] = useTransition();

  function navigate(url) {
    // 在 transition 中更新路由状态。
    startTransition(() => {
      go(url);
    });
  }




  const [routerState, setRouterState] = useState({
    pendingNav: () => {},
    url: document.location.pathname,
  });


  function go(url) {
    setRouterState({
      url,
      pendingNav() {
        window.history.pushState({}, "", url);
      },
    });
  }


  function navigateBack(url) {
    startTransition(() => {
      go(url);
    });
  }

  useEffect(() => {
    function handlePopState() {
      // 这不应该产生动画，因为恢复必须是同步的。
      // 即使它是一个 transition。
      startTransition(() => {
        setRouterState({
          url: document.location.pathname + document.location.search,
          pendingNav() {
            // 无操作。URL 已经更新。
          },
        });
      });
    }
    window.addEventListener("popstate", handlePopState);
    return () => {
      window.removeEventListener("popstate", handlePopState);
    };
  }, []);
  const pendingNav = routerState.pendingNav;
  useLayoutEffect(() => {
    pendingNav();
  }, [pendingNav]);

  return (
    <RouterContext
      value={{
        url: routerState.url,
        navigate,
        navigateBack,
        isPending,
        params: {},
      }}
    >
      {children}
    </RouterContext>
  );
}

const RouterContext = createContext({ url: "/", params: {} });

export function useRouter() {
  return use(RouterContext);
}

export function useIsNavPending() {
  return use(RouterContext).isPending;
}
```

</Sandpack>

<Note>

#### View Transitions 不会取代由 CSS 和 JS 驱动的动画 {/*view-transitions-do-not-replace-css-and-js-driven-animations*/}

View Transitions 适用于导航、展开、打开或重新排序等 UI 过渡。它们并不是用来取代应用中的所有动画的。

在上面的示例应用中，请注意，当你点击“like”按钮以及在 Suspense fallback 闪烁效果中时，已经存在动画了。对于 CSS 动画来说，这是很好的用例，因为它们是在为某个特定元素做动画。

</Note>

### 为导航添加动画 {/*animating-navigations*/}

我们的应用包含一个支持 Suspense 的路由器，并且[页面过渡已经被标记为 Transitions](/reference/react/useTransition#building-a-suspense-enabled-router)，这意味着导航是通过 `startTransition` 执行的：

```js
function navigate(url) {
  startTransition(() => {
    go(url);
  });
}
```

`startTransition` 是一个 View Transition 触发器，因此我们可以添加 `<ViewTransition>` 来为页面之间的切换添加动画：

```js
// 要动画的“什么”
<ViewTransition key={url}>
  {url === '/' ? <Home /> : <TalkDetails />}
</ViewTransition>
```

当 `url` 变化时，`<ViewTransition>` 和新的路由会被渲染。由于 `<ViewTransition>` 是在 `startTransition` 内部更新的，因此 `<ViewTransition>` 会被激活动画。


默认情况下，View Transitions 包含浏览器默认的交叉淡入淡出动画。在我们的示例中加入这一点后，现在每当我们在页面之间导航时，都会有交叉淡入淡出效果：

<Sandpack>

```js src/App.js active
import {ViewTransition} from 'react'; import Details from './Details';
import Home from './Home'; import {useRouter} from './router';

export default function App() {
  const {url} = useRouter();

  // 使用 ViewTransition 在页面之间添加动画。
  // 默认情况下不需要额外的 CSS。
  return (
    <ViewTransition>
      {url === '/' ? <Home /> : <Details />}
    </ViewTransition>
  );
}
```

```js src/Details.js hidden
import { fetchVideo, fetchVideoDetails } from "./data";
import { Thumbnail, VideoControls } from "./Videos";
import { useRouter } from "./router";
import Layout from "./Layout";
import { use, Suspense } from "react";
import { ChevronLeft } from "./Icons";

function VideoInfo({ id }) {
  const details = use(fetchVideoDetails(id));
  return (
    <>
      <p className="info-title">{details.title}</p>
      <p className="info-description">{details.description}</p>
    </>
  );
}

function VideoInfoFallback() {
  return (
    <>
      <div className="fallback title"></div>
      <div className="fallback description"></div>
    </>
  );
}

export default function Details() {
  const { url, navigateBack } = useRouter();
  const videoId = url.split("/").pop();
  const video = use(fetchVideo(videoId));

  return (
    <Layout
      heading={
        <div
          className="fit back"
          onClick={() => {
            navigateBack("/");
          }}
        >
          <ChevronLeft /> 返回
        </div>
      }
    >
      <div className="details">
        <Thumbnail video={video} large>
          <VideoControls />
        </Thumbnail>
        <Suspense fallback={<VideoInfoFallback />}>
          <VideoInfo id={video.id} />
        </Suspense>
      </div>
    </Layout>
  );
}

```

```js src/Home.js hidden
import { Video } from "./Videos";
import Layout from "./Layout";
import { fetchVideos } from "./data";
import { useId, useState, use } from "react";
import { IconSearch } from "./Icons";

function SearchInput({ value, onChange }) {
  const id = useId();
  return (
    <form className="search" onSubmit={(e) => e.preventDefault()}>
      <label htmlFor={id} className="sr-only">
        搜索
      </label>
      <div className="search-input">
        <div className="search-icon">
          <IconSearch />
        </div>
        <input
          type="text"
          id={id}
          placeholder="搜索"
          value={value}
          onChange={(e) => onChange(e.target.value)}
        />
      </div>
    </form>
  );
}

function filterVideos(videos, query) {
  const keywords = query
    .toLowerCase()
    .split(" ")
    .filter((s) => s !== "");
  if (keywords.length === 0) {
    return videos;
  }
  return videos.filter((video) => {
    const words = (video.title + " " + video.description)
      .toLowerCase()
      .split(" ");
    return keywords.every((kw) => words.some((w) => w.includes(kw)));
  });
}

export default function Home() {
  const videos = use(fetchVideos());
  const count = videos.length;
  const [searchText, setSearchText] = useState("");
  const foundVideos = filterVideos(videos, searchText);
  return (
    <Layout heading={<div className="fit">{count} 个视频</div>}>
      <SearchInput value={searchText} onChange={setSearchText} />
      <div className="video-list">
        {foundVideos.length === 0 && (
          <div className="no-results">无结果</div>
        )}
        <div className="videos">
          {foundVideos.map((video) => (
            <Video key={video.id} video={video} />
          ))}
        </div>
      </div>
    </Layout>
  );
}

```

```js src/Icons.js hidden
export function ChevronLeft() {
  return (
    <svg
      className="chevron-left"
      xmlns="http://www.w3.org/2000/svg"
      width="20"
      height="20"
      viewBox="0 0 20 20">
      <g fill="none" fillRule="evenodd" transform="translate(-446 -398)">
        <path
          fill="currentColor"
          fillRule="nonzero"
          d="M95.8838835,240.366117 C95.3957281,239.877961 94.6042719,239.877961 94.1161165,240.366117 C93.6279612,240.854272 93.6279612,241.645728 94.1161165,242.133883 L98.6161165,246.633883 C99.1042719,247.122039 99.8957281,247.122039 100.383883,246.633883 L104.883883,242.133883 C105.372039,241.645728 105.372039,240.854272 104.883883,240.366117 C104.395728,239.877961 103.604272,239.877961 103.116117,240.366117 L99.5,243.982233 L95.8838835,240.366117 Z"
          transform="translate(356.5 164.5)"
        />
        <polygon points="446 418 466 418 466 398 446 398" />
      </g>
    </svg>
  );
}

export function PauseIcon() {
  return (
    <svg
      className="control-icon"
      style={{padding: '4px'}}
      width="100"
      height="100"
      viewBox="0 0 512 512"
      fill="none"
      xmlns="http://www.w3.org/2000/svg">
      <path
        fillRule="evenodd"
        clipRule="evenodd"
        d="M256 0C114.617 0 0 114.615 0 256s114.617 256 256 256 256-114.615 256-256S397.383 0 256 0zm-32 320c0 8.836-7.164 16-16 16h-32c-8.836 0-16-7.164-16-16V192c0-8.836 7.164-16 16-16h32c8.836 0 16 7.164 16 16v128zm128 0c0 8.836-7.164 16-16 16h-32c-8.836 0-16-7.164-16-16V192c0-8.836 7.164-16 16-16h32c8.836 0 16 7.164 16 16v128z"
        fill="currentColor"
      />
    </svg>
  );
}

export function PlayIcon() {
  return (
    <svg
      className="control-icon"
      width="100"
      height="100"
      viewBox="0 0 72 72"
      fill="none"
      xmlns="http://www.w3.org/2000/svg">
      <path
        fillRule="evenodd"
        clipRule="evenodd"
        d="M36 69C54.2254 69 69 54.2254 69 36C69 17.7746 54.2254 3 36 3C17.7746 3 3 17.7746 3 36C3 54.2254 17.7746 69 36 69ZM52.1716 38.6337L28.4366 51.5801C26.4374 52.6705 24 51.2235 24 48.9464V23.0536C24 20.7764 26.4374 19.3295 28.4366 20.4199L52.1716 33.3663C54.2562 34.5034 54.2562 37.4966 52.1716 38.6337Z"
        fill="currentColor"
      />
    </svg>
  );
}
export function Heart({liked, animate}) {
  return (
    <>
      <svg
        className="absolute overflow-visible"
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg">
        <circle
          className={`circle ${liked ? 'liked' : ''} ${animate ? 'animate' : ''}`}
          cx="12"
          cy="12"
          r="11.5"
          fill="transparent"
          strokeWidth="0"
          stroke="currentColor"
        />
      </svg>

      <svg
        className={`heart ${liked ? 'liked' : ''} ${animate ? 'animate' : ''}`}
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg">
        {liked ? (
          <path
            d="M12 23a.496.496 0 0 1-.26-.074C7.023 19.973 0 13.743 0 8.68c0-4.12 2.322-6.677 6.058-6.677 2.572 0 5.108 2.387 5.134 2.41l.808.771.808-.771C12.834 4.387 15.367 2 17.935 2 21.678 2 24 4.558 24 8.677c0 5.06-7.022 11.293-11.74 14.246a.496.496 0 0 1-.26.074V23z"
            fill="currentColor"
          />
        ) : (
          <path
            fillRule="evenodd"
            clipRule="evenodd"
            d="m12 5.184-.808-.771-.004-.004C11.065 4.299 8.522 2.003 6 2.003c-3.736 0-6 2.558-6 6.677 0 4.47 5.471 9.848 10 13.079.602.43 1.187.82 1.74 1.167A.497.497 0 0 0 12 23v-.003c.09 0 .182-.026.26-.074C16.977 19.97 24 13.737 24 8.677 24 4.557 21.743 2 18 2c-2.569 0-5.166 2.387-5.192 2.413L12 5.184zm-.002 15.525c2.071-1.388 4.477-3.342 6.427-5.47C20.72 12.733 22 10.401 22 8.677c0-1.708-.466-2.855-1.087-3.55C20.316 4.459 19.392 4 18 4c-.726 0-1.63.364-2.5.9-.67.412-1.148.82-1.266.92-.03.025-.037.031-.019.014l-.013.013L12 7.949 9.832 5.88a10.08 10.08 0 0 0-1.33-.977C7.633 4.367 6.728 4.003 6 4.003c-1.388 0-2.312.459-2.91 1.128C2.466 5.826 2 6.974 2 8.68c0 1.726 1.28 4.058 3.575 6.563 1.948 2.127 4.352 4.078 6.423 5.466z"
            fill="currentColor"
          />
        )}
      </svg>
    </>
  );
}

export function IconSearch(props) {
  return (
    <svg width="1em" height="1em" viewBox="0 0 20 20">
      <path
        d="M14.386 14.386l4.0877 4.0877-4.0877-4.0877c-2.9418 2.9419-7.7115 2.9419-10.6533 0-2.9419-2.9418-2.9419-7.7115 0-10.6533 2.9418-2.9419 7.7115-2.9419 10.6533 0 2.9419 2.9418 2.9419 7.7115 0 10.6533z"
        stroke="currentColor"
        fill="none"
        strokeWidth="2"
        fillRule="evenodd"
        strokeLinecap="round"
        strokeLinejoin="round"></path>
    </svg>
  );
}
```

```js src/Layout.js
import {ViewTransition} from 'react'; import { useIsNavPending } from "./router";

export default function Page({ heading, children }) {
  const isPending = useIsNavPending();

  return (
    <div className="page">
      <div className="top">
        <div className="top-nav">
          {heading}
          {isPending && <span className="loader"></span>}
        </div>
      </div>
      {/* 对内容禁用 ViewTransition。 */}
      {/* 内容可以定义自己的 ViewTransition。 */}
      <ViewTransition default="none">
        <div className="bottom">
          <div className="content">{children}</div>
        </div>
      </ViewTransition>
    </div>
  );
}
```

```js src/LikeButton.js hidden
import {useState} from 'react';
import {Heart} from './Icons';

// 由于我们实际上没有后端，这是一个变通方案。
// 与本地状态不同，它可以在视频被过滤后仍然保留。
const likedVideos = new Set();

export default function LikeButton({video}) {
  const [isLiked, setIsLiked] = useState(() => likedVideos.has(video.id));
  const [animate, setAnimate] = useState(false);
  return (
    <button
      className={`like-button ${isLiked && 'liked'}`}
      aria-label={isLiked ? 'Unsave' : 'Save'}
      onClick={() => {
        const nextIsLiked = !isLiked;
        if (nextIsLiked) {
          likedVideos.add(video.id);
        } else {
          likedVideos.delete(video.id);
        }
        setAnimate(true);
        setIsLiked(nextIsLiked);
      }}>
      <Heart liked={isLiked} animate={animate} />
    </button>
  );
}
```

```js src/Videos.js hidden
import { useState } from "react";
import LikeButton from "./LikeButton";
import { useRouter } from "./router";
import { PauseIcon, PlayIcon } from "./Icons";
import { startTransition } from "react";

export function VideoControls() {
  const [isPlaying, setIsPlaying] = useState(false);

  return (
    <span
      className="controls"
      onClick={() =>
        startTransition(() => {
          setIsPlaying((p) => !p);
        })
      }
    >
      {isPlaying ? <PauseIcon /> : <PlayIcon />}
    </span>
  );
}

export function Thumbnail({ video, children }) {
  return (
    <div
      aria-hidden="true"
      tabIndex={-1}
      className={`thumbnail ${video.image}`}
    >
      {children}
    </div>
  );
}

export function Video({ video }) {
  const { navigate } = useRouter();

  return (
    <div className="video">
      <div
        className="link"
        onClick={(e) => {
          e.preventDefault();
          navigate(`/video/${video.id}`);
        }}
      >
        <Thumbnail video={video}></Thumbnail>

        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
      <LikeButton video={video} />
    </div>
  );
}
```


```js src/data.js hidden
const videos = [
  {
    id: '1',
    title: 'First video',
    description: 'Video description',
    image: 'blue',
  },
  {
    id: '2',
    title: 'Second video',
    description: 'Video description',
    image: 'red',
  },
  {
    id: '3',
    title: 'Third video',
    description: 'Video description',
    image: 'green',
  },
  {
    id: '4',
    title: 'Fourth video',
    description: 'Video description',
    image: 'purple',
  },
  {
    id: '5',
    title: 'Fifth video',
    description: 'Video description',
    image: 'yellow',
  },
  {
    id: '6',
    title: 'Sixth video',
    description: 'Video description',
    image: 'gray',
  },
];

let videosCache = new Map();
let videoCache = new Map();
let videoDetailsCache = new Map();
const VIDEO_DELAY = 1;
const VIDEO_DETAILS_DELAY = 1000;
export function fetchVideos() {
  if (videosCache.has(0)) {
    return videosCache.get(0);
  }
  const promise = new Promise((resolve) => {
    setTimeout(() => {
      resolve(videos);
    }, VIDEO_DELAY);
  });
  videosCache.set(0, promise);
  return promise;
}

export function fetchVideo(id) {
  if (videoCache.has(id)) {
    return videoCache.get(id);
  }
  const promise = new Promise((resolve) => {
    setTimeout(() => {
      resolve(videos.find((video) => video.id === id));
    }, VIDEO_DELAY);
  });
  videoCache.set(id, promise);
  return promise;
}

export function fetchVideoDetails(id) {
  if (videoDetailsCache.has(id)) {
    return videoDetailsCache.get(id);
  }
  const promise = new Promise((resolve) => {
    setTimeout(() => {
      resolve(videos.find((video) => video.id === id));
    }, VIDEO_DETAILS_DELAY);
  });
  videoDetailsCache.set(id, promise);
  return promise;
}
```

```js src/router.js
import {useState, createContext,use,useTransition,useLayoutEffect,useEffect} from "react";

export function Router({ children }) {
  const [isPending, startTransition] = useTransition();

  function navigate(url) {
    // 在 transition 中更新路由状态。
    startTransition(() => {
      go(url);
    });
  }




  const [routerState, setRouterState] = useState({
    pendingNav: () => {},
    url: document.location.pathname,
  });


  function go(url) {
    setRouterState({
      url,
      pendingNav() {
        window.history.pushState({}, "", url);
      },
    });
  }


  function navigateBack(url) {
    startTransition(() => {
      go(url);
    });
  }

  useEffect(() => {
    function handlePopState() {
      // 这不应该产生动画，因为恢复必须是同步的。
      // 即使它是一个 transition。
      startTransition(() => {
        setRouterState({
          url: document.location.pathname + document.location.search,
          pendingNav() {
            // 无操作。URL 已经更新。
          },
        });
      });
    }
    window.addEventListener("popstate", handlePopState);
    return () => {
      window.removeEventListener("popstate", handlePopState);
    };
  }, []);
  const pendingNav = routerState.pendingNav;
  useLayoutEffect(() => {
    pendingNav();
  }, [pendingNav]);

  return (
    <RouterContext
      value={{
        url: routerState.url,
        navigate,
        navigateBack,
        isPending,
        params: {},
      }}
    >
      {children}
    </RouterContext>
  );
}

const RouterContext = createContext({ url: "/", params: {} });

export function useRouter() {
  return use(RouterContext);
}

export function useIsNavPending() {
  return use(RouterContext).isPending;
}
```

```css src/styles.css hidden
@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Rg.woff2) format("woff2");
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Md.woff2) format("woff2");
  font-weight: 500;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Bd.woff2) format("woff2");
  font-weight: 600;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Bd.woff2) format("woff2");
  font-weight: 700;
  font-style: normal;
  font-display: swap;
}

* {
  box-sizing: border-box;
}

html {
  background-image: url(https://react.dev/images/meta-gradient-dark.png);
  background-size: 100%;
  background-position: -100%;
  background-color: rgb(64 71 86);
  background-repeat: no-repeat;
  height: 100%;
  width: 100%;
}

body {
  font-family: Optimistic Text, -apple-system, ui-sans-serif, system-ui, sans-serif, Apple Color Emoji, Segoe UI Emoji, Segoe UI Symbol, Noto Color Emoji;
  padding: 10px 0 10px 0;
  margin: 0;
  display: flex;
  justify-content: center;
}

#root {
  flex: 1 1;
  height: auto;
  background-color: #fff;
  border-radius: 10px;
  max-width: 450px;
  min-height: 600px;
  padding-bottom: 10px;
}

h1 {
  margin-top: 0;
  font-size: 22px;
}

h2 {
  margin-top: 0;
  font-size: 20px;
}

h3 {
  margin-top: 0;
  font-size: 18px;
}

h4 {
  margin-top: 0;
  font-size: 16px;
}

h5 {
  margin-top: 0;
  font-size: 14px;
}

h6 {
  margin-top: 0;
  font-size: 12px;
}

code {
  font-size: 1.2em;
}

ul {
  padding-inline-start: 20px;
}

.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}

.absolute {
  position: absolute;
}

.overflow-visible {
  overflow: visible;
}

.visible {
  overflow: visible;
}

.fit {
  width: fit-content;
}


/* 布局 */
.page {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.top-hero {
  height: 200px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-image: conic-gradient(
      from 90deg at -10% 100%,
      #2b303b 0deg,
      #2b303b 90deg,
      #16181d 1turn
  );
}

.bottom {
  flex: 1;
  overflow: auto;
}

.top-nav {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 0;
  padding: 0 12px;
  top: 0;
  width: 100%;
  height: 44px;
  color: #23272f;
  font-weight: 700;
  font-size: 20px;
  z-index: 100;
  cursor: default;
}

.content {
  padding: 0 12px;
  margin-top: 4px;
}


.loader {
  color: #23272f;
  font-size: 3px;
  width: 1em;
  margin-right: 18px;
  height: 1em;
  border-radius: 50%;
  position: relative;
  text-indent: -9999em;
  animation: loading-spinner 1.3s infinite linear;
  animation-delay: 200ms;
  transform: translateZ(0);
}

@keyframes loading-spinner {
  0%,
  100% {
    box-shadow: 0 -3em 0 0.2em,
    2em -2em 0 0em, 3em 0 0 -1em,
    2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 -1em, -3em 0 0 -1em,
    -2em -2em 0 0;
  }
  12.5% {
    box-shadow: 0 -3em 0 0, 2em -2em 0 0.2em,
    3em 0 0 0, 2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 -1em, -3em 0 0 -1em,
    -2em -2em 0 -1em;
  }
  25% {
    box-shadow: 0 -3em 0 -0.5em,
    2em -2em 0 0, 3em 0 0 0.2em,
    2em 2em 0 0, 0 3em 0 -1em,
    -2em 2em 0 -1em, -3em 0 0 -1em,
    -2em -2em 0 -1em;
  }
  37.5% {
    box-shadow: 0 -3em 0 -1em, 2em -2em 0 -1em,
    3em 0em 0 0, 2em 2em 0 0.2em, 0 3em 0 0em,
    -2em 2em 0 -1em, -3em 0em 0 -1em, -2em -2em 0 -1em;
  }
  50% {
    box-shadow: 0 -3em 0 -1em, 2em -2em 0 -1em,
    3em 0 0 -1em, 2em 2em 0 0em, 0 3em 0 0.2em,
    -2em 2em 0 0, -3em 0em 0 -1em, -2em -2em 0 -1em;
  }
  62.5% {
    box-shadow: 0 -3em 0 -1em, 2em -2em 0 -1em,
    3em 0 0 -1em, 2em 2em 0 -1em, 0 3em 0 0,
    -2em 2em 0 0.2em, -3em 0 0 0, -2em -2em 0 -1em;
  }
  75% {
    box-shadow: 0em -3em 0 -1em, 2em -2em 0 -1em,
    3em 0em 0 -1em, 2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 0, -3em 0em 0 0.2em, -2em -2em 0 0;
  }
  87.5% {
    box-shadow: 0em -3em 0 0, 2em -2em 0 -1em,
    3em 0 0 -1em, 2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 0, -3em 0em 0 0, -2em -2em 0 0.2em;
  }
}

/* LikeButton */
.like-button {
  outline-offset: 2px;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 2.5rem;
  height: 2.5rem;
  cursor: pointer;
  border-radius: 9999px;
  border: none;
  outline: none 2px;
  color: #5e687e;
  background: none;
}

.like-button:focus {
  color: #a6423a;
  background-color: rgba(166, 66, 58, .05);
}

.like-button:active {
  color: #a6423a;
  background-color: rgba(166, 66, 58, .05);
  transform: scaleX(0.95) scaleY(0.95);
}

.like-button:hover {
  background-color: #f6f7f9;
}

.like-button.liked {
  color: #a6423a;
}

/* 图标 */
@keyframes circle {
  0% {
    transform: scale(0);
    stroke-width: 16px;
  }

  50% {
    transform: scale(.5);
    stroke-width: 16px;
  }

  to {
    transform: scale(1);
    stroke-width: 0;
  }
}

.circle {
  color: rgba(166, 66, 58, .5);
  transform-origin: center;
  transition-property: all;
  transition-duration: .15s;
  transition-timing-function: cubic-bezier(.4,0,.2,1);
}

.circle.liked.animate {
  animation: circle .3s forwards;
}

.heart {
  width: 1.5rem;
  height: 1.5rem;
}

.heart.liked {
  transform-origin: center;
  transition-property: all;
  transition-duration: .15s;
  transition-timing-function: cubic-bezier(.4, 0, .2, 1);
}

.heart.liked.animate {
  animation: scale .35s ease-in-out forwards;
}

.control-icon {
  color: hsla(0, 0%, 100%, .5);
  filter:  drop-shadow(0 20px 13px rgba(0, 0, 0, .03)) drop-shadow(0 8px 5px rgba(0, 0, 0, .08));
}

.chevron-left {
  margin-top: 2px;
  rotate: 90deg;
}


/* 视频 */
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}

.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}

.thumbnail.red {
  background-image: conic-gradient(at top right, #c76a15, #a6423a, #2b3491);
}

.thumbnail.green {
  background-image: conic-gradient(at top right, #c76a15, #388f7f, #2b3491);
}

.thumbnail.purple {
  background-image: conic-gradient(at top right, #c76a15, #575fb7, #2b3491);
}

.thumbnail.yellow {
  background-image: conic-gradient(at top right, #c76a15, #FABD62, #2b3491);
}

.thumbnail.gray {
  background-image: conic-gradient(at top right, #c76a15, #4E5769, #2b3491);
}

.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
}

.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
  cursor: pointer;
}

.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}

.video .info:hover {
  text-decoration: underline;
}

.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}

.video-description {
  color: #5e687e;
  font-size: 13px;
}

/* 详情 */
.details .thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 100%;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}

.video-details-title {
  margin-top: 8px;
}

.video-details-speaker {
  display: flex;
  gap: 8px;
  margin-top: 10px
}

.back {
  display: flex;
  align-items: center;
  margin-left: -5px;
  cursor: pointer;
}

.back:hover {
  text-decoration: underline;
}

.info-title {
  font-size: 1.5rem;
  font-weight: 700;
  line-height: 1.25;
  margin: 8px 0 0 0 ;
}

.info-description {
  margin: 8px 0 0 0;
}

.controls {
  cursor: pointer;
}

.fallback {
  background: #f6f7f8 linear-gradient(to right, #e6e6e6 5%, #cccccc 25%, #e6e6e6 35%) no-repeat;
  background-size: 800px 104px;
  display: block;
  line-height: 1.25;
  margin: 8px 0 0 0;
  border-radius: 5px;
  overflow: hidden;

  animation: 1s linear 1s infinite shimmer;
  animation-delay: 300ms;
  animation-duration: 1s;
  animation-fill-mode: forwards;
  animation-iteration-count: infinite;
  animation-name: shimmer;
  animation-timing-function: linear;
}


.fallback.title {
  width: 130px;
  height: 30px;

}

.fallback.description {
  width: 150px;
  height: 21px;
}

@keyframes shimmer {
  0% {
    background-position: -468px 0;
  }

  100% {
    background-position: 468px 0;
  }
}

.search {
  margin-bottom: 10px;
}
.search-input {
  width: 100%;
  position: relative;
}

.search-icon {
  position: absolute;
  top: 0;
  bottom: 0;
  inset-inline-start: 0;
  display: flex;
  align-items: center;
  padding-inline-start: 1rem;
  pointer-events: none;
  color: #99a1b3;
}

.search-input input {
  display: flex;
  padding-inline-start: 2.75rem;
  padding-top: 10px;
  padding-bottom: 10px;
  width: 100%;
  text-align: start;
  background-color: rgb(235 236 240);
  outline: 2px solid transparent;
  cursor: pointer;
  border: none;
  align-items: center;
  color: rgb(35 39 47);
  border-radius: 9999px;
  vertical-align: middle;
  font-size: 15px;
}

.search-input input:hover, .search-input input:active {
  background-color: rgb(235 236 240/ 0.8);
  color: rgb(35 39 47/ 0.8);
}

/* 首页 */
.video-list {
  position: relative;
}

.video-list .videos {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  overflow-y: auto;
  height: 100%;
}
```

```js src/index.js hidden
import React, {StrictMode} from 'react';
import {createRoot} from 'react-dom/client';
import './styles.css';

import App from './App';
import {Router} from './router';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <Router>
      <App />
    </Router>
  </StrictMode>
);
```

```json package.json hidden
{
  "dependencies": {
    "react": "canary",
    "react-dom": "canary",
    "react-scripts": "latest"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

</Sandpack>

由于我们的路由器已经使用 `startTransition` 更新路由，只需这一行改动添加 `<ViewTransition>`，就会启用默认的交叉淡入淡出动画。

如果你想了解它的工作原理，请参阅文档：[`<ViewTransition>` 是如何工作的？](/reference/react/ViewTransition#how-does-viewtransition-work)

<Note>

#### 选择退出 `<ViewTransition>` 动画 {/*opting-out-of-viewtransition-animations*/}

在这个示例中，为了简单起见，我们把应用根节点包裹在 `<ViewTransition>` 中，但这意味着应用中的所有过渡都会被动画化，这可能会导致意料之外的动画。

为了解决这个问题，我们将路由子项包裹在 `"none"` 中，这样每个页面都可以控制自己的动画：

```js
// Layout.js
<ViewTransition default="none">
  {children}
</ViewTransition>
```

在实践中，导航应通过 `"enter"` 和 `"exit"` 属性完成，或者使用过渡类型。

</Note>

### 自定义动画 {/*customizing-animations*/}

默认情况下，`<ViewTransition>` 包含浏览器默认的交叉淡入淡出效果。

要自定义动画，你可以向 `<ViewTransition>` 组件提供属性，根据 [`<ViewTransition>` 的激活方式](/reference/react/ViewTransition#props) 来指定使用哪些动画。

例如，我们可以放慢 `default` 的交叉淡入淡出动画：

```js
<ViewTransition default="slow-fade">
  <Home />
</ViewTransition>
```

并使用 [视图过渡类](/reference/react/ViewTransition#view-transition-class) 在 CSS 中定义 `slow-fade`：

```css
::view-transition-old(.slow-fade) {
    animation-duration: 500ms;
}

::view-transition-new(.slow-fade) {
    animation-duration: 500ms;
}
```

现在，交叉淡入淡出更慢了：

<Sandpack>

```js src/App.js active
import { ViewTransition } from "react";
import Details from "./Details";
import Home from "./Home";
import { useRouter } from "./router";

export default function App() {
  const { url } = useRouter();

  // 定义一个 .slow-fade 的默认动画。
  // 动画定义见 animations.css。
  return (
    <ViewTransition default="slow-fade">
      {url === '/' ? <Home /> : <Details />}
    </ViewTransition>
  );
}
```

```js src/Details.js hidden
import { fetchVideo, fetchVideoDetails } from "./data";
import { Thumbnail, VideoControls } from "./Videos";
import { useRouter } from "./router";
import Layout from "./Layout";
import { use, Suspense } from "react";
import { ChevronLeft } from "./Icons";

function VideoInfo({ id }) {
  const details = use(fetchVideoDetails(id));
  return (
    <>
      <p className="info-title">{details.title}</p>
      <p className="info-description">{details.description}</p>
    </>
  );
}

function VideoInfoFallback() {
  return (
    <>
      <div className="fallback title"></div>
      <div className="fallback description"></div>
    </>
  );
}

export default function Details() {
  const { url, navigateBack } = useRouter();
  const videoId = url.split("/").pop();
  const video = use(fetchVideo(videoId));

  return (
    <Layout
      heading={
        <div
          className="fit back"
          onClick={() => {
            navigateBack("/");
          }}
        >
          <ChevronLeft /> 返回
        </div>
      }
    >
      <div className="details">
        <Thumbnail video={video} large>
          <VideoControls />
        </Thumbnail>
        <Suspense fallback={<VideoInfoFallback />}>
          <VideoInfo id={video.id} />
        </Suspense>
      </div>
    </Layout>
  );
}

```

```js src/Home.js hidden
import { Video } from "./Videos";
import Layout from "./Layout";
import { fetchVideos } from "./data";
import { useId, useState, use } from "react";
import { IconSearch } from "./Icons";

function SearchInput({ value, onChange }) {
  const id = useId();
  return (
    <form className="search" onSubmit={(e) => e.preventDefault()}>
      <label htmlFor={id} className="sr-only">
        Search
      </label>
      <div className="search-input">
        <div className="search-icon">
          <IconSearch />
        </div>
        <input
          type="text"
          id={id}
          placeholder="Search"
          value={value}
          onChange={(e) => onChange(e.target.value)}
        />
      </div>
    </form>
  );
}

function filterVideos(videos, query) {
  const keywords = query
    .toLowerCase()
    .split(" ")
    .filter((s) => s !== "");
  if (keywords.length === 0) {
    return videos;
  }
  return videos.filter((video) => {
    const words = (video.title + " " + video.description)
      .toLowerCase()
      .split(" ");
    return keywords.every((kw) => words.some((w) => w.includes(kw)));
  });
}

export default function Home() {
  const videos = use(fetchVideos());
  const count = videos.length;
  const [searchText, setSearchText] = useState("");
  const foundVideos = filterVideos(videos, searchText);
  return (
    <Layout heading={<div className="fit">{count} Videos</div>}>
      <SearchInput value={searchText} onChange={setSearchText} />
      <div className="video-list">
        {foundVideos.length === 0 && (
          <div className="no-results">No results</div>
        )}
        <div className="videos">
          {foundVideos.map((video) => (
            <Video key={video.id} video={video} />
          ))}
        </div>
      </div>
    </Layout>
  );
}

```

```js src/Icons.js hidden
export function ChevronLeft() {
  return (
    <svg
      className="chevron-left"
      xmlns="http://www.w3.org/2000/svg"
      width="20"
      height="20"
      viewBox="0 0 20 20">
      <g fill="none" fillRule="evenodd" transform="translate(-446 -398)">
        <path
          fill="currentColor"
          fillRule="nonzero"
          d="M95.8838835,240.366117 C95.3957281,239.877961 94.6042719,239.877961 94.1161165,240.366117 C93.6279612,240.854272 93.6279612,241.645728 94.1161165,242.133883 L98.6161165,246.633883 C99.1042719,247.122039 99.8957281,247.122039 100.383883,246.633883 L104.883883,242.133883 C105.372039,241.645728 105.372039,240.854272 104.883883,240.366117 C104.395728,239.877961 103.604272,239.877961 103.116117,240.366117 L99.5,243.982233 L95.8838835,240.366117 Z"
          transform="translate(356.5 164.5)"
        />
        <polygon points="446 418 466 418 466 398 446 398" />
      </g>
    </svg>
  );
}

export function PauseIcon() {
  return (
    <svg
      className="control-icon"
      style={{padding: '4px'}}
      width="100"
      height="100"
      viewBox="0 0 512 512"
      fill="none"
      xmlns="http://www.w3.org/2000/svg">
      <path
        fillRule="evenodd"
        clipRule="evenodd"
        d="M256 0C114.617 0 0 114.615 0 256s114.617 256 256 256 256-114.615 256-256S397.383 0 256 0zm-32 320c0 8.836-7.164 16-16 16h-32c-8.836 0-16-7.164-16-16V192c0-8.836 7.164-16 16-16h32c8.836 0 16 7.164 16 16v128zm128 0c0 8.836-7.164 16-16 16h-32c-8.836 0-16-7.164-16-16V192c0-8.836 7.164-16 16-16h32c8.836 0 16 7.164 16 16v128z"
        fill="currentColor"
      />
    </svg>
  );
}

export function PlayIcon() {
  return (
    <svg
      className="control-icon"
      width="100"
      height="100"
      viewBox="0 0 72 72"
      fill="none"
      xmlns="http://www.w3.org/2000/svg">
      <path
        fillRule="evenodd"
        clipRule="evenodd"
        d="M36 69C54.2254 69 69 54.2254 69 36C69 17.7746 54.2254 3 36 3C17.7746 3 3 17.7746 3 36C3 54.2254 17.7746 69 36 69ZM52.1716 38.6337L28.4366 51.5801C26.4374 52.6705 24 51.2235 24 48.9464V23.0536C24 20.7764 26.4374 19.3295 28.4366 20.4199L52.1716 33.3663C54.2562 34.5034 54.2562 37.4966 52.1716 38.6337Z"
        fill="currentColor"
      />
    </svg>
  );
}
export function Heart({liked, animate}) {
  return (
    <>
      <svg
        className="absolute overflow-visible"
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg">
        <circle
          className={`circle ${liked ? 'liked' : ''} ${animate ? 'animate' : ''}`}
          cx="12"
          cy="12"
          r="11.5"
          fill="transparent"
          strokeWidth="0"
          stroke="currentColor"
        />
      </svg>

      <svg
        className={`heart ${liked ? 'liked' : ''} ${animate ? 'animate' : ''}`}
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg">
        {liked ? (
          <path
            d="M12 23a.496.496 0 0 1-.26-.074C7.023 19.973 0 13.743 0 8.68c0-4.12 2.322-6.677 6.058-6.677 2.572 0 5.108 2.387 5.134 2.41l.808.771.808-.771C12.834 4.387 15.367 2 17.935 2 21.678 2 24 4.558 24 8.677c0 5.06-7.022 11.293-11.74 14.246a.496.496 0 0 1-.26.074V23z"
            fill="currentColor"
          />
        ) : (
          <path
            fillRule="evenodd"
            clipRule="evenodd"
            d="m12 5.184-.808-.771-.004-.004C11.065 4.299 8.522 2.003 6 2.003c-3.736 0-6 2.558-6 6.677 0 4.47 5.471 9.848 10 13.079.602.43 1.187.82 1.74 1.167A.497.497 0 0 0 12 23v-.003c.09 0 .182-.026.26-.074C16.977 19.97 24 13.737 24 8.677 24 4.557 21.743 2 18 2c-2.569 0-5.166 2.387-5.192 2.413L12 5.184zm-.002 15.525c2.071-1.388 4.477-3.342 6.427-5.47C20.72 12.733 22 10.401 22 8.677c0-1.708-.466-2.855-1.087-3.55C20.316 4.459 19.392 4 18 4c-.726 0-1.63.364-2.5.9-.67.412-1.148.82-1.266.92-.03.025-.037.031-.019.014l-.013.013L12 7.949 9.832 5.88a10.08 10.08 0 0 0-1.33-.977C7.633 4.367 6.728 4.003 6 4.003c-1.388 0-2.312.459-2.91 1.128C2.466 5.826 2 6.974 2 8.68c0 1.726 1.28 4.058 3.575 6.563 1.948 2.127 4.352 4.078 6.423 5.466z"
            fill="currentColor"
          />
        )}
      </svg>
    </>
  );
}

export function IconSearch(props) {
  return (
    <svg width="1em" height="1em" viewBox="0 0 20 20">
      <path
        d="M14.386 14.386l4.0877 4.0877-4.0877-4.0877c-2.9418 2.9419-7.7115 2.9419-10.6533 0-2.9419-2.9418-2.9419-7.7115 0-10.6533 2.9418-2.9419 7.7115-2.9419 10.6533 0 2.9419 2.9418 2.9419 7.7115 0 10.6533z"
        stroke="currentColor"
        fill="none"
        strokeWidth="2"
        fillRule="evenodd"
        strokeLinecap="round"
        strokeLinejoin="round"></path>
    </svg>
  );
}
```

```js src/Layout.js hidden
import {ViewTransition} from 'react'; import { useIsNavPending } from "./router";

export default function Page({ heading, children }) {
  const isPending = useIsNavPending();

  return (
    <div className="page">
      <div className="top">
        <div className="top-nav">
          {heading}
          {isPending && <span className="loader"></span>}
        </div>
      </div>
      {/* 内容禁用 ViewTransition。 */}
      {/* 内容可以定义自己的 ViewTransition。 */}
      <ViewTransition default="none">
        <div className="bottom">
          <div className="content">{children}</div>
        </div>
      </ViewTransition>
    </div>
  );
}
```

```js src/LikeButton.js hidden
import {useState} from 'react';
import {Heart} from './Icons';

// 由于我们实际上没有后端，这是一个变通方案。
// 与本地状态不同，这个状态在视频被筛选时仍会保留。
const likedVideos = new Set();

export default function LikeButton({video}) {
  const [isLiked, setIsLiked] = useState(() => likedVideos.has(video.id));
  const [animate, setAnimate] = useState(false);
  return (
    <button
      className={`like-button ${isLiked && 'liked'}`}
      aria-label={isLiked ? '取消保存' : '保存'}
      onClick={() => {
        const nextIsLiked = !isLiked;
        if (nextIsLiked) {
          likedVideos.add(video.id);
        } else {
          likedVideos.delete(video.id);
        }
        setAnimate(true);
        setIsLiked(nextIsLiked);
      }}>
      <Heart liked={isLiked} animate={animate} />
    </button>
  );
}
```

```js src/Videos.js active
import { useState, ViewTransition } from "react"; import LikeButton from "./LikeButton"; import { useRouter } from "./router"; import { PauseIcon, PlayIcon } from "./Icons"; import { startTransition } from "react";

export function Thumbnail({ video, children }) {
  // 添加一个名称，以便使用共享元素过渡进行动画。
  // 这会使用默认动画，不需要额外的 css。
  return (
    <ViewTransition name={`video-${video.id}`}>
      <div
        aria-hidden="true"
        tabIndex={-1}
        className={`thumbnail ${video.image}`}
      >
        {children}
      </div>
    </ViewTransition>
  );
}

export function VideoControls() {
  const [isPlaying, setIsPlaying] = useState(false);

  return (
    <span
      className="controls"
      onClick={() =>
        startTransition(() => {
          setIsPlaying((p) => !p);
        })
      }
    >
      {isPlaying ? <PauseIcon /> : <PlayIcon />}
    </span>
  );
}

export function Video({ video }) {
  const { navigate } = useRouter();

  return (
    <div className="video">
      <div
        className="link"
        onClick={(e) => {
          e.preventDefault();
          navigate(`/video/${video.id}`);
        }}
      >
        <Thumbnail video={video}></Thumbnail>

        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
      <LikeButton video={video} />
    </div>
  );
}
```


```js src/data.js hidden
const videos = [
  {
    id: '1',
    title: '第一条视频',
    description: '视频描述',
    image: 'blue',
  },
  {
    id: '2',
    title: '第二条视频',
    description: '视频描述',
    image: 'red',
  },
  {
    id: '3',
    title: '第三条视频',
    description: '视频描述',
    image: 'green',
  },
  {
    id: '4',
    title: '第四条视频',
    description: '视频描述',
    image: 'purple',
  },
  {
    id: '5',
    title: '第五条视频',
    description: '视频描述',
    image: 'yellow',
  },
  {
    id: '6',
    title: '第六条视频',
    description: '视频描述',
    image: 'gray',
  },
];

let videosCache = new Map();
let videoCache = new Map();
let videoDetailsCache = new Map();
const VIDEO_DELAY = 1;
const VIDEO_DETAILS_DELAY = 1000;
export function fetchVideos() {
  if (videosCache.has(0)) {
    return videosCache.get(0);
  }
  const promise = new Promise((resolve) => {
    setTimeout(() => {
      resolve(videos);
    }, VIDEO_DELAY);
  });
  videosCache.set(0, promise);
  return promise;
}

export function fetchVideo(id) {
  if (videoCache.has(id)) {
    return videoCache.get(id);
  }
  const promise = new Promise((resolve) => {
    setTimeout(() => {
      resolve(videos.find((video) => video.id === id));
    }, VIDEO_DELAY);
  });
  videoCache.set(id, promise);
  return promise;
}

export function fetchVideoDetails(id) {
  if (videoDetailsCache.has(id)) {
    return videoDetailsCache.get(id);
  }
  const promise = new Promise((resolve) => {
    setTimeout(() => {
      resolve(videos.find((video) => video.id === id));
    }, VIDEO_DETAILS_DELAY);
  });
  videoDetailsCache.set(id, promise);
  return promise;
}
```

```js src/router.js hidden
import {
  useState,
  createContext,
  use,
  useTransition,
  useLayoutEffect,
  useEffect,
} from "react";

const RouterContext = createContext({ url: "/", params: {} });

export function useRouter() {
  return use(RouterContext);
}

export function useIsNavPending() {
  return use(RouterContext).isPending;
}

export function Router({ children }) {
  const [routerState, setRouterState] = useState({
    pendingNav: () => {},
    url: document.location.pathname,
  });
  const [isPending, startTransition] = useTransition();

  function go(url) {
    setRouterState({
      url,
      pendingNav() {
        window.history.pushState({}, "", url);
      },
    });
  }
  function navigate(url) {
    // 在过渡中更新路由状态。
    startTransition(() => {
      go(url);
    });
  }

  function navigateBack(url) {
    // 在过渡中更新路由状态。
    startTransition(() => {
      go(url);
    });
  }

  useEffect(() => {
    function handlePopState() {
      // 这不应当产生动画，因为恢复必须是同步的。
      // 即使它是一个过渡。
      startTransition(() => {
        setRouterState({
          url: document.location.pathname + document.location.search,
          pendingNav() {
            // 空操作。URL 已经更新。
          },
        });
      });
    }
    window.addEventListener("popstate", handlePopState);
    return () => {
      window.removeEventListener("popstate", handlePopState);
    };
  }, []);
  const pendingNav = routerState.pendingNav;
  useLayoutEffect(() => {
    pendingNav();
  }, [pendingNav]);

  return (
    <RouterContext
      value={{
        url: routerState.url,
        navigate,
        navigateBack,
        isPending,
        params: {},
      }}
    >
      {children}
    </RouterContext>
  );
}
```

```css src/styles.css hidden
@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Rg.woff2) format("woff2");
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Md.woff2) format("woff2");
  font-weight: 500;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Bd.woff2) format("woff2");
  font-weight: 600;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Bd.woff2) format("woff2");
  font-weight: 700;
  font-style: normal;
  font-display: swap;
}

* {
  box-sizing: border-box;
}

html {
  background-image: url(https://react.dev/images/meta-gradient-dark.png);
  background-size: 100%;
  background-position: -100%;
  background-color: rgb(64 71 86);
  background-repeat: no-repeat;
  height: 100%;
  width: 100%;
}

body {
  font-family: Optimistic Text, -apple-system, ui-sans-serif, system-ui, sans-serif, Apple Color Emoji, Segoe UI Emoji, Segoe UI Symbol, Noto Color Emoji;
  padding: 10px 0 10px 0;
  margin: 0;
  display: flex;
  justify-content: center;
}

#root {
  flex: 1 1;
  height: auto;
  background-color: #fff;
  border-radius: 10px;
  max-width: 450px;
  min-height: 600px;
  padding-bottom: 10px;
}

h1 {
  margin-top: 0;
  font-size: 22px;
}

h2 {
  margin-top: 0;
  font-size: 20px;
}

h3 {
  margin-top: 0;
  font-size: 18px;
}

h4 {
  margin-top: 0;
  font-size: 16px;
}

h5 {
  margin-top: 0;
  font-size: 14px;
}

h6 {
  margin-top: 0;
  font-size: 12px;
}

code {
  font-size: 1.2em;
}

ul {
  padding-inline-start: 20px;
}

.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}

.absolute {
  position: absolute;
}

.overflow-visible {
  overflow: visible;
}

.visible {
  overflow: visible;
}

.fit {
  width: fit-content;
}


/* 布局 */
.page {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.top-hero {
  height: 200px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-image: conic-gradient(
      from 90deg at -10% 100%,
      #2b303b 0deg,
      #2b303b 90deg,
      #16181d 1turn
  );
}

.bottom {
  flex: 1;
  overflow: auto;
}

.top-nav {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 0;
  padding: 0 12px;
  top: 0;
  width: 100%;
  height: 44px;
  color: #23272f;
  font-weight: 700;
  font-size: 20px;
  z-index: 100;
  cursor: default;
}

.content {
  padding: 0 12px;
  margin-top: 4px;
}


.loader {
  color: #23272f;
  font-size: 3px;
  width: 1em;
  margin-right: 18px;
  height: 1em;
  border-radius: 50%;
  position: relative;
  text-indent: -9999em;
  animation: loading-spinner 1.3s infinite linear;
  animation-delay: 200ms;
  transform: translateZ(0);
}

@keyframes loading-spinner {
  0%,
  100% {
    box-shadow: 0 -3em 0 0.2em,
    2em -2em 0 0em, 3em 0 0 -1em,
    2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 -1em, -3em 0 0 -1em,
    -2em -2em 0 0;
  }
  12.5% {
    box-shadow: 0 -3em 0 0, 2em -2em 0 0.2em,
    3em 0 0 0, 2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 -1em, -3em 0 0 -1em,
    -2em -2em 0 -1em;
  }
  25% {
    box-shadow: 0 -3em 0 -0.5em,
    2em -2em 0 0, 3em 0 0 0.2em,
    2em 2em 0 0, 0 3em 0 -1em,
    -2em 2em 0 -1em, -3em 0 0 -1em,
    -2em -2em 0 -1em;
  }
  37.5% {
    box-shadow: 0 -3em 0 -1em, 2em -2em 0 -1em,
    3em 0em 0 0, 2em 2em 0 0.2em, 0 3em 0 0em,
    -2em 2em 0 -1em, -3em 0em 0 -1em, -2em -2em 0 -1em;
  }
  50% {
    box-shadow: 0 -3em 0 -1em, 2em -2em 0 -1em,
    3em 0 0 -1em, 2em 2em 0 0em, 0 3em 0 0.2em,
    -2em 2em 0 0, -3em 0em 0 -1em, -2em -2em 0 -1em;
  }
  62.5% {
    box-shadow: 0 -3em 0 -1em, 2em -2em 0 -1em,
    3em 0 0 -1em, 2em 2em 0 -1em, 0 3em 0 0,
    -2em 2em 0 0.2em, -3em 0 0 0, -2em -2em 0 -1em;
  }
  75% {
    box-shadow: 0em -3em 0 -1em, 2em -2em 0 -1em,
    3em 0em 0 -1em, 2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 0, -3em 0em 0 0.2em, -2em -2em 0 0;
  }
  87.5% {
    box-shadow: 0em -3em 0 0, 2em -2em 0 -1em,
    3em 0 0 -1em, 2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 0, -3em 0em 0 0, -2em -2em 0 0.2em;
  }
}

/* LikeButton */
.like-button {
  outline-offset: 2px;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 2.5rem;
  height: 2.5rem;
  cursor: pointer;
  border-radius: 9999px;
  border: none;
  outline: none 2px;
  color: #5e687e;
  background: none;
}

.like-button:focus {
  color: #a6423a;
  background-color: rgba(166, 66, 58, .05);
}

.like-button:active {
  color: #a6423a;
  background-color: rgba(166, 66, 58, .05);
  transform: scaleX(0.95) scaleY(0.95);
}

.like-button:hover {
  background-color: #f6f7f9;
}

.like-button.liked {
  color: #a6423a;
}

/* 图标 */
@keyframes circle {
  0% {
    transform: scale(0);
    stroke-width: 16px;
  }

  50% {
    transform: scale(.5);
    stroke-width: 16px;
  }

  to {
    transform: scale(1);
    stroke-width: 0;
  }
}

.circle {
  color: rgba(166, 66, 58, .5);
  transform-origin: center;
  transition-property: all;
  transition-duration: .15s;
  transition-timing-function: cubic-bezier(.4,0,.2,1);
}

.circle.liked.animate {
  animation: circle .3s forwards;
}

.heart {
  width: 1.5rem;
  height: 1.5rem;
}

.heart.liked {
  transform-origin: center;
  transition-property: all;
  transition-duration: .15s;
  transition-timing-function: cubic-bezier(.4, 0, .2, 1);
}

.heart.liked.animate {
  animation: scale .35s ease-in-out forwards;
}

.control-icon {
  color: hsla(0, 0%, 100%, .5);
  filter:  drop-shadow(0 20px 13px rgba(0, 0, 0, .03)) drop-shadow(0 8px 5px rgba(0, 0, 0, .08));
}

.chevron-left {
  margin-top: 2px;
  rotate: 90deg;
}


/* 视频 */
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}

.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}

.thumbnail.red {
  background-image: conic-gradient(at top right, #c76a15, #a6423a, #2b3491);
}

.thumbnail.green {
  background-image: conic-gradient(at top right, #c76a15, #388f7f, #2b3491);
}

.thumbnail.purple {
  background-image: conic-gradient(at top right, #c76a15, #575fb7, #2b3491);
}

.thumbnail.yellow {
  background-image: conic-gradient(at top right, #c76a15, #FABD62, #2b3491);
}

.thumbnail.gray {
  background-image: conic-gradient(at top right, #c76a15, #4E5769, #2b3491);
}

.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
}

.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
  cursor: pointer;
}

.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}

.video .info:hover {
  text-decoration: underline;
}

.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}

.video-description {
  color: #5e687e;
  font-size: 13px;
}

/* 详情 */
.details .thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 100%;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}

.video-details-title {
  margin-top: 8px;
}

.video-details-speaker {
  display: flex;
  gap: 8px;
  margin-top: 10px
}

.back {
  display: flex;
  align-items: center;
  margin-left: -5px;
  cursor: pointer;
}

.back:hover {
  text-decoration: underline;
}

.info-title {
  font-size: 1.5rem;
  font-weight: 700;
  line-height: 1.25;
  margin: 8px 0 0 0 ;
}

.info-description {
  margin: 8px 0 0 0;
}

.controls {
  cursor: pointer;
}

.fallback {
  background: #f6f7f8 linear-gradient(to right, #e6e6e6 5%, #cccccc 25%, #e6e6e6 35%) no-repeat;
  background-size: 800px 104px;
  display: block;
  line-height: 1.25;
  margin: 8px 0 0 0;
  border-radius: 5px;
  overflow: hidden;

  animation: 1s linear 1s infinite shimmer;
  animation-delay: 300ms;
  animation-duration: 1s;
  animation-fill-mode: forwards;
  animation-iteration-count: infinite;
  animation-name: shimmer;
  animation-timing-function: linear;
}


.fallback.title {
  width: 130px;
  height: 30px;

}

.fallback.description {
  width: 150px;
  height: 21px;
}

@keyframes shimmer {
  0% {
    background-position: -468px 0;
  }

  100% {
    background-position: 468px 0;
  }
}

.search {
  margin-bottom: 10px;
}
.search-input {
  width: 100%;
  position: relative;
}

.search-icon {
  position: absolute;
  top: 0;
  bottom: 0;
  inset-inline-start: 0;
  display: flex;
  align-items: center;
  padding-inline-start: 1rem;
  pointer-events: none;
  color: #99a1b3;
}

.search-input input {
  display: flex;
  padding-inline-start: 2.75rem;
  padding-top: 10px;
  padding-bottom: 10px;
  width: 100%;
  text-align: start;
  background-color: rgb(235 236 240);
  outline: 2px solid transparent;
  cursor: pointer;
  border: none;
  align-items: center;
  color: rgb(35 39 47);
  border-radius: 9999px;
  vertical-align: middle;
  font-size: 15px;
}

.search-input input:hover, .search-input input:active {
  background-color: rgb(235 236 240/ 0.8);
  color: rgb(35 39 47/ 0.8);
}

/* 首页 */
.video-list {
  position: relative;
}

.video-list .videos {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  overflow-y: auto;
  height: 100%;
}
```


```css src/animations.css
/* 不需要额外的动画 */









/* 下面是之前定义的动画 */





::view-transition-old(.slow-fade) {
    animation-duration: 500ms;
}

::view-transition-new(.slow-fade) {
    animation-duration: 500ms;
}
```

```js src/index.js hidden
import React, {StrictMode} from 'react';
import {createRoot} from 'react-dom/client';
import './styles.css';
import './animations.css';

import App from './App';
import {Router} from './router';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <Router>
      <App />
    </Router>
  </StrictMode>
);
```

```json package.json hidden
{
  "dependencies": {
    "react": "canary",
    "react-dom": "canary",
    "react-scripts": "latest"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

</Sandpack>

默认情况下，React 会自动为每个参与过渡的元素生成唯一的 `name`（参见 [How does `<ViewTransition>` work](/reference/react/ViewTransition#how-does-viewtransition-work)）。当 React 看到一个过渡中带有 `name` 的 `<ViewTransition>` 被移除，同时又添加了一个相同 `name` 的新 `<ViewTransition>` 时，它会激活共享元素过渡。

更多信息请参见 [Animating a Shared Element](/reference/react/ViewTransition#animating-a-shared-element) 的文档。

### 基于原因进行动画 {/*animating-based-on-cause-2*/}

有时，你可能希望元素根据触发方式的不同而以不同方式动画。针对这个用例，我们新增了一个名为 `addTransitionType` 的 API，用于指定过渡的原因：

```js {4,11}
function navigate(url) {
  startTransition(() => {
    // 原因 "nav forward" 的过渡类型
    addTransitionType('nav-forward');
    go(url);
  });
}
function navigateBack(url) {
  startTransition(() => {
    // 原因 "nav backward" 的过渡类型
    addTransitionType('nav-back');
    go(url);
  });
}
```

借助过渡类型，你可以通过 props 为 `<ViewTransition>` 提供自定义动画。让我们为“6 Videos”和“Back”的标题添加共享元素过渡：

```js {4,5}
<ViewTransition
  name="nav"
  share={{
    'nav-forward': 'slide-forward',
    'nav-back': 'slide-back',
  }}>
  {heading}
</ViewTransition>
```

这里我们传入一个 `share` prop，来定义如何根据过渡类型进行动画。当共享过渡由 `nav-forward` 激活时，会应用 `slide-forward` 这个视图过渡类。当它来自 `nav-back` 时，则会激活 `slide-back` 动画。让我们在 CSS 中定义这些动画：

```css
::view-transition-old(.slide-forward) {
    /* 向前滑动时，“旧”页面应该向左滑出。 */
    animation: ...
}

::view-transition-new(.slide-forward) {
    /* 向前滑动时，“新”页面应该从右侧滑入。 */
    animation: ...
}

::view-transition-old(.slide-back) {
    /* 向后滑动时，“旧”页面应该向右滑出。 */
    animation: ...
}

::view-transition-new(.slide-back) {
    /* 向后滑动时，“新”页面应该从左侧滑入。 */
    animation: ...
}
```

现在我们可以根据导航类型，让标题和缩略图一起动画：

<Sandpack>

```js src/App.js hidden
import { ViewTransition } from "react";
import Details from "./Details";
import Home from "./Home";
import { useRouter } from "./router";

export default function App() {
  const { url } = useRouter();

  // 保持我们默认的 slow-fade。
  return (
    <ViewTransition default="slow-fade">
      {url === "/" ? <Home /> : <Details />}
    </ViewTransition>
  );
}
```

```js src/Details.js hidden
import { fetchVideo, fetchVideoDetails } from "./data";
import { Thumbnail, VideoControls } from "./Videos";
import { useRouter } from "./router";
import Layout from "./Layout";
import { use, Suspense } from "react";
import { ChevronLeft } from "./Icons";

function VideoInfo({ id }) {
  const details = use(fetchVideoDetails(id));
  return (
    <>
      <p className="info-title">{details.title}</p>
      <p className="info-description">{details.description}</p>
    </>
  );
}

function VideoInfoFallback() {
  return (
    <>
      <div className="fallback title"></div>
      <div className="fallback description"></div>
    </>
  );
}

export default function Details() {
  const { url, navigateBack } = useRouter();
  const videoId = url.split("/").pop();
  const video = use(fetchVideo(videoId));

  return (
    <Layout
      heading={
        <div
          className="fit back"
          onClick={() => {
            navigateBack("/");
          }}
        >
          <ChevronLeft /> Back
        </div>
      }
    >
      <div className="details">
        <Thumbnail video={video} large>
          <VideoControls />
        </Thumbnail>
        <Suspense fallback={<VideoInfoFallback />}>
          <VideoInfo id={video.id} />
        </Suspense>
      </div>
    </Layout>
  );
}

```

```js src/Home.js hidden
import { Video } from "./Videos";
import Layout from "./Layout";
import { fetchVideos } from "./data";
import { useId, useState, use } from "react";
import { IconSearch } from "./Icons";

function SearchInput({ value, onChange }) {
  const id = useId();
  return (
    <form className="search" onSubmit={(e) => e.preventDefault()}>
      <label htmlFor={id} className="sr-only">
        搜索
      </label>
      <div className="search-input">
        <div className="search-icon">
          <IconSearch />
        </div>
        <input
          type="text"
          id={id}
          placeholder="Search"
          value={value}
          onChange={(e) => onChange(e.target.value)}
        />
      </div>
    </form>
  );
}

function filterVideos(videos, query) {
  const keywords = query
    .toLowerCase()
    .split(" ")
    .filter((s) => s !== "");
  if (keywords.length === 0) {
    return videos;
  }
  return videos.filter((video) => {
    const words = (video.title + " " + video.description)
      .toLowerCase()
      .split(" ");
    return keywords.every((kw) => words.some((w) => w.includes(kw)));
  });
}

export default function Home() {
  const videos = use(fetchVideos());
  const count = videos.length;
  const [searchText, setSearchText] = useState("");
  const foundVideos = filterVideos(videos, searchText);
  return (
    <Layout heading={<div className="fit">{count} Videos</div>}>
      <SearchInput value={searchText} onChange={setSearchText} />
      <div className="video-list">
        {foundVideos.length === 0 && (
          <div className="no-results">没有结果</div>
        )}
        <div className="videos">
          {foundVideos.map((video) => (
            <Video key={video.id} video={video} />
          ))}
        </div>
      </div>
    </Layout>
  );
}

```

```js src/Icons.js hidden
export function ChevronLeft() {
  return (
    <svg
      className="chevron-left"
      xmlns="http://www.w3.org/2000/svg"
      width="20"
      height="20"
      viewBox="0 0 20 20">
      <g fill="none" fillRule="evenodd" transform="translate(-446 -398)">
        <path
          fill="currentColor"
          fillRule="nonzero"
          d="M95.8838835,240.366117 C95.3957281,239.877961 94.6042719,239.877961 94.1161165,240.366117 C93.6279612,240.854272 93.6279612,241.645728 94.1161165,242.133883 L98.6161165,246.633883 C99.1042719,247.122039 99.8957281,247.122039 100.383883,246.633883 L104.883883,242.133883 C105.372039,241.645728 105.372039,240.854272 104.883883,240.366117 C104.395728,239.877961 103.604272,239.877961 103.116117,240.366117 L99.5,243.982233 L95.8838835,240.366117 Z"
          transform="translate(356.5 164.5)"
        />
        <polygon points="446 418 466 418 466 398 446 398" />
      </g>
    </svg>
  );
}

export function PauseIcon() {
  return (
    <svg
      className="control-icon"
      style={{padding: '4px'}}
      width="100"
      height="100"
      viewBox="0 0 512 512"
      fill="none"
      xmlns="http://www.w3.org/2000/svg">
      <path
        fillRule="evenodd"
        clipRule="evenodd"
        d="M256 0C114.617 0 0 114.615 0 256s114.617 256 256 256 256-114.615 256-256S397.383 0 256 0zm-32 320c0 8.836-7.164 16-16 16h-32c-8.836 0-16-7.164-16-16V192c0-8.836 7.164-16 16-16h32c8.836 0 16 7.164 16 16v128zm128 0c0 8.836-7.164 16-16 16h-32c-8.836 0-16-7.164-16-16V192c0-8.836 7.164-16 16-16h32c8.836 0 16 7.164 16 16v128z"
        fill="currentColor"
      />
    </svg>
  );
}

export function PlayIcon() {
  return (
    <svg
      className="control-icon"
      width="100"
      height="100"
      viewBox="0 0 72 72"
      fill="none"
      xmlns="http://www.w3.org/2000/svg">
      <path
        fillRule="evenodd"
        clipRule="evenodd"
        d="M36 69C54.2254 69 69 54.2254 69 36C69 17.7746 54.2254 3 36 3C17.7746 3 3 17.7746 3 36C3 54.2254 17.7746 69 36 69ZM52.1716 38.6337L28.4366 51.5801C26.4374 52.6705 24 51.2235 24 48.9464V23.0536C24 20.7764 26.4374 19.3295 28.4366 20.4199L52.1716 33.3663C54.2562 34.5034 54.2562 37.4966 52.1716 38.6337Z"
        fill="currentColor"
      />
    </svg>
  );
}
export function Heart({liked, animate}) {
  return (
    <>
      <svg
        className="absolute overflow-visible"
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg">
        <circle
          className={`circle ${liked ? 'liked' : ''} ${animate ? 'animate' : ''}`}
          cx="12"
          cy="12"
          r="11.5"
          fill="transparent"
          strokeWidth="0"
          stroke="currentColor"
        />
      </svg>

      <svg
        className={`heart ${liked ? 'liked' : ''} ${animate ? 'animate' : ''}`}
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg">
        {liked ? (
          <path
            d="M12 23a.496.496 0 0 1-.26-.074C7.023 19.973 0 13.743 0 8.68c0-4.12 2.322-6.677 6.058-6.677 2.572 0 5.108 2.387 5.134 2.41l.808.771.808-.771C12.834 4.387 15.367 2 17.935 2 21.678 2 24 4.558 24 8.677c0 5.06-7.022 11.293-11.74 14.246a.496.496 0 0 1-.26.074V23z"
            fill="currentColor"
          />
        ) : (
          <path
            fillRule="evenodd"
            clipRule="evenodd"
            d="m12 5.184-.808-.771-.004-.004C11.065 4.299 8.522 2.003 6 2.003c-3.736 0-6 2.558-6 6.677 0 4.47 5.471 9.848 10 13.079.602.43 1.187.82 1.74 1.167A.497.497 0 0 0 12 23v-.003c.09 0 .182-.026.26-.074C16.977 19.97 24 13.737 24 8.677 24 4.557 21.743 2 18 2c-2.569 0-5.166 2.387-5.192 2.413L12 5.184zm-.002 15.525c2.071-1.388 4.477-3.342 6.427-5.47C20.72 12.733 22 10.401 22 8.677c0-1.708-.466-2.855-1.087-3.55C20.316 4.459 19.392 4 18 4c-.726 0-1.63.364-2.5.9-.67.412-1.148.82-1.266.92-.03.025-.037.031-.019.014l-.013.013L12 7.949 9.832 5.88a10.08 10.08 0 0 0-1.33-.977C7.633 4.367 6.728 4.003 6 4.003c-1.388 0-2.312.459-2.91 1.128C2.466 5.826 2 6.974 2 8.68c0 1.726 1.28 4.058 3.575 6.563 1.948 2.127 4.352 4.078 6.423 5.466z"
            fill="currentColor"
          />
        )}
      </svg>
    </>
  );
}

export function IconSearch(props) {
  return (
    <svg width="1em" height="1em" viewBox="0 0 20 20">
      <path
        d="M14.386 14.386l4.0877 4.0877-4.0877-4.0877c-2.9418 2.9419-7.7115 2.9419-10.6533 0-2.9419-2.9418-2.9419-7.7115 0-10.6533 2.9418-2.9419 7.7115-2.9419 10.6533 0 2.9419 2.9418 2.9419 7.7115 0 10.6533z"
        stroke="currentColor"
        fill="none"
        strokeWidth="2"
        fillRule="evenodd"
        strokeLinecap="round"
        strokeLinejoin="round"></path>
    </svg>
  );
}
```

```js src/Layout.js active
import {ViewTransition} from 'react'; import { useIsNavPending } from "./router";

export default function Page({ heading, children }) {
  const isPending = useIsNavPending();
  return (
    <div className="page">
      <div className="top">
        <div className="top-nav">
          {/* 根据过渡类型使用自定义类。 */}
          <ViewTransition
            name="nav"
            share={{
              'nav-forward': 'slide-forward',
              'nav-back': 'slide-back',
            }}>
            {heading}
          </ViewTransition>
          {isPending && <span className="loader"></span>}
        </div>
      </div>
      {/* 对内容禁用 ViewTransition。 */}
      {/* 内容可以自己定义 ViewTransition。 */}
      <ViewTransition default="none">
        <div className="bottom">
          <div className="content">{children}</div>
        </div>
      </ViewTransition>
    </div>
  );
}
```

```js src/LikeButton.js hidden
import {useState} from 'react';
import {Heart} from './Icons';

// 一个变通办法，因为我们实际上没有后端。
// 与本地状态不同，这会在视频被过滤后仍然保留。
const likedVideos = new Set();

export default function LikeButton({video}) {
  const [isLiked, setIsLiked] = useState(() => likedVideos.has(video.id));
  const [animate, setAnimate] = useState(false);
  return (
    <button
      className={`like-button ${isLiked && 'liked'}`}
      aria-label={isLiked ? 'Unsave' : 'Save'}
      onClick={() => {
        const nextIsLiked = !isLiked;
        if (nextIsLiked) {
          likedVideos.add(video.id);
        } else {
          likedVideos.delete(video.id);
        }
        setAnimate(true);
        setIsLiked(nextIsLiked);
      }}>
      <Heart liked={isLiked} animate={animate} />
    </button>
  );
}
```

```js src/Videos.js hidden
import { useState, ViewTransition } from "react";
import LikeButton from "./LikeButton";
import { useRouter } from "./router";
import { PauseIcon, PlayIcon } from "./Icons";
import { startTransition } from "react";

export function Thumbnail({ video, children }) {
  // 添加一个名称，以便使用共享元素过渡进行动画。
  // 这会使用默认动画，不需要额外的 css。
  return (
    <ViewTransition name={`video-${video.id}`}>
      <div
        aria-hidden="true"
        tabIndex={-1}
        className={`thumbnail ${video.image}`}
      >
        {children}
      </div>
    </ViewTransition>
  );
}

export function VideoControls() {
  const [isPlaying, setIsPlaying] = useState(false);

  return (
    <span
      className="controls"
      onClick={() =>
        startTransition(() => {
          setIsPlaying((p) => !p);
        })
      }
    >
      {isPlaying ? <PauseIcon /> : <PlayIcon />}
    </span>
  );
}

export function Video({ video }) {
  const { navigate } = useRouter();

  return (
    <div className="video">
      <div
        className="link"
        onClick={(e) => {
          e.preventDefault();
          navigate(`/video/${video.id}`);
        }}
      >
        <Thumbnail video={video}></Thumbnail>

        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
      <LikeButton video={video} />
    </div>
  );
}
```


```js src/data.js hidden
const videos = [
  {
    id: '1',
    title: '第一条视频',
    description: '视频描述',
    image: 'blue',
  },
  {
    id: '2',
    title: '第二条视频',
    description: '视频描述',
    image: 'red',
  },
  {
    id: '3',
    title: '第三条视频',
    description: '视频描述',
    image: 'green',
  },
  {
    id: '4',
    title: '第四条视频',
    description: '视频描述',
    image: 'purple',
  },
  {
    id: '5',
    title: '第五条视频',
    description: '视频描述',
    image: 'yellow',
  },
  {
    id: '6',
    title: '第六条视频',
    description: '视频描述',
    image: 'gray',
  },
];

let videosCache = new Map();
let videoCache = new Map();
let videoDetailsCache = new Map();
const VIDEO_DELAY = 1;
const VIDEO_DETAILS_DELAY = 1000;
export function fetchVideos() {
  if (videosCache.has(0)) {
    return videosCache.get(0);
  }
  const promise = new Promise((resolve) => {
    setTimeout(() => {
      resolve(videos);
    }, VIDEO_DELAY);
  });
  videosCache.set(0, promise);
  return promise;
}

export function fetchVideo(id) {
  if (videoCache.has(id)) {
    return videoCache.get(id);
  }
  const promise = new Promise((resolve) => {
    setTimeout(() => {
      resolve(videos.find((video) => video.id === id));
    }, VIDEO_DELAY);
  });
  videoCache.set(id, promise);
  return promise;
}

export function fetchVideoDetails(id) {
  if (videoDetailsCache.has(id)) {
    return videoDetailsCache.get(id);
  }
  const promise = new Promise((resolve) => {
    setTimeout(() => {
      resolve(videos.find((video) => video.id === id));
    }, VIDEO_DETAILS_DELAY);
  });
  videoDetailsCache.set(id, promise);
  return promise;
}
```

```js src/router.js
import {useState, createContext, use, useTransition, useLayoutEffect, useEffect, addTransitionType} from "react";

export function Router({ children }) {
  const [isPending, startTransition] = useTransition();

  function navigate(url) {
    startTransition(() => {
      // 过渡类型，对应原因“nav forward”
      addTransitionType('nav-forward');
      go(url);
    });
  }
  function navigateBack(url) {
    startTransition(() => {
      // 过渡类型，对应原因“nav backward”
      addTransitionType('nav-back');
      go(url);
    });
  }


  const [routerState, setRouterState] = useState({pendingNav: () => {}, url: document.location.pathname});

  function go(url) {
    setRouterState({
      url,
      pendingNav() {
        window.history.pushState({}, "", url);
      },
    });
  }

  useEffect(() => {
    function handlePopState() {
      // 这不应当产生动画，因为恢复必须是同步的。
      // 即使它是一个过渡。
      startTransition(() => {
        setRouterState({
          url: document.location.pathname + document.location.search,
          pendingNav() {
            // 空操作。URL 已经更新。
          },
        });
      });
    }
    window.addEventListener("popstate", handlePopState);
    return () => {
      window.removeEventListener("popstate", handlePopState);
    };
  }, []);
  const pendingNav = routerState.pendingNav;
  useLayoutEffect(() => {
    pendingNav();
  }, [pendingNav]);

  return (
    <RouterContext
      value={{
        url: routerState.url,
        navigate,
        navigateBack,
        isPending,
        params: {},
      }}
    >
      {children}
    </RouterContext>
  );
}

const RouterContext = createContext({ url: "/", params: {} });

export function useRouter() {
  return use(RouterContext);
}

export function useIsNavPending() {
  return use(RouterContext).isPending;
}

```

```css src/styles.css hidden
@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Rg.woff2) format("woff2");
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Md.woff2) format("woff2");
  font-weight: 500;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Bd.woff2) format("woff2");
  font-weight: 600;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Bd.woff2) format("woff2");
  font-weight: 700;
  font-style: normal;
  font-display: swap;
}

* {
  box-sizing: border-box;
}

html {
  background-image: url(https://react.dev/images/meta-gradient-dark.png);
  background-size: 100%;
  background-position: -100%;
  background-color: rgb(64 71 86);
  background-repeat: no-repeat;
  height: 100%;
  width: 100%;
}

body {
  font-family: Optimistic Text, -apple-system, ui-sans-serif, system-ui, sans-serif, Apple Color Emoji, Segoe UI Emoji, Segoe UI Symbol, Noto Color Emoji;
  padding: 10px 0 10px 0;
  margin: 0;
  display: flex;
  justify-content: center;
}

#root {
  flex: 1 1;
  height: auto;
  background-color: #fff;
  border-radius: 10px;
  max-width: 450px;
  min-height: 600px;
  padding-bottom: 10px;
}

h1 {
  margin-top: 0;
  font-size: 22px;
}

h2 {
  margin-top: 0;
  font-size: 20px;
}

h3 {
  margin-top: 0;
  font-size: 18px;
}

h4 {
  margin-top: 0;
  font-size: 16px;
}

h5 {
  margin-top: 0;
  font-size: 14px;
}

h6 {
  margin-top: 0;
  font-size: 12px;
}

code {
  font-size: 1.2em;
}

ul {
  padding-inline-start: 20px;
}

.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}

.absolute {
  position: absolute;
}

.overflow-visible {
  overflow: visible;
}

.visible {
  overflow: visible;
}

.fit {
  width: fit-content;
}


/* 布局 */
.page {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.top-hero {
  height: 200px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-image: conic-gradient(
      from 90deg at -10% 100%,
      #2b303b 0deg,
      #2b303b 90deg,
      #16181d 1turn
  );
}

.bottom {
  flex: 1;
  overflow: auto;
}

.top-nav {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 0;
  padding: 0 12px;
  top: 0;
  width: 100%;
  height: 44px;
  color: #23272f;
  font-weight: 700;
  font-size: 20px;
  z-index: 100;
  cursor: default;
}

.content {
  padding: 0 12px;
  margin-top: 4px;
}


.loader {
  color: #23272f;
  font-size: 3px;
  width: 1em;
  margin-right: 18px;
  height: 1em;
  border-radius: 50%;
  position: relative;
  text-indent: -9999em;
  animation: loading-spinner 1.3s infinite linear;
  animation-delay: 200ms;
  transform: translateZ(0);
}

@keyframes loading-spinner {
  0%,
  100% {
    box-shadow: 0 -3em 0 0.2em,
    2em -2em 0 0em, 3em 0 0 -1em,
    2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 -1em, -3em 0 0 -1em,
    -2em -2em 0 0;
  }
  12.5% {
    box-shadow: 0 -3em 0 0, 2em -2em 0 0.2em,
    3em 0 0 0, 2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 -1em, -3em 0 0 -1em,
    -2em -2em 0 -1em;
  }
  25% {
    box-shadow: 0 -3em 0 -0.5em,
    2em -2em 0 0, 3em 0 0 0.2em,
    2em 2em 0 0, 0 3em 0 -1em,
    -2em 2em 0 -1em, -3em 0 0 -1em,
    -2em -2em 0 -1em;
  }
  37.5% {
    box-shadow: 0 -3em 0 -1em, 2em -2em 0 -1em,
    3em 0em 0 0, 2em 2em 0 0.2em, 0 3em 0 0em,
    -2em 2em 0 -1em, -3em 0em 0 -1em, -2em -2em 0 -1em;
  }
  50% {
    box-shadow: 0 -3em 0 -1em, 2em -2em 0 -1em,
    3em 0 0 -1em, 2em 2em 0 0em, 0 3em 0 0.2em,
    -2em 2em 0 0, -3em 0em 0 -1em, -2em -2em 0 -1em;
  }
  62.5% {
    box-shadow: 0 -3em 0 -1em, 2em -2em 0 -1em,
    3em 0 0 -1em, 2em 2em 0 -1em, 0 3em 0 0,
    -2em 2em 0 0.2em, -3em 0 0 0, -2em -2em 0 -1em;
  }
  75% {
    box-shadow: 0em -3em 0 -1em, 2em -2em 0 -1em,
    3em 0em 0 -1em, 2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 0, -3em 0em 0 0.2em, -2em -2em 0 0;
  }
  87.5% {
    box-shadow: 0em -3em 0 0, 2em -2em 0 -1em,
    3em 0 0 -1em, 2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 0, -3em 0em 0 0, -2em -2em 0 0.2em;
  }
}

/* LikeButton */
.like-button {
  outline-offset: 2px;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 2.5rem;
  height: 2.5rem;
  cursor: pointer;
  border-radius: 9999px;
  border: none;
  outline: none 2px;
  color: #5e687e;
  background: none;
}

.like-button:focus {
  color: #a6423a;
  background-color: rgba(166, 66, 58, .05);
}

.like-button:active {
  color: #a6423a;
  background-color: rgba(166, 66, 58, .05);
  transform: scaleX(0.95) scaleY(0.95);
}

.like-button:hover {
  background-color: #f6f7f9;
}

.like-button.liked {
  color: #a6423a;
}

/* 图标 */
@keyframes circle {
  0% {
    transform: scale(0);
    stroke-width: 16px;
  }

  50% {
    transform: scale(.5);
    stroke-width: 16px;
  }

  to {
    transform: scale(1);
    stroke-width: 0;
  }
}

.circle {
  color: rgba(166, 66, 58, .5);
  transform-origin: center;
  transition-property: all;
  transition-duration: .15s;
  transition-timing-function: cubic-bezier(.4,0,.2,1);
}

.circle.liked.animate {
  animation: circle .3s forwards;
}

.heart {
  width: 1.5rem;
  height: 1.5rem;
}

.heart.liked {
  transform-origin: center;
  transition-property: all;
  transition-duration: .15s;
  transition-timing-function: cubic-bezier(.4, 0, .2, 1);
}

.heart.liked.animate {
  animation: scale .35s ease-in-out forwards;
}

.control-icon {
  color: hsla(0, 0%, 100%, .5);
  filter:  drop-shadow(0 20px 13px rgba(0, 0, 0, .03)) drop-shadow(0 8px 5px rgba(0, 0, 0, .08));
}

.chevron-left {
  margin-top: 2px;
  rotate: 90deg;
}


/* 视频 */
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}

.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}

.thumbnail.red {
  background-image: conic-gradient(at top right, #c76a15, #a6423a, #2b3491);
}

.thumbnail.green {
  background-image: conic-gradient(at top right, #c76a15, #388f7f, #2b3491);
}

.thumbnail.purple {
  background-image: conic-gradient(at top right, #c76a15, #575fb7, #2b3491);
}

.thumbnail.yellow {
  background-image: conic-gradient(at top right, #c76a15, #FABD62, #2b3491);
}

.thumbnail.gray {
  background-image: conic-gradient(at top right, #c76a15, #4E5769, #2b3491);
}

.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
}

.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
  cursor: pointer;
}

.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}

.video .info:hover {
  text-decoration: underline;
}

.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}

.video-description {
  color: #5e687e;
  font-size: 13px;
}

/* 详情 */
.details .thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 100%;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}

.video-details-title {
  margin-top: 8px;
}

.video-details-speaker {
  display: flex;
  gap: 8px;
  margin-top: 10px
}

.back {
  display: flex;
  align-items: center;
  margin-left: -5px;
  cursor: pointer;
}

.back:hover {
  text-decoration: underline;
}

.info-title {
  font-size: 1.5rem;
  font-weight: 700;
  line-height: 1.25;
  margin: 8px 0 0 0 ;
}

.info-description {
  margin: 8px 0 0 0;
}

.controls {
  cursor: pointer;
}

.fallback {
  background: #f6f7f8 linear-gradient(to right, #e6e6e6 5%, #cccccc 25%, #e6e6e6 35%) no-repeat;
  background-size: 800px 104px;
  display: block;
  line-height: 1.25;
  margin: 8px 0 0 0;
  border-radius: 5px;
  overflow: hidden;

  animation: 1s linear 1s infinite shimmer;
  animation-delay: 300ms;
  animation-duration: 1s;
  animation-fill-mode: forwards;
  animation-iteration-count: infinite;
  animation-name: shimmer;
  animation-timing-function: linear;
}


.fallback.title {
  width: 130px;
  height: 30px;

}

.fallback.description {
  width: 150px;
  height: 21px;
}

@keyframes shimmer {
  0% {
    background-position: -468px 0;
  }

  100% {
    background-position: 468px 0;
  }
}

.search {
  margin-bottom: 10px;
}
.search-input {
  width: 100%;
  position: relative;
}

.search-icon {
  position: absolute;
  top: 0;
  bottom: 0;
  inset-inline-start: 0;
  display: flex;
  align-items: center;
  padding-inline-start: 1rem;
  pointer-events: none;
  color: #99a1b3;
}

.search-input input {
  display: flex;
  padding-inline-start: 2.75rem;
  padding-top: 10px;
  padding-bottom: 10px;
  width: 100%;
  text-align: start;
  background-color: rgb(235 236 240);
  outline: 2px solid transparent;
  cursor: pointer;
  border: none;
  align-items: center;
  color: rgb(35 39 47);
  border-radius: 9999px;
  vertical-align: middle;
  font-size: 15px;
}

.search-input input:hover, .search-input input:active {
  background-color: rgb(235 236 240/ 0.8);
  color: rgb(35 39 47/ 0.8);
}

/* 首页 */
.video-list {
  position: relative;
}

.video-list .videos {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  overflow-y: auto;
  height: 100%;
}
```


```css src/animations.css
/* 不需要额外的动画 */









/* 之前定义的动画如下 */





::view-transition-old(.slow-fade) {
    animation-duration: 500ms;
}

::view-transition-new(.slow-fade) {
    animation-duration: 500ms;
}
```

```js src/index.js hidden
import React, {StrictMode} from 'react';
import {createRoot} from 'react-dom/client';
import './styles.css';
import './animations.css';

import App from './App';
import {Router} from './router';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <Router>
      <App />
    </Router>
  </StrictMode>
);
```

```json package.json hidden
{
  "dependencies": {
    "react": "canary",
    "react-dom": "canary",
    "react-scripts": "latest"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

</Sandpack>

默认情况下，React 会自动为每个参与过渡的元素生成唯一的 `name`（参见 [How does `<ViewTransition>` work](/reference/react/ViewTransition#how-does-viewtransition-work)）。当 React 看到一个过渡中带有 `name` 的 `<ViewTransition>` 被移除，同时又添加了一个相同 `name` 的新 `<ViewTransition>` 时，它会激活共享元素过渡。

更多信息请参见 [Animating a Shared Element](/reference/react/ViewTransition#animating-a-shared-element) 的文档。

### 基于原因进行动画 {/*animating-based-on-cause*/}

有时，你可能希望元素根据触发方式的不同而以不同方式动画。针对这个用例，我们新增了一个名为 `addTransitionType` 的 API，用于指定过渡的原因：

```js {4,11}
function navigate(url) {
  startTransition(() => {
    // 原因 "nav forward" 的过渡类型
    addTransitionType('nav-forward');
    go(url);
  });
}
function navigateBack(url) {
  startTransition(() => {
    // 原因 "nav backward" 的过渡类型
    addTransitionType('nav-back');
    go(url);
  });
}
```

借助过渡类型，你可以通过 props 为 `<ViewTransition>` 提供自定义动画。让我们为“6 Videos”和“Back”的标题添加共享元素过渡：

```js {4,5}
<ViewTransition
  name="nav"
  share={{
    'nav-forward': 'slide-forward',
    'nav-back': 'slide-back',
  }}>
  {heading}
</ViewTransition>
```

这里我们传入一个 `share` prop，来定义如何根据过渡类型进行动画。当共享过渡由 `nav-forward` 激活时，会应用 `slide-forward` 这个视图过渡类。当它来自 `nav-back` 时，则会激活 `slide-back` 动画。让我们在 CSS 中定义这些动画：

```css
::view-transition-old(.slide-forward) {
    /* 向前滑动时，“旧”页面应该向左滑出。 */
    animation: ...
}

::view-transition-new(.slide-forward) {
    /* 向前滑动时，“新”页面应该从右侧滑入。 */
    animation: ...
}

::view-transition-old(.slide-back) {
    /* 向后滑动时，“旧”页面应该向右滑出。 */
    animation: ...
}

::view-transition-new(.slide-back) {
    /* 向后滑动时，“新”页面应该从左侧滑入。 */
    animation: ...
}
```

现在我们可以根据导航类型，让标题和缩略图一起动画：

<Sandpack>

```js src/App.js hidden
import { ViewTransition } from "react";
import Details from "./Details";
import Home from "./Home";
import { useRouter } from "./router";

export default function App() {
  const { url } = useRouter();

  // 保持我们默认的 slow-fade。
  return (
    <ViewTransition default="slow-fade">
      {url === "/" ? <Home /> : <Details />}
    </ViewTransition>
  );
}
```

```js src/Details.js hidden
import { fetchVideo, fetchVideoDetails } from "./data";
import { Thumbnail, VideoControls } from "./Videos";
import { useRouter } from "./router";
import Layout from "./Layout";
import { use, Suspense } from "react";
import { ChevronLeft } from "./Icons";

function VideoInfo({ id }) {
  const details = use(fetchVideoDetails(id));
  return (
    <>
      <p className="info-title">{details.title}</p>
      <p className="info-description">{details.description}</p>
    </>
  );
}

function VideoInfoFallback() {
  return (
    <>
      <div className="fallback title"></div>
      <div className="fallback description"></div>
    </>
  );
}

export default function Details() {
  const { url, navigateBack } = useRouter();
  const videoId = url.split("/").pop();
  const video = use(fetchVideo(videoId));

  return (
    <Layout
      heading={
        <div
          className="fit back"
          onClick={() => {
            navigateBack("/");
          }}
        >
          <ChevronLeft /> Back
        </div>
      }
    >
      <div className="details">
        <Thumbnail video={video} large>
          <VideoControls />
        </Thumbnail>
        <Suspense fallback={<VideoInfoFallback />}>
          <VideoInfo id={video.id} />
        </Suspense>
      </div>
    </Layout>
  );
}

```

```js src/Home.js hidden
import { Video } from "./Videos";
import Layout from "./Layout";
import { fetchVideos } from "./data";
import { useId, useState, use } from "react";
import { IconSearch } from "./Icons";

function SearchInput({ value, onChange }) {
  const id = useId();
  return (
    <form className="search" onSubmit={(e) => e.preventDefault()}>
      <label htmlFor={id} className="sr-only">
        搜索
      </label>
      <div className="search-input">
        <div className="search-icon">
          <IconSearch />
        </div>
        <input
          type="text"
          id={id}
          placeholder="Search"
          value={value}
          onChange={(e) => onChange(e.target.value)}
        />
      </div>
    </form>
  );
}

function filterVideos(videos, query) {
  const keywords = query
    .toLowerCase()
    .split(" ")
    .filter((s) => s !== "");
  if (keywords.length === 0) {
    return videos;
  }
  return videos.filter((video) => {
    const words = (video.title + " " + video.description)
      .toLowerCase()
      .split(" ");
    return keywords.every((kw) => words.some((w) => w.includes(kw)));
  });
}

export default function Home() {
  const videos = use(fetchVideos());
  const count = videos.length;
  const [searchText, setSearchText] = useState("");
  const foundVideos = filterVideos(videos, searchText);
  return (
    <Layout heading={<div className="fit">{count} Videos</div>}>
      <SearchInput value={searchText} onChange={setSearchText} />
      <div className="video-list">
        {foundVideos.length === 0 && (
          <div className="no-results">没有结果</div>
        )}
        <div className="videos">
          {foundVideos.map((video) => (
            <Video key={video.id} video={video} />
          ))}
        </div>
      </div>
    </Layout>
  );
}

```

```js src/Icons.js hidden
export function ChevronLeft() {
  return (
    <svg
      className="chevron-left"
      xmlns="http://www.w3.org/2000/svg"
      width="20"
      height="20"
      viewBox="0 0 20 20">
      <g fill="none" fillRule="evenodd" transform="translate(-446 -398)">
        <path
          fill="currentColor"
          fillRule="nonzero"
          d="M95.8838835,240.366117 C95.3957281,239.877961 94.6042719,239.877961 94.1161165,240.366117 C93.6279612,240.854272 93.6279612,241.645728 94.1161165,242.133883 L98.6161165,246.633883 C99.1042719,247.122039 99.8957281,247.122039 100.383883,246.633883 L104.883883,242.133883 C105.372039,241.645728 105.372039,240.854272 104.883883,240.366117 C104.395728,239.877961 103.604272,239.877961 103.116117,240.366117 L99.5,243.982233 L95.8838835,240.366117 Z"
          transform="translate(356.5 164.5)"
        />
        <polygon points="446 418 466 418 466 398 446 398" />
      </g>
    </svg>
  );
}

export function PauseIcon() {
  return (
    <svg
      className="control-icon"
      style={{padding: '4px'}}
      width="100"
      height="100"
      viewBox="0 0 512 512"
      fill="none"
      xmlns="http://www.w3.org/2000/svg">
      <path
        fillRule="evenodd"
        clipRule="evenodd"
        d="M256 0C114.617 0 0 114.615 0 256s114.617 256 256 256 256-114.615 256-256S397.383 0 256 0zm-32 320c0 8.836-7.164 16-16 16h-32c-8.836 0-16-7.164-16-16V192c0-8.836 7.164-16 16-16h32c8.836 0 16 7.164 16 16v128zm128 0c0 8.836-7.164 16-16 16h-32c-8.836 0-16-7.164-16-16V192c0-8.836 7.164-16 16-16h32c8.836 0 16 7.164 16 16v128z"
        fill="currentColor"
      />
    </svg>
  );
}

export function PlayIcon() {
  return (
    <svg
      className="control-icon"
      width="100"
      height="100"
      viewBox="0 0 72 72"
      fill="none"
      xmlns="http://www.w3.org/2000/svg">
      <path
        fillRule="evenodd"
        clipRule="evenodd"
        d="M36 69C54.2254 69 69 54.2254 69 36C69 17.7746 54.2254 3 36 3C17.7746 3 3 17.7746 3 36C3 54.2254 17.7746 69 36 69ZM52.1716 38.6337L28.4366 51.5801C26.4374 52.6705 24 51.2235 24 48.9464V23.0536C24 20.7764 26.4374 19.3295 28.4366 20.4199L52.1716 33.3663C54.2562 34.5034 54.2562 37.4966 52.1716 38.6337Z"
        fill="currentColor"
      />
    </svg>
  );
}
export function Heart({liked, animate}) {
  return (
    <>
      <svg
        className="absolute overflow-visible"
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg">
        <circle
          className={`circle ${liked ? 'liked' : ''} ${animate ? 'animate' : ''}`}
          cx="12"
          cy="12"
          r="11.5"
          fill="transparent"
          strokeWidth="0"
          stroke="currentColor"
        />
      </svg>

      <svg
        className={`heart ${liked ? 'liked' : ''} ${animate ? 'animate' : ''}`}
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg">
        {liked ? (
          <path
            d="M12 23a.496.496 0 0 1-.26-.074C7.023 19.973 0 13.743 0 8.68c0-4.12 2.322-6.677 6.058-6.677 2.572 0 5.108 2.387 5.134 2.41l.808.771.808-.771C12.834 4.387 15.367 2 17.935 2 21.678 2 24 4.558 24 8.677c0 5.06-7.022 11.293-11.74 14.246a.496.496 0 0 1-.26.074V23z"
            fill="currentColor"
          />
        ) : (
          <path
            fillRule="evenodd"
            clipRule="evenodd"
            d="m12 5.184-.808-.771-.004-.004C11.065 4.299 8.522 2.003 6 2.003c-3.736 0-6 2.558-6 6.677 0 4.47 5.471 9.848 10 13.079.602.43 1.187.82 1.74 1.167A.497.497 0 0 0 12 23v-.003c.09 0 .182-.026.26-.074C16.977 19.97 24 13.737 24 8.677 24 4.557 21.743 2 18 2c-2.569 0-5.166 2.387-5.192 2.413L12 5.184zm-.002 15.525c2.071-1.388 4.477-3.342 6.427-5.47C20.72 12.733 22 10.401 22 8.677c0-1.708-.466-2.855-1.087-3.55C20.316 4.459 19.392 4 18 4c-.726 0-1.63.364-2.5.9-.67.412-1.148.82-1.266.92-.03.025-.037.031-.019.014l-.013.013L12 7.949 9.832 5.88a10.08 10.08 0 0 0-1.33-.977C7.633 4.367 6.728 4.003 6 4.003c-1.388 0-2.312.459-2.91 1.128C2.466 5.826 2 6.974 2 8.68c0 1.726 1.28 4.058 3.575 6.563 1.948 2.127 4.352 4.078 6.423 5.466z"
            fill="currentColor"
          />
        )}
      </svg>
    </>
  );
}

export function IconSearch(props) {
  return (
    <svg width="1em" height="1em" viewBox="0 0 20 20">
      <path
        d="M14.386 14.386l4.0877 4.0877-4.0877-4.0877c-2.9418 2.9419-7.7115 2.9419-10.6533 0-2.9419-2.9418-2.9419-7.7115 0-10.6533 2.9418-2.9419 7.7115-2.9419 10.6533 0 2.9419 2.9418 2.9419 7.7115 0 10.6533z"
        stroke="currentColor"
        fill="none"
        strokeWidth="2"
        fillRule="evenodd"
        strokeLinecap="round"
        strokeLinejoin="round"></path>
    </svg>
  );
}
```

```js src/Layout.js active
import {ViewTransition} from 'react'; import { useIsNavPending } from "./router";

export default function Page({ heading, children }) {
  const isPending = useIsNavPending();
  return (
    <div className="page">
      <div className="top">
        <div className="top-nav">
          {/* 根据过渡类型使用自定义类。 */}
          <ViewTransition
            name="nav"
            share={{
              'nav-forward': 'slide-forward',
              'nav-back': 'slide-back',
            }}>
            {heading}
          </ViewTransition>
          {isPending && <span className="loader"></span>}
        </div>
      </div>
      {/* 为内容禁用 ViewTransition。 */}
      {/* 内容可以定义自己的 ViewTransition。 */}
      <ViewTransition default="none">
        <div className="bottom">
          <div className="content">{children}</div>
        </div>
      </ViewTransition>
    </div>
  );
}
```

```js src/LikeButton.js hidden
import {useState} from 'react';
import {Heart} from './Icons';

// 一个变通办法，因为我们实际上没有后端。
// 不同于本地状态，这会在视频被过滤后仍然保留。
const likedVideos = new Set();

export default function LikeButton({video}) {
  const [isLiked, setIsLiked] = useState(() => likedVideos.has(video.id));
  const [animate, setAnimate] = useState(false);
  return (
    <button
      className={`like-button ${isLiked && 'liked'}`}
      aria-label={isLiked ? 'Unsave' : 'Save'}
      onClick={() => {
        const nextIsLiked = !isLiked;
        if (nextIsLiked) {
          likedVideos.add(video.id);
        } else {
          likedVideos.delete(video.id);
        }
        setAnimate(true);
        setIsLiked(nextIsLiked);
      }}>
      <Heart liked={isLiked} animate={animate} />
    </button>
  );
}
```

```js src/Videos.js hidden
import { useState, ViewTransition } from "react";
import LikeButton from "./LikeButton";
import { useRouter } from "./router";
import { PauseIcon, PlayIcon } from "./Icons";
import { startTransition } from "react";

export function Thumbnail({ video, children }) {
  // 添加一个名称，以便使用共享元素过渡进行动画。
  // 这会使用默认动画，不需要额外的 css。
  return (
    <ViewTransition name={`video-${video.id}`}>
      <div
        aria-hidden="true"
        tabIndex={-1}
        className={`thumbnail ${video.image}`}
      >
        {children}
      </div>
    </ViewTransition>
  );
}

export function VideoControls() {
  const [isPlaying, setIsPlaying] = useState(false);

  return (
    <span
      className="controls"
      onClick={() =>
        startTransition(() => {
          setIsPlaying((p) => !p);
        })
      }
    >
      {isPlaying ? <PauseIcon /> : <PlayIcon />}
    </span>
  );
}

export function Video({ video }) {
  const { navigate } = useRouter();

  return (
    <div className="video">
      <div
        className="link"
        onClick={(e) => {
          e.preventDefault();
          navigate(`/video/${video.id}`);
        }}
      >
        <Thumbnail video={video}></Thumbnail>

        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
      <LikeButton video={video} />
    </div>
  );
}
```

```js src/Home.js hidden
import { Video } from "./Videos";
import Layout from "./Layout";
import { fetchVideos } from "./data";
import { useId, useState, use } from "react";
import { IconSearch } from "./Icons";

function SearchInput({ value, onChange }) {
  const id = useId();
  return (
    <form className="search" onSubmit={(e) => e.preventDefault()}>
      <label htmlFor={id} className="sr-only">
        搜索
      </label>
      <div className="search-input">
        <div className="search-icon">
          <IconSearch />
        </div>
        <input
          type="text"
          id={id}
          placeholder="搜索"
          value={value}
          onChange={(e) => onChange(e.target.value)}
        />
      </div>
    </form>
  );
}

function filterVideos(videos, query) {
  const keywords = query
    .toLowerCase()
    .split(" ")
    .filter((s) => s !== "");
  if (keywords.length === 0) {
    return videos;
  }
  return videos.filter((video) => {
    const words = (video.title + " " + video.description)
      .toLowerCase()
      .split(" ");
    return keywords.every((kw) => words.some((w) => w.includes(kw)));
  });
}

export default function Home() {
  const videos = use(fetchVideos());
  const count = videos.length;
  const [searchText, setSearchText] = useState("");
  const foundVideos = filterVideos(videos, searchText);
  return (
    <Layout heading={<div className="fit">{count} 个视频</div>}>
      <SearchInput value={searchText} onChange={setSearchText} />
      <div className="video-list">
        {foundVideos.length === 0 && (
          <div className="no-results">没有结果</div>
        )}
        <div className="videos">
          {foundVideos.map((video) => (
            <Video key={video.id} video={video} />
          ))}
        </div>
      </div>
    </Layout>
  );
}

```

```js src/Icons.js hidden
export function ChevronLeft() {
  return (
    <svg
      className="chevron-left"
      xmlns="http://www.w3.org/2000/svg"
      width="20"
      height="20"
      viewBox="0 0 20 20">
      <g fill="none" fillRule="evenodd" transform="translate(-446 -398)">
        <path
          fill="currentColor"
          fillRule="nonzero"
          d="M95.8838835,240.366117 C95.3957281,239.877961 94.6042719,239.877961 94.1161165,240.366117 C93.6279612,240.854272 93.6279612,241.645728 94.1161165,242.133883 L98.6161165,246.633883 C99.1042719,247.122039 99.8957281,247.122039 100.383883,246.633883 L104.883883,242.133883 C105.372039,241.645728 105.372039,240.854272 104.883883,240.366117 C104.395728,239.877961 103.604272,239.877961 103.116117,240.366117 L99.5,243.982233 L95.8838835,240.366117 Z"
          transform="translate(356.5 164.5)"
        />
        <polygon points="446 418 466 418 466 398 446 398" />
      </g>
    </svg>
  );
}

export function PauseIcon() {
  return (
    <svg
      className="control-icon"
      style={{padding: '4px'}}
      width="100"
      height="100"
      viewBox="0 0 512 512"
      fill="none"
      xmlns="http://www.w3.org/2000/svg">
      <path
        fillRule="evenodd"
        clipRule="evenodd"
        d="M256 0C114.617 0 0 114.615 0 256s114.617 256 256 256 256-114.615 256-256S397.383 0 256 0zm-32 320c0 8.836-7.164 16-16 16h-32c-8.836 0-16-7.164-16-16V192c0-8.836 7.164-16 16-16h32c8.836 0 16 7.164 16 16v128zm128 0c0 8.836-7.164 16-16 16h-32c-8.836 0-16-7.164-16-16V192c0-8.836 7.164-16 16-16h32c8.836 0 16 7.164 16 16v128z"
        fill="currentColor"
      />
    </svg>
  );
}

export function PlayIcon() {
  return (
    <svg
      className="control-icon"
      width="100"
      height="100"
      viewBox="0 0 72 72"
      fill="none"
      xmlns="http://www.w3.org/2000/svg">
      <path
        fillRule="evenodd"
        clipRule="evenodd"
        d="M36 69C54.2254 69 69 54.2254 69 36C69 17.7746 54.2254 3 36 3C17.7746 3 3 17.7746 3 36C3 54.2254 17.7746 69 36 69ZM52.1716 38.6337L28.4366 51.5801C26.4374 52.6705 24 51.2235 24 48.9464V23.0536C24 20.7764 26.4374 19.3295 28.4366 20.4199L52.1716 33.3663C54.2562 34.5034 54.2562 37.4966 52.1716 38.6337Z"
        fill="currentColor"
      />
    </svg>
  );
}
export function Heart({liked, animate}) {
  return (
    <>
      <svg
        className="absolute overflow-visible"
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg">
        <circle
          className={`circle ${liked ? 'liked' : ''} ${animate ? 'animate' : ''}`}
          cx="12"
          cy="12"
          r="11.5"
          fill="transparent"
          strokeWidth="0"
          stroke="currentColor"
        />
      </svg>

      <svg
        className={`heart ${liked ? 'liked' : ''} ${animate ? 'animate' : ''}`}
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg">
        {liked ? (
          <path
            d="M12 23a.496.496 0 0 1-.26-.074C7.023 19.973 0 13.743 0 8.68c0-4.12 2.322-6.677 6.058-6.677 2.572 0 5.108 2.387 5.134 2.41l.808.771.808-.771C12.834 4.387 15.367 2 17.935 2 21.678 2 24 4.558 24 8.677c0 5.06-7.022 11.293-11.74 14.246a.496.496 0 0 1-.26.074V23z"
            fill="currentColor"
          />
        ) : (
          <path
            fillRule="evenodd"
            clipRule="evenodd"
            d="m12 5.184-.808-.771-.004-.004C11.065 4.299 8.522 2.003 6 2.003c-3.736 0-6 2.558-6 6.677 0 4.47 5.471 9.848 10 13.079.602.43 1.187.82 1.74 1.167A.497.497 0 0 0 12 23v-.003c.09 0 .182-.026.26-.074C16.977 19.97 24 13.737 24 8.677 24 4.557 21.743 2 18 2c-2.569 0-5.166 2.387-5.192 2.413L12 5.184zm-.002 15.525c2.071-1.388 4.477-3.342 6.427-5.47C20.72 12.733 22 10.401 22 8.677c0-1.708-.466-2.855-1.087-3.55C20.316 4.459 19.392 4 18 4c-.726 0-1.63.364-2.5.9-.67.412-1.148.82-1.266.92-.03.025-.037.031-.019.014l-.013.013L12 7.949 9.832 5.88a10.08 10.08 0 0 0-1.33-.977C7.633 4.367 6.728 4.003 6 4.003c-1.388 0-2.312.459-2.91 1.128C2.466 5.826 2 6.974 2 8.68c0 1.726 1.28 4.058 3.575 6.563 1.948 2.127 4.352 4.078 6.423 5.466z"
            fill="currentColor"
          />
        )}
      </svg>
    </>
  );
}

export function IconSearch(props) {
  return (
    <svg width="1em" height="1em" viewBox="0 0 20 20">
      <path
        d="M14.386 14.386l4.0877 4.0877-4.0877-4.0877c-2.9418 2.9419-7.7115 2.9419-10.6533 0-2.9419-2.9418-2.9419-7.7115 0-10.6533 2.9418-2.9419 7.7115-2.9419 10.6533 0 2.9419 2.9418 2.9419 7.7115 0 10.6533z"
        stroke="currentColor"
        fill="none"
        strokeWidth="2"
        fillRule="evenodd"
        strokeLinecap="round"
        strokeLinejoin="round"></path>
    </svg>
  );
}
```

```js src/Layout.js hidden
import {ViewTransition} from 'react';
import { useIsNavPending } from "./router";

export default function Page({ heading, children }) {
  const isPending = useIsNavPending();
  return (
    <div className="page">
      <div className="top">
        <div className="top-nav">
          {/* 基于过渡类型的自定义类。 */}
          <ViewTransition
            name="nav"
            share={{
              'nav-forward': 'slide-forward',
              'nav-back': 'slide-back',
            }}>
            {heading}
          </ViewTransition>
          {isPending && <span className="loader"></span>}
        </div>
      </div>
      {/* 内容不使用 ViewTransition。 */}
      {/* 内容可以定义自己的 ViewTransition。 */}
      <ViewTransition default="none">
        <div className="bottom">
          <div className="content">{children}</div>
        </div>
      </ViewTransition>
    </div>
  );
}
```

```js src/LikeButton.js hidden
import {useState} from 'react';
import {Heart} from './Icons';

// 由于我们实际上没有后端，这是一个变通办法。
// 与本地状态不同，即使视频被筛选掉，它也能保留。
const likedVideos = new Set();

export default function LikeButton({video}) {
  const [isLiked, setIsLiked] = useState(() => likedVideos.has(video.id));
  const [animate, setAnimate] = useState(false);
  return (
    <button
      className={`like-button ${isLiked && 'liked'}`}
      aria-label={isLiked ? '取消保存' : '保存'}
      onClick={() => {
        const nextIsLiked = !isLiked;
        if (nextIsLiked) {
          likedVideos.add(video.id);
        } else {
          likedVideos.delete(video.id);
        }
        setAnimate(true);
        setIsLiked(nextIsLiked);
      }}>
      <Heart liked={isLiked} animate={animate} />
    </button>
  );
}
```

```js src/Videos.js hidden
import { useState, ViewTransition } from "react";
import LikeButton from "./LikeButton";
import { useRouter } from "./router";
import { PauseIcon, PlayIcon } from "./Icons";
import { startTransition } from "react";

export function Thumbnail({ video, children }) {
  // 添加名称，以便使用共享元素过渡进行动画。
  // 这使用默认动画，不需要额外的 css。
  return (
    <ViewTransition name={`video-${video.id}`}>
      <div
        aria-hidden="true"
        tabIndex={-1}
        className={`thumbnail ${video.image}`}
      >
        {children}
      </div>
    </ViewTransition>
  );
}

export function VideoControls() {
  const [isPlaying, setIsPlaying] = useState(false);

  return (
    <span
      className="controls"
      onClick={() =>
        startTransition(() => {
          setIsPlaying((p) => !p);
        })
      }
    >
      {isPlaying ? <PauseIcon /> : <PlayIcon />}
    </span>
  );
}

export function Video({ video }) {
  const { navigate } = useRouter();

  return (
    <div className="video">
      <div
        className="link"
        onClick={(e) => {
          e.preventDefault();
          navigate(`/video/${video.id}`);
        }}
      >
        <Thumbnail video={video}></Thumbnail>

        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
      <LikeButton video={video} />
    </div>
  );
}
```


```js src/data.js hidden
const videos = [
  {
    id: '1',
    title: '第一条视频',
    description: '视频描述',
    image: 'blue',
  },
  {
    id: '2',
    title: '第二条视频',
    description: '视频描述',
    image: 'red',
  },
  {
    id: '3',
    title: '第三条视频',
    description: '视频描述',
    image: 'green',
  },
  {
    id: '4',
    title: '第四条视频',
    description: '视频描述',
    image: 'purple',
  },
  {
    id: '5',
    title: '第五条视频',
    description: '视频描述',
    image: 'yellow',
  },
  {
    id: '6',
    title: '第六条视频',
    description: '视频描述',
    image: 'gray',
  },
];

let videosCache = new Map();
let videoCache = new Map();
let videoDetailsCache = new Map();
const VIDEO_DELAY = 1;
const VIDEO_DETAILS_DELAY = 1000;
export function fetchVideos() {
  if (videosCache.has(0)) {
    return videosCache.get(0);
  }
  const promise = new Promise((resolve) => {
    setTimeout(() => {
      resolve(videos);
    }, VIDEO_DELAY);
  });
  videosCache.set(0, promise);
  return promise;
}

export function fetchVideo(id) {
  if (videoCache.has(id)) {
    return videoCache.get(id);
  }
  const promise = new Promise((resolve) => {
    setTimeout(() => {
      resolve(videos.find((video) => video.id === id));
    }, VIDEO_DELAY);
  });
  videoCache.set(id, promise);
  return promise;
}

export function fetchVideoDetails(id) {
  if (videoDetailsCache.has(id)) {
    return videoDetailsCache.get(id);
  }
  const promise = new Promise((resolve) => {
    setTimeout(() => {
      resolve(videos.find((video) => video.id === id));
    }, VIDEO_DETAILS_DELAY);
  });
  videoDetailsCache.set(id, promise);
  return promise;
}
```

```js src/router.js hidden
import {useState, createContext, use, useTransition, useLayoutEffect, useEffect, addTransitionType} from "react";

export function Router({ children }) {
  const [isPending, startTransition] = useTransition();
  const [routerState, setRouterState] = useState({pendingNav: () => {}, url: document.location.pathname});
  function navigate(url) {
    startTransition(() => {
      // 前进导航的过渡类型
      addTransitionType('nav-forward');
      go(url);
    });
  }
  function navigateBack(url) {
    startTransition(() => {
      // 后退导航的过渡类型
      addTransitionType('nav-back');
      go(url);
    });
  }

  function go(url) {
    setRouterState({
      url,
      pendingNav() {
        window.history.pushState({}, "", url);
      },
    });
  }

  useEffect(() => {
    function handlePopState() {
      // 这不应该有动画，因为恢复必须是同步的。
      // 即使它是一个过渡。
      startTransition(() => {
        setRouterState({
          url: document.location.pathname + document.location.search,
          pendingNav() {
            // 空操作。URL 已经更新。
          },
        });
      });
    }
    window.addEventListener("popstate", handlePopState);
    return () => {
      window.removeEventListener("popstate", handlePopState);
    };
  }, []);
  const pendingNav = routerState.pendingNav;
  useLayoutEffect(() => {
    pendingNav();
  }, [pendingNav]);

  return (
    <RouterContext
      value={{
        url: routerState.url,
        navigate,
        navigateBack,
        isPending,
        params: {},
      }}
    >
      {children}
    </RouterContext>
  );
}

const RouterContext = createContext({ url: "/", params: {} });

export function useRouter() {
  return use(RouterContext);
}

export function useIsNavPending() {
  return use(RouterContext).isPending;
}

```

```css src/styles.css hidden
@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Rg.woff2) format("woff2");
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Md.woff2) format("woff2");
  font-weight: 500;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Bd.woff2) format("woff2");
  font-weight: 600;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Bd.woff2) format("woff2");
  font-weight: 700;
  font-style: normal;
  font-display: swap;
}

* {
  box-sizing: border-box;
}

html {
  background-image: url(https://react.dev/images/meta-gradient-dark.png);
  background-size: 100%;
  background-position: -100%;
  background-color: rgb(64 71 86);
  background-repeat: no-repeat;
  height: 100%;
  width: 100%;
}

body {
  font-family: Optimistic Text, -apple-system, ui-sans-serif, system-ui, sans-serif, Apple Color Emoji, Segoe UI Emoji, Segoe UI Symbol, Noto Color Emoji;
  padding: 10px 0 10px 0;
  margin: 0;
  display: flex;
  justify-content: center;
}

#root {
  flex: 1 1;
  height: auto;
  background-color: #fff;
  border-radius: 10px;
  max-width: 450px;
  min-height: 600px;
  padding-bottom: 10px;
}

h1 {
  margin-top: 0;
  font-size: 22px;
}

h2 {
  margin-top: 0;
  font-size: 20px;
}

h3 {
  margin-top: 0;
  font-size: 18px;
}

h4 {
  margin-top: 0;
  font-size: 16px;
}

h5 {
  margin-top: 0;
  font-size: 14px;
}

h6 {
  margin-top: 0;
  font-size: 12px;
}

code {
  font-size: 1.2em;
}

ul {
  padding-inline-start: 20px;
}

.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}

.absolute {
  position: absolute;
}

.overflow-visible {
  overflow: visible;
}

.visible {
  overflow: visible;
}

.fit {
  width: fit-content;
}


/* 布局 */
.page {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.top-hero {
  height: 200px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-image: conic-gradient(
      from 90deg at -10% 100%,
      #2b303b 0deg,
      #2b303b 90deg,
      #16181d 1turn
  );
}

.bottom {
  flex: 1;
  overflow: auto;
}

.top-nav {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 0;
  padding: 0 12px;
  top: 0;
  width: 100%;
  height: 44px;
  color: #23272f;
  font-weight: 700;
  font-size: 20px;
  z-index: 100;
  cursor: default;
}

.content {
  padding: 0 12px;
  margin-top: 4px;
}


.loader {
  color: #23272f;
  font-size: 3px;
  width: 1em;
  margin-right: 18px;
  height: 1em;
  border-radius: 50%;
  position: relative;
  text-indent: -9999em;
  animation: loading-spinner 1.3s infinite linear;
  animation-delay: 200ms;
  transform: translateZ(0);
}

@keyframes loading-spinner {
  0%,
  100% {
    box-shadow: 0 -3em 0 0.2em,
    2em -2em 0 0em, 3em 0 0 -1em,
    2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 -1em, -3em 0 0 -1em,
    -2em -2em 0 0;
  }
  12.5% {
    box-shadow: 0 -3em 0 0, 2em -2em 0 0.2em,
    3em 0 0 0, 2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 -1em, -3em 0 0 -1em,
    -2em -2em 0 -1em;
  }
  25% {
    box-shadow: 0 -3em 0 -0.5em,
    2em -2em 0 0, 3em 0 0 0.2em,
    2em 2em 0 0, 0 3em 0 -1em,
    -2em 2em 0 -1em, -3em 0 0 -1em,
    -2em -2em 0 -1em;
  }
  37.5% {
    box-shadow: 0 -3em 0 -1em, 2em -2em 0 -1em,
    3em 0em 0 0, 2em 2em 0 0.2em, 0 3em 0 0em,
    -2em 2em 0 -1em, -3em 0em 0 -1em, -2em -2em 0 -1em;
  }
  50% {
    box-shadow: 0 -3em 0 -1em, 2em -2em 0 -1em,
    3em 0 0 -1em, 2em 2em 0 0em, 0 3em 0 0.2em,
    -2em 2em 0 0, -3em 0em 0 -1em, -2em -2em 0 -1em;
  }
  62.5% {
    box-shadow: 0 -3em 0 -1em, 2em -2em 0 -1em,
    3em 0 0 -1em, 2em 2em 0 -1em, 0 3em 0 0,
    -2em 2em 0 0.2em, -3em 0 0 0, -2em -2em 0 -1em;
  }
  75% {
    box-shadow: 0em -3em 0 -1em, 2em -2em 0 -1em,
    3em 0em 0 -1em, 2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 0, -3em 0em 0 0.2em, -2em -2em 0 0;
  }
  87.5% {
    box-shadow: 0em -3em 0 0, 2em -2em 0 -1em,
    3em 0 0 -1em, 2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 0, -3em 0em 0 0, -2em -2em 0 0.2em;
  }
}

/* LikeButton */
.like-button {
  outline-offset: 2px;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 2.5rem;
  height: 2.5rem;
  cursor: pointer;
  border-radius: 9999px;
  border: none;
  outline: none 2px;
  color: #5e687e;
  background: none;
}

.like-button:focus {
  color: #a6423a;
  background-color: rgba(166, 66, 58, .05);
}

.like-button:active {
  color: #a6423a;
  background-color: rgba(166, 66, 58, .05);
  transform: scaleX(0.95) scaleY(0.95);
}

.like-button:hover {
  background-color: #f6f7f9;
}

.like-button.liked {
  color: #a6423a;
}

/* 图标 */
@keyframes circle {
  0% {
    transform: scale(0);
    stroke-width: 16px;
  }

  50% {
    transform: scale(.5);
    stroke-width: 16px;
  }

  to {
    transform: scale(1);
    stroke-width: 0;
  }
}

.circle {
  color: rgba(166, 66, 58, .5);
  transform-origin: center;
  transition-property: all;
  transition-duration: .15s;
  transition-timing-function: cubic-bezier(.4,0,.2,1);
}

.circle.liked.animate {
  animation: circle .3s forwards;
}

.heart {
  width: 1.5rem;
  height: 1.5rem;
}

.heart.liked {
  transform-origin: center;
  transition-property: all;
  transition-duration: .15s;
  transition-timing-function: cubic-bezier(.4, 0, .2, 1);
}

.heart.liked.animate {
  animation: scale .35s ease-in-out forwards;
}

.control-icon {
  color: hsla(0, 0%, 100%, .5);
  filter:  drop-shadow(0 20px 13px rgba(0, 0, 0, .03)) drop-shadow(0 8px 5px rgba(0, 0, 0, .08));
}

.chevron-left {
  margin-top: 2px;
  rotate: 90deg;
}


/* 视频 */
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}

.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}

.thumbnail.red {
  background-image: conic-gradient(at top right, #c76a15, #a6423a, #2b3491);
}

.thumbnail.green {
  background-image: conic-gradient(at top right, #c76a15, #388f7f, #2b3491);
}

.thumbnail.purple {
  background-image: conic-gradient(at top right, #c76a15, #575fb7, #2b3491);
}

.thumbnail.yellow {
  background-image: conic-gradient(at top right, #c76a15, #FABD62, #2b3491);
}

.thumbnail.gray {
  background-image: conic-gradient(at top right, #c76a15, #4E5769, #2b3491);
}

.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
}

.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
  cursor: pointer;
}

.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}

.video .info:hover {
  text-decoration: underline;
}

.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}

.video-description {
  color: #5e687e;
  font-size: 13px;
}

/* 详情 */
.details .thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 100%;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}

.video-details-title {
  margin-top: 8px;
}

.video-details-speaker {
  display: flex;
  gap: 8px;
  margin-top: 10px
}

.back {
  display: flex;
  align-items: center;
  margin-left: -5px;
  cursor: pointer;
}

.back:hover {
  text-decoration: underline;
}

.info-title {
  font-size: 1.5rem;
  font-weight: 700;
  line-height: 1.25;
  margin: 8px 0 0 0 ;
}

.info-description {
  margin: 8px 0 0 0;
}

.controls {
  cursor: pointer;
}

.fallback {
  background: #f6f7f8 linear-gradient(to right, #e6e6e6 5%, #cccccc 25%, #e6e6e6 35%) no-repeat;
  background-size: 800px 104px;
  display: block;
  line-height: 1.25;
  margin: 8px 0 0 0;
  border-radius: 5px;
  overflow: hidden;

  animation: 1s linear 1s infinite shimmer;
  animation-delay: 300ms;
  animation-duration: 1s;
  animation-fill-mode: forwards;
  animation-iteration-count: infinite;
  animation-name: shimmer;
  animation-timing-function: linear;
}


.fallback.title {
  width: 130px;
  height: 30px;

}

.fallback.description {
  width: 150px;
  height: 21px;
}

@keyframes shimmer {
  0% {
    background-position: -468px 0;
  }

  100% {
    background-position: 468px 0;
  }
}

.search {
  margin-bottom: 10px;
}
.search-input {
  width: 100%;
  position: relative;
}

.search-icon {
  position: absolute;
  top: 0;
  bottom: 0;
  inset-inline-start: 0;
  display: flex;
  align-items: center;
  padding-inline-start: 1rem;
  pointer-events: none;
  color: #99a1b3;
}

.search-input input {
  display: flex;
  padding-inline-start: 2.75rem;
  padding-top: 10px;
  padding-bottom: 10px;
  width: 100%;
  text-align: start;
  background-color: rgb(235 236 240);
  outline: 2px solid transparent;
  cursor: pointer;
  border: none;
  align-items: center;
  color: rgb(35 39 47);
  border-radius: 9999px;
  vertical-align: middle;
  font-size: 15px;
}

.search-input input:hover, .search-input input:active {
  background-color: rgb(235 236 240/ 0.8);
  color: rgb(35 39 47/ 0.8);
}

/* 首页 */
.video-list {
  position: relative;
}

.video-list .videos {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  overflow-y: auto;
  height: 100%;
}
```


```css src/animations.css
/* 无需额外动画 */









/* 下面是之前定义的动画 */






/* Suspense 的滑动动画 */
::view-transition-old(.slide-down) {
    animation: 150ms ease-out both fade-out, 150ms ease-out both slide-down;
}

::view-transition-new(.slide-up) {
    animation: 210ms ease-in 150ms both fade-in, 400ms ease-in both slide-up;
}

/* 根据过渡类型添加的视图过渡类的动画 */
::view-transition-old(.slide-forward) {
    /* 当向前滑动时，“旧”页面应该向左滑出。 */
    animation: 150ms cubic-bezier(0.4, 0, 1, 1) both fade-out,
    400ms cubic-bezier(0.4, 0, 0.2, 1) both slide-to-left;
}

::view-transition-new(.slide-forward) {
    /* 当向前滑动时，“新”页面应该从右侧滑入。 */
    animation: 210ms cubic-bezier(0, 0, 0.2, 1) 150ms both fade-in,
    400ms cubic-bezier(0.4, 0, 0.2, 1) both slide-from-right;
}

::view-transition-old(.slide-back) {
    /* 当向后滑动时，“旧”页面应该向右滑出。 */
    animation: 150ms cubic-bezier(0.4, 0, 1, 1) both fade-out,
    400ms cubic-bezier(0.4, 0, 0.2, 1) both slide-to-right;
}

::view-transition-new(.slide-back) {
    /* 当向后滑动时，“新”页面应该从左侧滑入。 */
    animation: 210ms cubic-bezier(0, 0, 0.2, 1) 150ms both fade-in,
    400ms cubic-bezier(0.4, 0, 0.2, 1) both slide-from-left;
}

/* 支持上面动画的关键帧。 */
@keyframes slide-up {
    from {
        transform: translateY(10px);
    }
    to {
        transform: translateY(0);
    }
}

@keyframes slide-down {
    from {
        transform: translateY(0);
    }
    to {
        transform: translateY(10px);
    }
}

@keyframes fade-in {
    from {
        opacity: 0;
    }
}

@keyframes fade-out {
    to {
        opacity: 0;
    }
}

@keyframes slide-to-right {
    to {
        transform: translateX(50px);
    }
}

@keyframes slide-from-right {
    from {
        transform: translateX(50px);
    }
    to {
        transform: translateX(0);
    }
}

@keyframes slide-to-left {
    to {
        transform: translateX(-50px);
    }
}

@keyframes slide-from-left {
    from {
        transform: translateX(-50px);
    }
    to {
        transform: translateX(0);
    }
}


/* 默认的 .slow-fade。 */
::view-transition-old(.slow-fade) {
    animation-duration: 500ms;
}

::view-transition-new(.slow-fade) {
    animation-duration: 500ms;
}
```

```js src/index.js hidden
import React, {StrictMode} from 'react';
import {createRoot} from 'react-dom/client';
import './styles.css';
import './animations.css';

import App from './App';
import {Router} from './router';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <Router>
      <App />
    </Router>
  </StrictMode>
);
```

```json package.json hidden
{
  "dependencies": {
    "react": "canary",
    "react-dom": "canary",
    "react-scripts": "latest"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

</Sandpack>

### 动画列表 {/*animating-lists*/}

你也可以使用 `<ViewTransition>` 来为项目列表在重新排序时添加动画，例如可搜索项目列表：

```js {3,5}
<div className="videos">
  {filteredVideos.map((video) => (
    <ViewTransition key={video.id}>
      <Video video={video} />
    </ViewTransition>
  ))}
</div>
```

要激活 ViewTransition，我们可以使用 `useDeferredValue`：

```js {2}
const [searchText, setSearchText] = useState('');
const deferredSearchText = useDeferredValue(searchText);
const filteredVideos = filterVideos(videos, deferredSearchText);
```

现在，当你在搜索栏中输入时，项目会播放动画：

<Sandpack>

```js src/App.js hidden
import { ViewTransition } from "react";
import Details from "./Details";
import Home from "./Home";
import { useRouter } from "./router";

export default function App() {
  const { url } = useRouter();

  // 默认慢速淡入淡出动画。
  return (
    <ViewTransition default="slow-fade">
      {url === "/" ? <Home /> : <Details />}
    </ViewTransition>
  );
}
```

```js src/Details.js hidden
import { use, Suspense, ViewTransition } from "react";
import { fetchVideo, fetchVideoDetails } from "./data";
import { Thumbnail, VideoControls } from "./Videos";
import { useRouter } from "./router";
import Layout from "./Layout";
import { ChevronLeft } from "./Icons";

function VideoDetails({id}) {
  // 从 Suspense 回退内容过渡到实际内容
  return (
    <Suspense
      fallback={
        // 将回退内容向下动画。
        <ViewTransition exit="slide-down">
          <VideoInfoFallback />
        </ViewTransition>
      }
    >
      {/* 将内容向上动画 */}
      <ViewTransition enter="slide-up">
        <VideoInfo id={id} />
      </ViewTransition>
    </Suspense>
  );
}

function VideoInfoFallback() {
  return (
    <>
      <div className="fallback title"></div>
      <div className="fallback description"></div>
    </>
  );
}

export default function Details() {
  const { url, navigateBack } = useRouter();
  const videoId = url.split("/").pop();
  const video = use(fetchVideo(videoId));

  return (
    <Layout
      heading={
        <div
          className="fit back"
          onClick={() => {
            navigateBack("/");
          }}
        >
          <ChevronLeft /> 返回
        </div>
      }
    >
      <div className="details">
        <Thumbnail video={video} large>
          <VideoControls />
        </Thumbnail>
        <VideoDetails id={video.id} />
      </div>
    </Layout>
  );
}

function VideoInfo({ id }) {
  const details = use(fetchVideoDetails(id));
  return (
    <>
      <p className="info-title">{details.title}</p>
      <p className="info-description">{details.description}</p>
    </>
  );
}
```

```js src/Home.js
import { useId, useState, use, useDeferredValue, ViewTransition } from "react";import { Video } from "./Videos";import Layout from "./Layout";import { fetchVideos } from "./data";import { IconSearch } from "./Icons";

function SearchList({searchText, videos}) {
  // 使用 useDeferredValue 激活（“何时”）
  const deferredSearchText = useDeferredValue(searchText);
  const filteredVideos = filterVideos(videos, deferredSearchText);
  return (
    <div className="video-list">
      <div className="videos">
        {filteredVideos.map((video) => (
          // 为列表中的每一项添加动画（“什么”）
          <ViewTransition key={video.id}>
            <Video video={video} />
          </ViewTransition>
        ))}
      </div>
      {filteredVideos.length === 0 && (
        <div className="no-results">没有结果</div>
      )}
    </div>
  );
}

export default function Home() {
  const videos = use(fetchVideos());
  const count = videos.length;
  const [searchText, setSearchText] = useState('');

  return (
    <Layout heading={<div className="fit">{count} 个视频</div>}>
      <SearchInput value={searchText} onChange={setSearchText} />
      <SearchList videos={videos} searchText={searchText} />
    </Layout>
  );
}

function SearchInput({ value, onChange }) {
  const id = useId();
  return (
    <form className="search" onSubmit={(e) => e.preventDefault()}>
      <label htmlFor={id} className="sr-only">
        搜索
      </label>
      <div className="search-input">
        <div className="search-icon">
          <IconSearch />
        </div>
        <input
          type="text"
          id={id}
          placeholder="搜索"
          value={value}
          onChange={(e) => onChange(e.target.value)}
        />
      </div>
    </form>
  );
}

function filterVideos(videos, query) {
  const keywords = query
    .toLowerCase()
    .split(" ")
    .filter((s) => s !== "");
  if (keywords.length === 0) {
    return videos;
  }
  return videos.filter((video) => {
    const words = (video.title + " " + video.description)
      .toLowerCase()
      .split(" ");
    return keywords.every((kw) => words.some((w) => w.includes(kw)));
  });
}
```

```js src/Icons.js hidden
export function ChevronLeft() {
  return (
    <svg
      className="chevron-left"
      xmlns="http://www.w3.org/2000/svg"
      width="20"
      height="20"
      viewBox="0 0 20 20">
      <g fill="none" fillRule="evenodd" transform="translate(-446 -398)">
        <path
          fill="currentColor"
          fillRule="nonzero"
          d="M95.8838835,240.366117 C95.3957281,239.877961 94.6042719,239.877961 94.1161165,240.366117 C93.6279612,240.854272 93.6279612,241.645728 94.1161165,242.133883 L98.6161165,246.633883 C99.1042719,247.122039 99.8957281,247.122039 100.383883,246.633883 L104.883883,242.133883 C105.372039,241.645728 105.372039,240.854272 104.883883,240.366117 C104.395728,239.877961 103.604272,239.877961 103.116117,240.366117 L99.5,243.982233 L95.8838835,240.366117 Z"
          transform="translate(356.5 164.5)"
        />
        <polygon points="446 418 466 418 466 398 446 398" />
      </g>
    </svg>
  );
}

export function PauseIcon() {
  return (
    <svg
      className="control-icon"
      style={{padding: '4px'}}
      width="100"
      height="100"
      viewBox="0 0 512 512"
      fill="none"
      xmlns="http://www.w3.org/2000/svg">
      <path
        fillRule="evenodd"
        clipRule="evenodd"
        d="M256 0C114.617 0 0 114.615 0 256s114.617 256 256 256 256-114.615 256-256S397.383 0 256 0zm-32 320c0 8.836-7.164 16-16 16h-32c-8.836 0-16-7.164-16-16V192c0-8.836 7.164-16 16-16h32c8.836 0 16 7.164 16 16v128zm128 0c0 8.836-7.164 16-16 16h-32c-8.836 0-16-7.164-16-16V192c0-8.836 7.164-16 16-16h32c8.836 0 16 7.164 16 16v128z"
        fill="currentColor"
      />
    </svg>
  );
}

export function PlayIcon() {
  return (
    <svg
      className="control-icon"
      width="100"
      height="100"
      viewBox="0 0 72 72"
      fill="none"
      xmlns="http://www.w3.org/2000/svg">
      <path
        fillRule="evenodd"
        clipRule="evenodd"
        d="M36 69C54.2254 69 69 54.2254 69 36C69 17.7746 54.2254 3 36 3C17.7746 3 3 17.7746 3 36C3 54.2254 17.7746 69 36 69ZM52.1716 38.6337L28.4366 51.5801C26.4374 52.6705 24 51.2235 24 48.9464V23.0536C24 20.7764 26.4374 19.3295 28.4366 20.4199L52.1716 33.3663C54.2562 34.5034 54.2562 37.4966 52.1716 38.6337Z"
        fill="currentColor"
      />
    </svg>
  );
}
export function Heart({liked, animate}) {
  return (
    <>
      <svg
        className="absolute overflow-visible"
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg">
        <circle
          className={`circle ${liked ? 'liked' : ''} ${animate ? 'animate' : ''}`}
          cx="12"
          cy="12"
          r="11.5"
          fill="transparent"
          strokeWidth="0"
          stroke="currentColor"
        />
      </svg>

      <svg
        className={`heart ${liked ? 'liked' : ''} ${animate ? 'animate' : ''}`}
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg">
        {liked ? (
          <path
            d="M12 23a.496.496 0 0 1-.26-.074C7.023 19.973 0 13.743 0 8.68c0-4.12 2.322-6.677 6.058-6.677 2.572 0 5.108 2.387 5.134 2.41l.808.771.808-.771C12.834 4.387 15.367 2 17.935 2 21.678 2 24 4.558 24 8.677c0 5.06-7.022 11.293-11.74 14.246a.496.496 0 0 1-.26.074V23z"
            fill="currentColor"
          />
        ) : (
          <path
            fillRule="evenodd"
            clipRule="evenodd"
            d="m12 5.184-.808-.771-.004-.004C11.065 4.299 8.522 2.003 6 2.003c-3.736 0-6 2.558-6 6.677 0 4.47 5.471 9.848 10 13.079.602.43 1.187.82 1.74 1.167A.497.497 0 0 0 12 23v-.003c.09 0 .182-.026.26-.074C16.977 19.97 24 13.737 24 8.677 24 4.557 21.743 2 18 2c-2.569 0-5.166 2.387-5.192 2.413L12 5.184zm-.002 15.525c2.071-1.388 4.477-3.342 6.427-5.47C20.72 12.733 22 10.401 22 8.677c0-1.708-.466-2.855-1.087-3.55C20.316 4.459 19.392 4 18 4c-.726 0-1.63.364-2.5.9-.67.412-1.148.82-1.266.92-.03.025-.037.031-.019.014l-.013.013L12 7.949 9.832 5.88a10.08 10.08 0 0 0-1.33-.977C7.633 4.367 6.728 4.003 6 4.003c-1.388 0-2.312.459-2.91 1.128C2.466 5.826 2 6.974 2 8.68c0 1.726 1.28 4.058 3.575 6.563 1.948 2.127 4.352 4.078 6.423 5.466z"
            fill="currentColor"
          />
        )}
      </svg>
    </>
  );
}

export function IconSearch(props) {
  return (
    <svg width="1em" height="1em" viewBox="0 0 20 20">
      <path
        d="M14.386 14.386l4.0877 4.0877-4.0877-4.0877c-2.9418 2.9419-7.7115 2.9419-10.6533 0-2.9419-2.9418-2.9419-7.7115 0-10.6533 2.9418-2.9419 7.7115-2.9419 10.6533 0 2.9419 2.9418 2.9419 7.7115 0 10.6533z"
        stroke="currentColor"
        fill="none"
        strokeWidth="2"
        fillRule="evenodd"
        strokeLinecap="round"
        strokeLinejoin="round"></path>
    </svg>
  );
}
```

```js src/Layout.js
import {ViewTransition} from 'react'; import { useIsNavPending } from "./router";

export default function Page({ heading, children }) {
  const isPending = useIsNavPending();
  return (
    <div className="page">
      <div className="top">
        <div className="top-nav">
          {/* 根据过渡类型使用自定义类。 */}
          <ViewTransition
            name="nav"
            share={{
              'nav-forward': 'slide-forward',
              'nav-back': 'slide-back',
            }}>
            {heading}
          </ViewTransition>
          {isPending && <span className="loader"></span>}
        </div>
      </div>
      {/* 为内容禁用 ViewTransition。 */}
      {/* 内容可以定义自己的 ViewTransition。 */}
      <ViewTransition default="none">
        <div className="bottom">
          <div className="content">{children}</div>
        </div>
      </ViewTransition>
    </div>
  );
}
```

```js src/LikeButton.js hidden
import {useState} from 'react';
import {Heart} from './Icons';

// 这是一个变通办法，因为我们实际上没有后端。
// 与本地状态不同，这种方式在视频被筛选时仍然会保留。
const likedVideos = new Set();

export default function LikeButton({video}) {
  const [isLiked, setIsLiked] = useState(() => likedVideos.has(video.id));
  const [animate, setAnimate] = useState(false);
  return (
    <button
      className={`like-button ${isLiked && 'liked'}`}
      aria-label={isLiked ? '取消保存' : '保存'}
      onClick={() => {
        const nextIsLiked = !isLiked;
        if (nextIsLiked) {
          likedVideos.add(video.id);
        } else {
          likedVideos.delete(video.id);
        }
        setAnimate(true);
        setIsLiked(nextIsLiked);
      }}>
      <Heart liked={isLiked} animate={animate} />
    </button>
  );
}
```

```js src/Videos.js
import { useState, ViewTransition } from "react"; import LikeButton from "./LikeButton"; import { useRouter } from "./router"; import { PauseIcon, PlayIcon } from "./Icons"; import { startTransition } from "react";

export function Thumbnail({ video, children }) {
  // 添加名称以使用共享元素过渡进行动画。
  return (
    <ViewTransition name={`video-${video.id}`}>
      <div
        aria-hidden="true"
        tabIndex={-1}
        className={`thumbnail ${video.image}`}
      >
        {children}
      </div>
    </ViewTransition>
  );
}



export function VideoControls() {
  const [isPlaying, setIsPlaying] = useState(false);

  return (
    <span
      className="controls"
      onClick={() =>
        startTransition(() => {
          setIsPlaying((p) => !p);
        })
      }
    >
      {isPlaying ? <PauseIcon /> : <PlayIcon />}
    </span>
  );
}

export function Video({ video }) {
  const { navigate } = useRouter();

  return (
    <div className="video">
      <div
        className="link"
        onClick={(e) => {
          e.preventDefault();
          navigate(`/video/${video.id}`);
        }}
      >
        <Thumbnail video={video}></Thumbnail>

        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
      <LikeButton video={video} />
    </div>
  );
}
```

```js src/data.js hidden
const videos = [
  {
    id: '1',
    title: '第一个视频',
    description: '视频描述',
    image: 'blue',
  },
  {
    id: '2',
    title: '第二个视频',
    description: '视频描述',
    image: 'red',
  },
  {
    id: '3',
    title: '第三个视频',
    description: '视频描述',
    image: 'green',
  },
  {
    id: '4',
    title: '第四个视频',
    description: '视频描述',
    image: 'purple',
  },
  {
    id: '5',
    title: '第五个视频',
    description: '视频描述',
    image: 'yellow',
  },
  {
    id: '6',
    title: '第六个视频',
    description: '视频描述',
    image: 'gray',
  },
];

let videosCache = new Map();
let videoCache = new Map();
let videoDetailsCache = new Map();
const VIDEO_DELAY = 1;
const VIDEO_DETAILS_DELAY = 1000;
export function fetchVideos() {
  if (videosCache.has(0)) {
    return videosCache.get(0);
  }
  const promise = new Promise((resolve) => {
    setTimeout(() => {
      resolve(videos);
    }, VIDEO_DELAY);
  });
  videosCache.set(0, promise);
  return promise;
}

export function fetchVideo(id) {
  if (videoCache.has(id)) {
    return videoCache.get(id);
  }
  const promise = new Promise((resolve) => {
    setTimeout(() => {
      resolve(videos.find((video) => video.id === id));
    }, VIDEO_DELAY);
  });
  videoCache.set(id, promise);
  return promise;
}

export function fetchVideoDetails(id) {
  if (videoDetailsCache.has(id)) {
    return videoDetailsCache.get(id);
  }
  const promise = new Promise((resolve) => {
    setTimeout(() => {
      resolve(videos.find((video) => video.id === id));
    }, VIDEO_DETAILS_DELAY);
  });
  videoDetailsCache.set(id, promise);
  return promise;
}
```

```js src/router.js
import {useState, createContext, use, useTransition, useLayoutEffect, useEffect, addTransitionType} from "react";

export function Router({ children }) {
  const [isPending, startTransition] = useTransition();
  function navigate(url) {
    startTransition(() => {
      // 导航前进的过渡类型
      addTransitionType('nav-forward');
      go(url);
    });
  }
  function navigateBack(url) {
    startTransition(() => {
      // 导航后退的过渡类型
      addTransitionType('nav-back');
      go(url);
    });
  }

  const [routerState, setRouterState] = useState({pendingNav: () => {}, url: document.location.pathname});

  function go(url) {
    setRouterState({
      url,
      pendingNav() {
        window.history.pushState({}, "", url);
      },
    });
  }

  useEffect(() => {
    function handlePopState() {
      // 这里不应进行动画，因为恢复必须是同步的。
      // 即使它是一个过渡。
      startTransition(() => {
        setRouterState({
          url: document.location.pathname + document.location.search,
          pendingNav() {
            // 空操作。URL 已经更新。
          },
        });
      });
    }
    window.addEventListener("popstate", handlePopState);
    return () => {
      window.removeEventListener("popstate", handlePopState);
    };
  }, []);
  const pendingNav = routerState.pendingNav;
  useLayoutEffect(() => {
    pendingNav();
  }, [pendingNav]);

  return (
    <RouterContext
      value={{
        url: routerState.url,
        navigate,
        navigateBack,
        isPending,
        params: {},
      }}
    >
      {children}
    </RouterContext>
  );
}

const RouterContext = createContext({ url: "/", params: {} });

export function useRouter() {
  return use(RouterContext);
}

export function useIsNavPending() {
  return use(RouterContext).isPending;
}

```

```css src/styles.css hidden
@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Rg.woff2) format("woff2");
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Md.woff2) format("woff2");
  font-weight: 500;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Bd.woff2) format("woff2");
  font-weight: 600;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Bd.woff2) format("woff2");
  font-weight: 700;
  font-style: normal;
  font-display: swap;
}

* {
  box-sizing: border-box;
}

html {
  background-image: url(https://react.dev/images/meta-gradient-dark.png);
  background-size: 100%;
  background-position: -100%;
  background-color: rgb(64 71 86);
  background-repeat: no-repeat;
  height: 100%;
  width: 100%;
}

body {
  font-family: Optimistic Text, -apple-system, ui-sans-serif, system-ui, sans-serif, Apple Color Emoji, Segoe UI Emoji, Segoe UI Symbol, Noto Color Emoji;
  padding: 10px 0 10px 0;
  margin: 0;
  display: flex;
  justify-content: center;
}

#root {
  flex: 1 1;
  height: auto;
  background-color: #fff;
  border-radius: 10px;
  max-width: 450px;
  min-height: 600px;
  padding-bottom: 10px;
}

h1 {
  margin-top: 0;
  font-size: 22px;
}

h2 {
  margin-top: 0;
  font-size: 20px;
}

h3 {
  margin-top: 0;
  font-size: 18px;
}

h4 {
  margin-top: 0;
  font-size: 16px;
}

h5 {
  margin-top: 0;
  font-size: 14px;
}

h6 {
  margin-top: 0;
  font-size: 12px;
}

code {
  font-size: 1.2em;
}

ul {
  padding-inline-start: 20px;
}

.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}

.absolute {
  position: absolute;
}

.overflow-visible {
  overflow: visible;
}

.visible {
  overflow: visible;
}

.fit {
  width: fit-content;
}


/* 布局 */
.page {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.top-hero {
  height: 200px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-image: conic-gradient(
      from 90deg at -10% 100%,
      #2b303b 0deg,
      #2b303b 90deg,
      #16181d 1turn
  );
}

.bottom {
  flex: 1;
  overflow: auto;
}

.top-nav {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 0;
  padding: 0 12px;
  top: 0;
  width: 100%;
  height: 44px;
  color: #23272f;
  font-weight: 700;
  font-size: 20px;
  z-index: 100;
  cursor: default;
}

.content {
  padding: 0 12px;
  margin-top: 4px;
}


.loader {
  color: #23272f;
  font-size: 3px;
  width: 1em;
  margin-right: 18px;
  height: 1em;
  border-radius: 50%;
  position: relative;
  text-indent: -9999em;
  animation: loading-spinner 1.3s infinite linear;
  animation-delay: 200ms;
  transform: translateZ(0);
}

@keyframes loading-spinner {
  0%,
  100% {
    box-shadow: 0 -3em 0 0.2em,
    2em -2em 0 0em, 3em 0 0 -1em,
    2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 -1em, -3em 0 0 -1em,
    -2em -2em 0 0;
  }
  12.5% {
    box-shadow: 0 -3em 0 0, 2em -2em 0 0.2em,
    3em 0 0 0, 2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 -1em, -3em 0 0 -1em,
    -2em -2em 0 -1em;
  }
  25% {
    box-shadow: 0 -3em 0 -0.5em,
    2em -2em 0 0, 3em 0 0 0.2em,
    2em 2em 0 0, 0 3em 0 -1em,
    -2em 2em 0 -1em, -3em 0 0 -1em,
    -2em -2em 0 -1em;
  }
  37.5% {
    box-shadow: 0 -3em 0 -1em, 2em -2em 0 -1em,
    3em 0em 0 0, 2em 2em 0 0.2em, 0 3em 0 0em,
    -2em 2em 0 -1em, -3em 0em 0 -1em, -2em -2em 0 -1em;
  }
  50% {
    box-shadow: 0 -3em 0 -1em, 2em -2em 0 -1em,
    3em 0 0 -1em, 2em 2em 0 0em, 0 3em 0 0.2em,
    -2em 2em 0 0, -3em 0em 0 -1em, -2em -2em 0 -1em;
  }
  62.5% {
    box-shadow: 0 -3em 0 -1em, 2em -2em 0 -1em,
    3em 0 0 -1em, 2em 2em 0 -1em, 0 3em 0 0,
    -2em 2em 0 0.2em, -3em 0 0 0, -2em -2em 0 -1em;
  }
  75% {
    box-shadow: 0em -3em 0 -1em, 2em -2em 0 -1em,
    3em 0em 0 -1em, 2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 0, -3em 0em 0 0.2em, -2em -2em 0 0;
  }
  87.5% {
    box-shadow: 0em -3em 0 0, 2em -2em 0 -1em,
    3em 0 0 -1em, 2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 0, -3em 0em 0 0, -2em -2em 0 0.2em;
  }
}

/* LikeButton */
.like-button {
  outline-offset: 2px;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 2.5rem;
  height: 2.5rem;
  cursor: pointer;
  border-radius: 9999px;
  border: none;
  outline: none 2px;
  color: #5e687e;
  background: none;
}

.like-button:focus {
  color: #a6423a;
  background-color: rgba(166, 66, 58, .05);
}

.like-button:active {
  color: #a6423a;
  background-color: rgba(166, 66, 58, .05);
  transform: scaleX(0.95) scaleY(0.95);
}

.like-button:hover {
  background-color: #f6f7f9;
}

.like-button.liked {
  color: #a6423a;
}

/* Icons */
@keyframes circle {
  0% {
    transform: scale(0);
    stroke-width: 16px;
  }

  50% {
    transform: scale(.5);
    stroke-width: 16px;
  }

  to {
    transform: scale(1);
    stroke-width: 0;
  }
}

.circle {
  color: rgba(166, 66, 58, .5);
  transform-origin: center;
  transition-property: all;
  transition-duration: .15s;
  transition-timing-function: cubic-bezier(.4,0,.2,1);
}

.circle.liked.animate {
  animation: circle .3s forwards;
}

.heart {
  width: 1.5rem;
  height: 1.5rem;
}

.heart.liked {
  transform-origin: center;
  transition-property: all;
  transition-duration: .15s;
  transition-timing-function: cubic-bezier(.4, 0, .2, 1);
}

.heart.liked.animate {
  animation: scale .35s ease-in-out forwards;
}

.control-icon {
  color: hsla(0, 0%, 100%, .5);
  filter:  drop-shadow(0 20px 13px rgba(0, 0, 0, .03)) drop-shadow(0 8px 5px rgba(0, 0, 0, .08));
}

.chevron-left {
  margin-top: 2px;
  rotate: 90deg;
}


/* 视频 */
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}

.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}

.thumbnail.red {
  background-image: conic-gradient(at top right, #c76a15, #a6423a, #2b3491);
}

.thumbnail.green {
  background-image: conic-gradient(at top right, #c76a15, #388f7f, #2b3491);
}

.thumbnail.purple {
  background-image: conic-gradient(at top right, #c76a15, #575fb7, #2b3491);
}

.thumbnail.yellow {
  background-image: conic-gradient(at top right, #c76a15, #FABD62, #2b3491);
}

.thumbnail.gray {
  background-image: conic-gradient(at top right, #c76a15, #4E5769, #2b3491);
}

.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
}

.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
  cursor: pointer;
}

.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}

.video .info:hover {
  text-decoration: underline;
}

.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}

.video-description {
  color: #5e687e;
  font-size: 13px;
}

/* 详情 */
.details .thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 100%;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}

.video-details-title {
  margin-top: 8px;
}

.video-details-speaker {
  display: flex;
  gap: 8px;
  margin-top: 10px
}

.back {
  display: flex;
  align-items: center;
  margin-left: -5px;
  cursor: pointer;
}

.back:hover {
  text-decoration: underline;
}

.info-title {
  font-size: 1.5rem;
  font-weight: 700;
  line-height: 1.25;
  margin: 8px 0 0 0 ;
}

.info-description {
  margin: 8px 0 0 0;
}

.controls {
  cursor: pointer;
}

.fallback {
  background: #f6f7f8 linear-gradient(to right, #e6e6e6 5%, #cccccc 25%, #e6e6e6 35%) no-repeat;
  background-size: 800px 104px;
  display: block;
  line-height: 1.25;
  margin: 8px 0 0 0;
  border-radius: 5px;
  overflow: hidden;

  animation: 1s linear 1s infinite shimmer;
  animation-delay: 300ms;
  animation-duration: 1s;
  animation-fill-mode: forwards;
  animation-iteration-count: infinite;
  animation-name: shimmer;
  animation-timing-function: linear;
}


.fallback.title {
  width: 130px;
  height: 30px;

}

.fallback.description {
  width: 150px;
  height: 21px;
}

@keyframes shimmer {
  0% {
    background-position: -468px 0;
  }

  100% {
    background-position: 468px 0;
  }
}

.search {
  margin-bottom: 10px;
}
.search-input {
  width: 100%;
  position: relative;
}

.search-icon {
  position: absolute;
  top: 0;
  bottom: 0;
  inset-inline-start: 0;
  display: flex;
  align-items: center;
  padding-inline-start: 1rem;
  pointer-events: none;
  color: #99a1b3;
}

.search-input input {
  display: flex;
  padding-inline-start: 2.75rem;
  padding-top: 10px;
  padding-bottom: 10px;
  width: 100%;
  text-align: start;
  background-color: rgb(235 236 240);
  outline: 2px solid transparent;
  cursor: pointer;
  border: none;
  align-items: center;
  color: rgb(35 39 47);
  border-radius: 9999px;
  vertical-align: middle;
  font-size: 15px;
}

.search-input input:hover, .search-input input:active {
  background-color: rgb(235 236 240/ 0.8);
  color: rgb(35 39 47/ 0.8);
}

/* 首页 */
.video-list {
  position: relative;
}

.video-list .videos {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  overflow-y: auto;
  height: 100%;
}
```


```css src/animations.css
/* 让 Suspense 的 fallback 向下滑动的动画 */
::view-transition-old(.slide-down) {
    animation: 150ms ease-out both fade-out, 150ms ease-out both slide-down;
}

::view-transition-new(.slide-up) {
    animation: 210ms ease-in 150ms both fade-in, 400ms ease-in both slide-up;
}

/* 由过渡类型添加的 view transition 类的动画 */
::view-transition-old(.slide-forward) {
    /* 向前滑动时，旧页面应向左滑出。 */
    animation: 150ms cubic-bezier(0.4, 0, 1, 1) both fade-out,
    400ms cubic-bezier(0.4, 0, 0.2, 1) both slide-to-left;
}

::view-transition-new(.slide-forward) {
    /* 向前滑动时，新页面应从右侧滑入。 */
    animation: 210ms cubic-bezier(0, 0, 0.2, 1) 150ms both fade-in,
    400ms cubic-bezier(0.4, 0, 0.2, 1) both slide-from-right;
}

::view-transition-old(.slide-back) {
    /* 向后滑动时，旧页面应向右滑出。 */
    animation: 150ms cubic-bezier(0.4, 0, 1, 1) both fade-out,
    400ms cubic-bezier(0.4, 0, 0.2, 1) both slide-to-right;
}

::view-transition-new(.slide-back) {
    /* 向后滑动时，新页面应从左侧滑入。 */
    animation: 210ms cubic-bezier(0, 0, 0.2, 1) 150ms both fade-in,
    400ms cubic-bezier(0.4, 0, 0.2, 1) both slide-from-left;
}

/* 支持上面动画的关键帧。 */
@keyframes slide-up {
    from {
        transform: translateY(10px);
    }
    to {
        transform: translateY(0);
    }
}

@keyframes slide-down {
    from {
        transform: translateY(0);
    }
    to {
        transform: translateY(10px);
    }
}

@keyframes fade-in {
    from {
        opacity: 0;
    }
}

@keyframes fade-out {
    to {
        opacity: 0;
    }
}

@keyframes slide-to-right {
    to {
        transform: translateX(50px);
    }
}

@keyframes slide-from-right {
    from {
        transform: translateX(50px);
    }
    to {
        transform: translateX(0);
    }
}

@keyframes slide-to-left {
    to {
        transform: translateX(-50px);
    }
}

@keyframes slide-from-left {
    from {
        transform: translateX(-50px);
    }
    to {
        transform: translateX(0);
    }
}
```

```js src/index.js hidden
import React, {StrictMode} from 'react';
import {createRoot} from 'react-dom/client';
import './styles.css';
import './animations.css';

import App from './App';
import {Router} from './router';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <Router>
      <App />
    </Router>
  </StrictMode>
);
```

```json package.json hidden
{
  "dependencies": {
    "react": "canary",
    "react-dom": "canary",
    "react-scripts": "latest"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

</Sandpack>

如果你想了解更多它们的工作原理，请查看文档中的 [ViewTransition 是如何工作的](/reference/react/ViewTransition#how-does-viewtransition-work)。

_如果想了解我们构建 View Transitions 的更多背景，请参见：[#31975](https://github.com/facebook/react/pull/31975)、[#32105](https://github.com/facebook/react/pull/32105)、[#32041](https://github.com/facebook/react/pull/32041)、[#32734](https://github.com/facebook/react/pull/32734)、[#32797](https://github.com/facebook/react/pull/32797) [#31999](https://github.com/facebook/react/pull/31999)、[#32031](https://github.com/facebook/react/pull/32031)、[#32050](https://github.com/facebook/react/pull/32050)、[#32820](https://github.com/facebook/react/pull/32820)、[#32029](https://github.com/facebook/react/pull/32029)、[#32028](https://github.com/facebook/react/pull/32028)，以及 [#32038](https://github.com/facebook/react/pull/32038)，作者是 [@sebmarkbage](https://twitter.com/sebmarkbage)（感谢 Seb！）。_

---

## Activity {/*activity*/}

<Note>

**`<Activity />` 现已在 React 的 Canary 版本通道中可用。**

[在此了解 React 的发布通道。](/community/versioning-policy#all-release-channels)

</Note>

在[过去](/blog/2022/06/15/react-labs-what-we-have-been-working-on-june-2022#offscreen)的[更新](/blog/2024/02/15/react-labs-what-we-have-been-working-on-february-2024#offscreen-renamed-to-activity)中，我们分享过：我们正在研究一种 API，用于让组件在视觉上隐藏并降低优先级，同时与通过卸载或使用 CSS 隐藏相比，以更低的性能成本保留 UI 状态。

现在我们已经准备好分享这个 API 以及它的工作方式，因此你可以开始在实验性 React 版本中测试它。

`<Activity>` 是一个用于隐藏和显示 UI 部分的新组件：

```js [[1, 1, "'visible'"], [2, 1, "'hidden'"]]
<Activity mode={isVisible ? 'visible' : 'hidden'}>
  <Page />
</Activity>
```

当 Activity 处于<CodeStep step={1}>可见</CodeStep>状态时，它会像平常一样渲染。当 Activity 处于<CodeStep step={2}>隐藏</CodeStep>状态时，它会被卸载，但会保存其状态，并继续以比屏幕上任何可见内容更低的优先级进行渲染。

你可以使用 `Activity` 来保存用户当前没有使用的 UI 部分的状态，或者预先渲染用户下一步很可能会使用的部分。

让我们看看一些改进上面 View Transition 示例的例子。

<Note>

**当 Activity 处于隐藏状态时，Effects 不会挂载。**

当 `<Activity>` 为 `hidden` 时，Effects 会被卸载。从概念上讲，组件被卸载了，但 React 会把状态保存起来以备后用。

实际上，如果你遵循了[你可能不需要 Effect](/learn/you-might-not-need-an-effect)指南，这种行为会符合预期。为了更早地发现有问题的 Effects，我们建议添加 [`<StrictMode>`](/reference/react/StrictMode)，它会更积极地执行 Activity 的卸载和挂载，以捕获任何意外的副作用。

</Note>

### 使用 Activity 恢复状态 {/*restoring-state-with-activity*/}

当用户离开某个页面时，通常会停止渲染旧页面：

```js {6,7}
function App() {
  const { url } = useRouter();

  return (
    <>
      {url === '/' && <Home />}
      {url !== '/' && <Details />}
    </>
  );
}
```

然而，这意味着如果用户返回旧页面，之前的所有状态都会丢失。例如，如果 `<Home />` 页面有一个 `<input>` 输入框，那么当用户离开页面时，这个 `<input>` 会被卸载，用户已经输入的所有文本也会丢失。

Activity 允许你在用户切换页面时保留这些状态，这样当他们 ফিরে लौट回时就可以从离开的地方继续。做法是把树的一部分包裹在 `<Activity>` 中，并切换 `mode`：

```js {6-8}
function App() {
  const { url } = useRouter();

  return (
    <>
      <Activity mode={url === '/' ? 'visible' : 'hidden'}>
        <Home />
      </Activity>
      {url !== '/' && <Details />}
    </>
  );
}
```

有了这个改动，我们可以进一步改进上面的 View Transitions 示例。之前，当你搜索某个视频、选择其中一个、然后返回时，你的搜索过滤条件会丢失。使用 Activity 后，搜索过滤条件会被恢复，你可以从上次中断的地方继续。

试着搜索一个视频，选中它，然后点击“back”：

<Sandpack>

```js src/App.js
import { Activity, ViewTransition } from "react"; import Details from "./Details"; import Home from "./Home"; import { useRouter } from "./router";

export default function App() {
  const { url } = useRouter();

  return (
    // View Transitions 知道 Activity 的存在
    <ViewTransition>
      {/* 在 Activity 中渲染 Home，这样我们就不会丢失状态 */}
      <Activity mode={url === '/' ? 'visible' : 'hidden'}>
        <Home />
      </Activity>
      {url !== '/' && <Details />}
    </ViewTransition>
  );
}
```

```js src/Details.js hidden
import { use, Suspense, ViewTransition } from "react";
import { fetchVideo, fetchVideoDetails } from "./data";
import { Thumbnail, VideoControls } from "./Videos";
import { useRouter } from "./router";
import Layout from "./Layout";
import { ChevronLeft } from "./Icons";

function VideoDetails({id}) {
  // 从 Suspense 的 fallback 动画切换到内容
  return (
    <Suspense
      fallback={
        // 将 fallback 向下滑出。
        <ViewTransition exit="slide-down">
          <VideoInfoFallback />
        </ViewTransition>
      }
    >
      {/* 将内容向上滑入 */}
      <ViewTransition enter="slide-up">
        <VideoInfo id={id} />
      </ViewTransition>
    </Suspense>
  );
}

function VideoInfoFallback() {
  return (
    <>
      <div className="fallback title"></div>
      <div className="fallback description"></div>
    </>
  );
}

export default function Details() {
  const { url, navigateBack } = useRouter();
  const videoId = url.split("/").pop();
  const video = use(fetchVideo(videoId));

  return (
    <Layout
      heading={
        <div
          className="fit back"
          onClick={() => {
            navigateBack("/");
          }}
        >
          <ChevronLeft /> 返回
        </div>
      }
    >
      <div className="details">
        <Thumbnail video={video} large>
          <VideoControls />
        </Thumbnail>
        <VideoDetails id={video.id} />
      </div>
    </Layout>
  );
}

function VideoInfo({ id }) {
  const details = use(fetchVideoDetails(id));
  return (
    <>
      <p className="info-title">{details.title}</p>
      <p className="info-description">{details.description}</p>
    </>
  );
}
```

```js src/Home.js hidden
import { useId, useState, use, useDeferredValue, ViewTransition } from "react";import { Video } from "./Videos";import Layout from "./Layout";import { fetchVideos } from "./data";import { IconSearch } from "./Icons";

function SearchList({searchText, videos}) {
  // 使用 useDeferredValue 激活（“when”）
  const deferredSearchText = useDeferredValue(searchText);
  const filteredVideos = filterVideos(videos, deferredSearchText);
  return (
    <div className="video-list">
      {filteredVideos.length === 0 && (
        <div className="no-results">没有结果</div>
      )}
      <div className="videos">
        {filteredVideos.map((video) => (
          // 为列表中的每一项添加动画（“what”）
          <ViewTransition key={video.id}>
            <Video video={video} />
          </ViewTransition>
        ))}
      </div>
    </div>
  );
}

export default function Home() {
  const videos = use(fetchVideos());
  const count = videos.length;
  const [searchText, setSearchText] = useState('');

  return (
    <Layout heading={<div className="fit">{count} 个视频</div>}>
      <SearchInput value={searchText} onChange={setSearchText} />
      <SearchList videos={videos} searchText={searchText} />
    </Layout>
  );
}

function SearchInput({ value, onChange }) {
  const id = useId();
  return (
    <form className="search" onSubmit={(e) => e.preventDefault()}>
      <label htmlFor={id} className="sr-only">
        搜索
      </label>
      <div className="search-input">
        <div className="search-icon">
          <IconSearch />
        </div>
        <input
          type="text"
          id={id}
          placeholder="搜索"
          value={value}
          onChange={(e) => onChange(e.target.value)}
        />
      </div>
    </form>
  );
}

function filterVideos(videos, query) {
  const keywords = query
    .toLowerCase()
    .split(" ")
    .filter((s) => s !== "");
  if (keywords.length === 0) {
    return videos;
  }
  return videos.filter((video) => {
    const words = (video.title + " " + video.description)
      .toLowerCase()
      .split(" ");
    return keywords.every((kw) => words.some((w) => w.includes(kw)));
  });
}
```

```js src/Icons.js hidden
export function ChevronLeft() {
  return (
    <svg
      className="chevron-left"
      xmlns="http://www.w3.org/2000/svg"
      width="20"
      height="20"
      viewBox="0 0 20 20">
      <g fill="none" fillRule="evenodd" transform="translate(-446 -398)">
        <path
          fill="currentColor"
          fillRule="nonzero"
          d="M95.8838835,240.366117 C95.3957281,239.877961 94.6042719,239.877961 94.1161165,240.366117 C93.6279612,240.854272 93.6279612,241.645728 94.1161165,242.133883 L98.6161165,246.633883 C99.1042719,247.122039 99.8957281,247.122039 100.383883,246.633883 L104.883883,242.133883 C105.372039,241.645728 105.372039,240.854272 104.883883,240.366117 C104.395728,239.877961 103.604272,239.877961 103.116117,240.366117 L99.5,243.982233 L95.8838835,240.366117 Z"
          transform="translate(356.5 164.5)"
        />
        <polygon points="446 418 466 418 466 398 446 398" />
      </g>
    </svg>
  );
}

export function PauseIcon() {
  return (
    <svg
      className="control-icon"
      style={{padding: '4px'}}
      width="100"
      height="100"
      viewBox="0 0 512 512"
      fill="none"
      xmlns="http://www.w3.org/2000/svg">
      <path
        fillRule="evenodd"
        clipRule="evenodd"
        d="M256 0C114.617 0 0 114.615 0 256s114.617 256 256 256 256-114.615 256-256S397.383 0 256 0zm-32 320c0 8.836-7.164 16-16 16h-32c-8.836 0-16-7.164-16-16V192c0-8.836 7.164-16 16-16h32c8.836 0 16 7.164 16 16v128zm128 0c0 8.836-7.164 16-16 16h-32c-8.836 0-16-7.164-16-16V192c0-8.836 7.164-16 16-16h32c8.836 0 16 7.164 16 16v128z"
        fill="currentColor"
      />
    </svg>
  );
}

export function PlayIcon() {
  return (
    <svg
      className="control-icon"
      width="100"
      height="100"
      viewBox="0 0 72 72"
      fill="none"
      xmlns="http://www.w3.org/2000/svg">
      <path
        fillRule="evenodd"
        clipRule="evenodd"
        d="M36 69C54.2254 69 69 54.2254 69 36C69 17.7746 54.2254 3 36 3C17.7746 3 3 17.7746 3 36C3 54.2254 17.7746 69 36 69ZM52.1716 38.6337L28.4366 51.5801C26.4374 52.6705 24 51.2235 24 48.9464V23.0536C24 20.7764 26.4374 19.3295 28.4366 20.4199L52.1716 33.3663C54.2562 34.5034 54.2562 37.4966 52.1716 38.6337Z"
        fill="currentColor"
      />
    </svg>
  );
}
export function Heart({liked, animate}) {
  return (
    <>
      <svg
        className="absolute overflow-visible"
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg">
        <circle
          className={`circle ${liked ? 'liked' : ''} ${animate ? 'animate' : ''}`}
          cx="12"
          cy="12"
          r="11.5"
          fill="transparent"
          strokeWidth="0"
          stroke="currentColor"
        />
      </svg>

      <svg
        className={`heart ${liked ? 'liked' : ''} ${animate ? 'animate' : ''}`}
        viewBox="0 0 24 24"
        fill="none"
        xmlns="http://www.w3.org/2000/svg">
        {liked ? (
          <path
            d="M12 23a.496.496 0 0 1-.26-.074C7.023 19.973 0 13.743 0 8.68c0-4.12 2.322-6.677 6.058-6.677 2.572 0 5.108 2.387 5.134 2.41l.808.771.808-.771C12.834 4.387 15.367 2 17.935 2 21.678 2 24 4.558 24 8.677c0 5.06-7.022 11.293-11.74 14.246a.496.496 0 0 1-.26.074V23z"
            fill="currentColor"
          />
        ) : (
          <path
            fillRule="evenodd"
            clipRule="evenodd"
            d="m12 5.184-.808-.771-.004-.004C11.065 4.299 8.522 2.003 6 2.003c-3.736 0-6 2.558-6 6.677 0 4.47 5.471 9.848 10 13.079.602.43 1.187.82 1.74 1.167A.497.497 0 0 0 12 23v-.003c.09 0 .182-.026.26-.074C16.977 19.97 24 13.737 24 8.677 24 4.557 21.743 2 18 2c-2.569 0-5.166 2.387-5.192 2.413L12 5.184zm-.002 15.525c2.071-1.388 4.477-3.342 6.427-5.47C20.72 12.733 22 10.401 22 8.677c0-1.708-.466-2.855-1.087-3.55C20.316 4.459 19.392 4 18 4c-.726 0-1.63.364-2.5.9-.67.412-1.148.82-1.266.92-.03.025-.037.031-.019.014l-.013.013L12 7.949 9.832 5.88a10.08 10.08 0 0 0-1.33-.977C7.633 4.367 6.728 4.003 6 4.003c-1.388 0-2.312.459-2.91 1.128C2.466 5.826 2 6.974 2 8.68c0 1.726 1.28 4.058 3.575 6.563 1.948 2.127 4.352 4.078 6.423 5.466z"
            fill="currentColor"
          />
        )}
      </svg>
    </>
  );
}

export function IconSearch(props) {
  return (
    <svg width="1em" height="1em" viewBox="0 0 20 20">
      <path
        d="M14.386 14.386l4.0877 4.0877-4.0877-4.0877c-2.9418 2.9419-7.7115 2.9419-10.6533 0-2.9419-2.9418-2.9419-7.7115 0-10.6533 2.9418-2.9419 7.7115-2.9419 10.6533 0 2.9419 2.9418 2.9419 7.7115 0 10.6533z"
        stroke="currentColor"
        fill="none"
        strokeWidth="2"
        fillRule="evenodd"
        strokeLinecap="round"
        strokeLinejoin="round"></path>
    </svg>
  );
}
```

```js src/Layout.js hidden
import {ViewTransition} from 'react'; import { useIsNavPending } from "./router";

export default function Page({ heading, children }) {
  const isPending = useIsNavPending();
  return (
    <div className="page">
      <div className="top">
        <div className="top-nav">
          {/* 基于过渡类型的自定义类。 */}
          <ViewTransition
            name="nav"
            share={{
              'nav-forward': 'slide-forward',
              'nav-back': 'slide-back',
            }}>
            {heading}
          </ViewTransition>
          {isPending && <span className="loader"></span>}
        </div>
      </div>
      {/* 对内容禁用 ViewTransition。 */}
      {/* 内容可以定义自己的 ViewTransition。 */}
      <ViewTransition default="none">
        <div className="bottom">
          <div className="content">{children}</div>
        </div>
      </ViewTransition>
    </div>
  );
}
```

```js src/LikeButton.js hidden
import {useState} from 'react';
import {Heart} from './Icons';

// 这是一个技巧，因为我们实际上没有后端。
// 不同于本地状态，它可以在视频被筛选后依然保留。
const likedVideos = new Set();

export default function LikeButton({video}) {
  const [isLiked, setIsLiked] = useState(() => likedVideos.has(video.id));
  const [animate, setAnimate] = useState(false);
  return (
    <button
      className={`like-button ${isLiked && 'liked'}`}
      aria-label={isLiked ? '取消保存' : '保存'}
      onClick={() => {
        const nextIsLiked = !isLiked;
        if (nextIsLiked) {
          likedVideos.add(video.id);
        } else {
          likedVideos.delete(video.id);
        }
        setAnimate(true);
        setIsLiked(nextIsLiked);
      }}>
      <Heart liked={isLiked} animate={animate} />
    </button>
  );
}
```

```js src/Videos.js hidden
import { useState, ViewTransition } from "react";
import LikeButton from "./LikeButton";
import { useRouter } from "./router";
import { PauseIcon, PlayIcon } from "./Icons";
import { startTransition } from "react";

export function Thumbnail({ video, children }) {
  // 添加一个名称，以便使用共享元素过渡进行动画。
  // 这会使用默认动画，不需要额外的 css。
  return (
    <ViewTransition name={`video-${video.id}`}>
      <div
        aria-hidden="true"
        tabIndex={-1}
        className={`thumbnail ${video.image}`}
      >
        {children}
      </div>
    </ViewTransition>
  );
}

export function VideoControls() {
  const [isPlaying, setIsPlaying] = useState(false);

  return (
    <span
      className="controls"
      onClick={() =>
        startTransition(() => {
          setIsPlaying((p) => !p);
        })
      }
    >
      {isPlaying ? <PauseIcon /> : <PlayIcon />}
    </span>
  );
}

export function Video({ video }) {
  const { navigate } = useRouter();

  return (
    <div className="video">
      <div
        className="link"
        onClick={(e) => {
          e.preventDefault();
          navigate(`/video/${video.id}`);
        }}
      >
        <Thumbnail video={video}></Thumbnail>

        <div className="info">
          <div className="video-title">{video.title}</div>
          <div className="video-description">{video.description}</div>
        </div>
      </div>
      <LikeButton video={video} />
    </div>
  );
}
```


```js src/data.js hidden
const videos = [
  {
    id: '1',
    title: '第一个视频',
    description: '视频描述',
    image: 'blue',
  },
  {
    id: '2',
    title: '第二个视频',
    description: '视频描述',
    image: 'red',
  },
  {
    id: '3',
    title: '第三个视频',
    description: '视频描述',
    image: 'green',
  },
  {
    id: '4',
    title: '第四个视频',
    description: '视频描述',
    image: 'purple',
  },
  {
    id: '5',
    title: '第五个视频',
    description: '视频描述',
    image: 'yellow',
  },
  {
    id: '6',
    title: '第六个视频',
    description: '视频描述',
    image: 'gray',
  },
];

let videosCache = new Map();
let videoCache = new Map();
let videoDetailsCache = new Map();
const VIDEO_DELAY = 1;
const VIDEO_DETAILS_DELAY = 1000;
export function fetchVideos() {
  if (videosCache.has(0)) {
    return videosCache.get(0);
  }
  const promise = new Promise((resolve) => {
    setTimeout(() => {
      resolve(videos);
    }, VIDEO_DELAY);
  });
  videosCache.set(0, promise);
  return promise;
}

export function fetchVideo(id) {
  if (videoCache.has(id)) {
    return videoCache.get(id);
  }
  const promise = new Promise((resolve) => {
    setTimeout(() => {
      resolve(videos.find((video) => video.id === id));
    }, VIDEO_DELAY);
  });
  videoCache.set(id, promise);
  return promise;
}

export function fetchVideoDetails(id) {
  if (videoDetailsCache.has(id)) {
    return videoDetailsCache.get(id);
  }
  const promise = new Promise((resolve) => {
    setTimeout(() => {
      resolve(videos.find((video) => video.id === id));
    }, VIDEO_DETAILS_DELAY);
  });
  videoDetailsCache.set(id, promise);
  return promise;
}
```

```js src/router.js hidden
import {useState, createContext, use, useTransition, useLayoutEffect, useEffect, addTransitionType} from "react";

export function Router({ children }) {
  const [isPending, startTransition] = useTransition();
  const [routerState, setRouterState] = useState({pendingNav: () => {}, url: document.location.pathname});
  function navigate(url) {
    startTransition(() => {
      // “nav forward”的过渡类型
      addTransitionType('nav-forward');
      go(url);
    });
  }
  function navigateBack(url) {
    startTransition(() => {
      // “nav backward”的过渡类型
      addTransitionType('nav-back');
      go(url);
    });
  }

  function go(url) {
    setRouterState({
      url,
      pendingNav() {
        window.history.pushState({}, "", url);
      },
    });
  }

  useEffect(() => {
    function handlePopState() {
      // 这不应该产生动画，因为恢复必须是同步的。
      // 即使它是一个过渡。
      startTransition(() => {
        setRouterState({
          url: document.location.pathname + document.location.search,
          pendingNav() {
            // 空操作。URL 已经更新。
          },
        });
      });
    }
    window.addEventListener("popstate", handlePopState);
    return () => {
      window.removeEventListener("popstate", handlePopState);
    };
  }, []);
  const pendingNav = routerState.pendingNav;
  useLayoutEffect(() => {
    pendingNav();
  }, [pendingNav]);

  return (
    <RouterContext
      value={{
        url: routerState.url,
        navigate,
        navigateBack,
        isPending,
        params: {},
      }}
    >
      {children}
    </RouterContext>
  );
}

const RouterContext = createContext({ url: "/", params: {} });

export function useRouter() {
  return use(RouterContext);
}

export function useIsNavPending() {
  return use(RouterContext).isPending;
}

```

```css src/styles.css hidden
@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Rg.woff2) format("woff2");
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Md.woff2) format("woff2");
  font-weight: 500;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Bd.woff2) format("woff2");
  font-weight: 600;
  font-style: normal;
  font-display: swap;
}

@font-face {
  font-family: Optimistic Text;
  src: url(https://react.dev/fonts/Optimistic_Text_W_Bd.woff2) format("woff2");
  font-weight: 700;
  font-style: normal;
  font-display: swap;
}

* {
  box-sizing: border-box;
}

html {
  background-image: url(https://react.dev/images/meta-gradient-dark.png);
  background-size: 100%;
  background-position: -100%;
  background-color: rgb(64 71 86);
  background-repeat: no-repeat;
  height: 100%;
  width: 100%;
}

body {
  font-family: Optimistic Text, -apple-system, ui-sans-serif, system-ui, sans-serif, Apple Color Emoji, Segoe UI Emoji, Segoe UI Symbol, Noto Color Emoji;
  padding: 10px 0 10px 0;
  margin: 0;
  display: flex;
  justify-content: center;
}

#root {
  flex: 1 1;
  height: auto;
  background-color: #fff;
  border-radius: 10px;
  max-width: 450px;
  min-height: 600px;
  padding-bottom: 10px;
}

h1 {
  margin-top: 0;
  font-size: 22px;
}

h2 {
  margin-top: 0;
  font-size: 20px;
}

h3 {
  margin-top: 0;
  font-size: 18px;
}

h4 {
  margin-top: 0;
  font-size: 16px;
}

h5 {
  margin-top: 0;
  font-size: 14px;
}

h6 {
  margin-top: 0;
  font-size: 12px;
}

code {
  font-size: 1.2em;
}

ul {
  padding-inline-start: 20px;
}

.sr-only {
  position: absolute;
  width: 1px;
  height: 1px;
  padding: 0;
  margin: -1px;
  overflow: hidden;
  clip: rect(0, 0, 0, 0);
  white-space: nowrap;
  border-width: 0;
}

.absolute {
  position: absolute;
}

.overflow-visible {
  overflow: visible;
}

.visible {
  overflow: visible;
}

.fit {
  width: fit-content;
}


/* 布局 */
.page {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.top-hero {
  height: 200px;
  display: flex;
  justify-content: center;
  align-items: center;
  background-image: conic-gradient(
      from 90deg at -10% 100%,
      #2b303b 0deg,
      #2b303b 90deg,
      #16181d 1turn
  );
}

.bottom {
  flex: 1;
  overflow: auto;
}

.top-nav {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 0;
  padding: 0 12px;
  top: 0;
  width: 100%;
  height: 44px;
  color: #23272f;
  font-weight: 700;
  font-size: 20px;
  z-index: 100;
  cursor: default;
}

.content {
  padding: 0 12px;
  margin-top: 4px;
}


.loader {
  color: #23272f;
  font-size: 3px;
  width: 1em;
  margin-right: 18px;
  height: 1em;
  border-radius: 50%;
  position: relative;
  text-indent: -9999em;
  animation: loading-spinner 1.3s infinite linear;
  animation-delay: 200ms;
  transform: translateZ(0);
}

@keyframes loading-spinner {
  0%,
  100% {
    box-shadow: 0 -3em 0 0.2em,
    2em -2em 0 0em, 3em 0 0 -1em,
    2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 -1em, -3em 0 0 -1em,
    -2em -2em 0 0;
  }
  12.5% {
    box-shadow: 0 -3em 0 0, 2em -2em 0 0.2em,
    3em 0 0 0, 2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 -1em, -3em 0 0 -1em,
    -2em -2em 0 -1em;
  }
  25% {
    box-shadow: 0 -3em 0 -0.5em,
    2em -2em 0 0, 3em 0 0 0.2em,
    2em 2em 0 0, 0 3em 0 -1em,
    -2em 2em 0 -1em, -3em 0 0 -1em,
    -2em -2em 0 -1em;
  }
  37.5% {
    box-shadow: 0 -3em 0 -1em, 2em -2em 0 -1em,
    3em 0em 0 0, 2em 2em 0 0.2em, 0 3em 0 0em,
    -2em 2em 0 -1em, -3em 0em 0 -1em, -2em -2em 0 -1em;
  }
  50% {
    box-shadow: 0 -3em 0 -1em, 2em -2em 0 -1em,
    3em 0 0 -1em, 2em 2em 0 0em, 0 3em 0 0.2em,
    -2em 2em 0 0, -3em 0em 0 -1em, -2em -2em 0 -1em;
  }
  62.5% {
    box-shadow: 0 -3em 0 -1em, 2em -2em 0 -1em,
    3em 0 0 -1em, 2em 2em 0 -1em, 0 3em 0 0,
    -2em 2em 0 0.2em, -3em 0 0 0, -2em -2em 0 -1em;
  }
  75% {
    box-shadow: 0em -3em 0 -1em, 2em -2em 0 -1em,
    3em 0em 0 -1em, 2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 0, -3em 0em 0 0.2em, -2em -2em 0 0;
  }
  87.5% {
    box-shadow: 0em -3em 0 0, 2em -2em 0 -1em,
    3em 0 0 -1em, 2em 2em 0 -1em, 0 3em 0 -1em,
    -2em 2em 0 0, -3em 0em 0 0, -2em -2em 0 0.2em;
  }
}

/* LikeButton */
.like-button {
  outline-offset: 2px;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 2.5rem;
  height: 2.5rem;
  cursor: pointer;
  border-radius: 9999px;
  border: none;
  outline: none 2px;
  color: #5e687e;
  background: none;
}

.like-button:focus {
  color: #a6423a;
  background-color: rgba(166, 66, 58, .05);
}

.like-button:active {
  color: #a6423a;
  background-color: rgba(166, 66, 58, .05);
  transform: scaleX(0.95) scaleY(0.95);
}

.like-button:hover {
  background-color: #f6f7f9;
}

.like-button.liked {
  color: #a6423a;
}

/* 图标 */
@keyframes circle {
  0% {
    transform: scale(0);
    stroke-width: 16px;
  }

  50% {
    transform: scale(.5);
    stroke-width: 16px;
  }

  to {
    transform: scale(1);
    stroke-width: 0;
  }
}

.circle {
  color: rgba(166, 66, 58, .5);
  transform-origin: center;
  transition-property: all;
  transition-duration: .15s;
  transition-timing-function: cubic-bezier(.4,0,.2,1);
}

.circle.liked.animate {
  animation: circle .3s forwards;
}

.heart {
  width: 1.5rem;
  height: 1.5rem;
}

.heart.liked {
  transform-origin: center;
  transition-property: all;
  transition-duration: .15s;
  transition-timing-function: cubic-bezier(.4, 0, .2, 1);
}

.heart.liked.animate {
  animation: scale .35s ease-in-out forwards;
}

.control-icon {
  color: hsla(0, 0%, 100%, .5);
  filter:  drop-shadow(0 20px 13px rgba(0, 0, 0, .03)) drop-shadow(0 8px 5px rgba(0, 0, 0, .08));
}

.chevron-left {
  margin-top: 2px;
  rotate: 90deg;
}


/* 视频 */
.thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 8rem;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}

.thumbnail.blue {
  background-image: conic-gradient(at top right, #c76a15, #087ea4, #2b3491);
}

.thumbnail.red {
  background-image: conic-gradient(at top right, #c76a15, #a6423a, #2b3491);
}

.thumbnail.green {
  background-image: conic-gradient(at top right, #c76a15, #388f7f, #2b3491);
}

.thumbnail.purple {
  background-image: conic-gradient(at top right, #c76a15, #575fb7, #2b3491);
}

.thumbnail.yellow {
  background-image: conic-gradient(at top right, #c76a15, #FABD62, #2b3491);
}

.thumbnail.gray {
  background-image: conic-gradient(at top right, #c76a15, #4E5769, #2b3491);
}

.video {
  display: flex;
  flex-direction: row;
  gap: 0.75rem;
  align-items: center;
}

.video .link {
  display: flex;
  flex-direction: row;
  flex: 1 1 0;
  gap: 0.125rem;
  outline-offset: 4px;
  cursor: pointer;
}

.video .info {
  display: flex;
  flex-direction: column;
  justify-content: center;
  margin-left: 8px;
  gap: 0.125rem;
}

.video .info:hover {
  text-decoration: underline;
}

.video-title {
  font-size: 15px;
  line-height: 1.25;
  font-weight: 700;
  color: #23272f;
}

.video-description {
  color: #5e687e;
  font-size: 13px;
}

/* 详情 */
.details .thumbnail {
  position: relative;
  aspect-ratio: 16 / 9;
  display: flex;
  overflow: hidden;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  border-radius: 0.5rem;
  outline-offset: 2px;
  width: 100%;
  vertical-align: middle;
  background-color: #ffffff;
  background-size: cover;
  user-select: none;
}

.video-details-title {
  margin-top: 8px;
}

.video-details-speaker {
  display: flex;
  gap: 8px;
  margin-top: 10px
}

.back {
  display: flex;
  align-items: center;
  margin-left: -5px;
  cursor: pointer;
}

.back:hover {
  text-decoration: underline;
}

.info-title {
  font-size: 1.5rem;
  font-weight: 700;
  line-height: 1.25;
  margin: 8px 0 0 0 ;
}

.info-description {
  margin: 8px 0 0 0;
}

.controls {
  cursor: pointer;
}

.fallback {
  background: #f6f7f8 linear-gradient(to right, #e6e6e6 5%, #cccccc 25%, #e6e6e6 35%) no-repeat;
  background-size: 800px 104px;
  display: block;
  line-height: 1.25;
  margin: 8px 0 0 0;
  border-radius: 5px;
  overflow: hidden;

  animation: 1s linear 1s infinite shimmer;
  animation-delay: 300ms;
  animation-duration: 1s;
  animation-fill-mode: forwards;
  animation-iteration-count: infinite;
  animation-name: shimmer;
  animation-timing-function: linear;
}


.fallback.title {
  width: 130px;
  height: 30px;

}

.fallback.description {
  width: 150px;
  height: 21px;
}

@keyframes shimmer {
  0% {
    background-position: -468px 0;
  }

  100% {
    background-position: 468px 0;
  }
}

.search {
  margin-bottom: 10px;
}
.search-input {
  width: 100%;
  position: relative;
}

.search-icon {
  position: absolute;
  top: 0;
  bottom: 0;
  inset-inline-start: 0;
  display: flex;
  align-items: center;
  padding-inline-start: 1rem;
  pointer-events: none;
  color: #99a1b3;
}

.search-input input {
  display: flex;
  padding-inline-start: 2.75rem;
  padding-top: 10px;
  padding-bottom: 10px;
  width: 100%;
  text-align: start;
  background-color: rgb(235 236 240);
  outline: 2px solid transparent;
  cursor: pointer;
  border: none;
  align-items: center;
  color: rgb(35 39 47);
  border-radius: 9999px;
  vertical-align: middle;
  font-size: 15px;
}

.search-input input:hover, .search-input input:active {
  background-color: rgb(235 236 240/ 0.8);
  color: rgb(35 39 47/ 0.8);
}

/* 首页 */
.video-list {
  position: relative;
}

.video-list .videos {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  overflow-y: auto;
  height: 100%;
}
```

```css src/animations.css
/* 不需要额外的动画 */









/* 下面是之前定义的动画 */






/* 用于让 Suspense 的 fallback 向下滑动的切换动画 */
::view-transition-old(.slide-down) {
    animation: 150ms ease-out both fade-out, 150ms ease-out both slide-down;
}

::view-transition-new(.slide-up) {
    animation: 210ms ease-in 150ms both fade-in, 400ms ease-in both slide-up;
}

/* 通过 transition type 添加的带类名的视图过渡动画 */
::view-transition-old(.slide-forward) {
    /* 向前滑动时，“旧”页面应向左滑出。 */
    animation: 150ms cubic-bezier(0.4, 0, 1, 1) both fade-out,
    400ms cubic-bezier(0.4, 0, 0.2, 1) both slide-to-left;
}

::view-transition-new(.slide-forward) {
    /* 向前滑动时，“新”页面应从右侧滑入。 */
    animation: 210ms cubic-bezier(0, 0, 0.2, 1) 150ms both fade-in,
    400ms cubic-bezier(0.4, 0, 0.2, 1) both slide-from-right;
}

::view-transition-old(.slide-back) {
    /* 向后滑动时，“旧”页面应向右滑出。 */
    animation: 150ms cubic-bezier(0.4, 0, 1, 1) both fade-out,
    400ms cubic-bezier(0.4, 0, 0.2, 1) both slide-to-right;
}

::view-transition-new(.slide-back) {
    /* 向后滑动时，“新”页面应从左侧滑入。 */
    animation: 210ms cubic-bezier(0, 0, 0.2, 1) 150ms both fade-in,
    400ms cubic-bezier(0.4, 0, 0.2, 1) both slide-from-left;
}

/* 支持上面动画的关键帧。 */
@keyframes slide-up {
    from {
        transform: translateY(10px);
    }
    to {
        transform: translateY(0);
    }
}

@keyframes slide-down {
    from {
        transform: translateY(0);
    }
    to {
        transform: translateY(10px);
    }
}

@keyframes fade-in {
    from {
        opacity: 0;
    }
}

@keyframes fade-out {
    to {
        opacity: 0;
    }
}

@keyframes slide-to-right {
    to {
        transform: translateX(50px);
    }
}

@keyframes slide-from-right {
    from {
        transform: translateX(50px);
    }
    to {
        transform: translateX(0);
    }
}

@keyframes slide-to-left {
    to {
        transform: translateX(-50px);
    }
}

@keyframes slide-from-left {
    from {
        transform: translateX(-50px);
    }
    to {
        transform: translateX(0);
    }
}

/* 默认的 .slow-fade。 */
::view-transition-old(.slow-fade) {
    animation-duration: 500ms;
}

::view-transition-new(.slow-fade) {
    animation-duration: 500ms;
}
```

```js src/index.js hidden
import React, {StrictMode} from 'react';
import {createRoot} from 'react-dom/client';
import './styles.css';
import './animations.css';

import App from './App';
import {Router} from './router';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <Router>
      <App />
    </Router>
  </StrictMode>
);
```

```json package.json hidden
{
  "dependencies": {
    "react": "canary",
    "react-dom": "canary",
    "react-scripts": "latest"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

</Sandpack>

### 带 Activity 的服务端渲染 {/*server-side-rendering-with-activity*/}

当在使用服务端渲染（SSR）的页面上使用 Activity 时，会有额外的优化。

如果页面的一部分以 `mode="hidden"` 渲染，那么它将不会包含在 SSR 响应中。相反，React 会在页面其余部分进行 hydration 的同时，为 Activity 内部的内容安排一次客户端渲染，从而优先处理屏幕上可见的内容。

对于以 `mode="visible"` 渲染的 UI 部分，React 会降低 Activity 内部内容的 hydration 优先级，类似于 Suspense 内容以较低优先级进行 hydration 的方式。如果用户与页面交互，我们会在需要时优先对该边界内进行 hydration。

这些都是高级用例，但它们展示了 Activity 所带来的额外收益。

### Activity 的未来模式 {/*future-modes-for-activity*/}

未来，我们可能会为 Activity 添加更多模式。

例如，一个常见的用例是渲染模态框，其中之前“非活动”的页面会显示在“活动”的模态视图后面。`hidden` 模式不适用于这个用例，因为它既不可见，也不会包含在 SSR 中。

相反，我们正在考虑一种新模式，它会让内容保持可见——并且包含在 SSR 中——但保持未挂载状态并降低更新优先级。这个模式可能还需要“暂停” DOM 更新，因为在模态框打开时看到后台内容不断更新可能会令人分心。

我们还在考虑为 Activity 提供另一种模式：如果内存使用过多，能够自动销毁隐藏 Activity 的状态。由于组件本身已经未挂载，也许销毁应用中最近最少使用的隐藏部分的状态，而不是消耗过多资源，会更合适。

这些都是我们仍在探索的方向，我们会随着进展分享更多内容。关于 Activity 当前包含的内容，更多信息请[查看文档](/reference/react/Activity)。

---

# 开发中的特性 {/*features-in-development*/}

我们也在开发一些特性，以帮助解决下面这些常见问题。

在我们迭代可能的解决方案时，你可能会看到一些我们正在测试的潜在 API，随着我们落地某些 PR，这些内容也会被分享出来。请记住，在我们尝试不同想法的过程中，通常会在试验后更改或移除不同的解决方案。

如果我们过早分享正在开发的解决方案，这可能会在社区中引起变动和困惑。为了在保持透明与减少混乱之间取得平衡，我们会分享当前正在为之开发解决方案的问题，但不会提前分享我们设想中的具体方案。

随着这些特性的推进，我们会在博客上发布公告，并附上文档，这样你就可以尝试它们。

## React 性能轨道 {/*react-performance-tracks*/}

我们正在为性能分析器开发一组新的自定义轨道，使用浏览器 API 来[允许添加自定义轨道](https://developer.chrome.com/docs/devtools/performance/extension)，以提供更多关于你的 React 应用性能的信息。

这个特性仍在开发中，所以我们还没有准备好发布文档并将其作为实验性特性完整发布。不过，当你使用 React 的实验版本时，可以抢先预览，它会自动将性能轨道添加到性能分析结果中：

<div style={{display: 'flex', justifyContent: 'center', marginBottom: '1rem'}}>
  <picture >
      <source srcset="/images/blog/react-labs-april-2025/perf_tracks.png" />
      <img className="w-full light-image" src="/images/blog/react-labs-april-2025/perf_tracks.webp" />
  </picture>
  <picture >
      <source srcset="/images/blog/react-labs-april-2025/perf_tracks_dark.png" />
      <img className="w-full dark-image" src="/images/blog/react-labs-april-2025/perf_tracks_dark.webp" />
  </picture>
</div>

我们计划解决一些已知问题，例如性能问题，以及调度器轨道并不总是能跨越被 Suspense 挂起的树“连接”工作，因此它还不太适合尝试。我们也仍在收集早期采用者的反馈，以改进这些轨道的设计和可用性。

一旦我们解决这些问题，就会发布实验性文档，并宣布它已准备好尝试。

---

## 自动化 Effect 依赖项 {/*automatic-effect-dependencies*/}

当我们发布 Hooks 时，我们有三个目标：

- **在组件之间共享代码**：Hooks 取代了 render props 和高阶组件等模式，让你可以在不改变组件层级结构的情况下复用有状态逻辑。
- **从函数而非生命周期的角度思考**：Hooks 让你可以根据相关的功能片段（例如建立订阅或获取数据）把一个组件拆分成更小的函数，而不是被迫按照生命周期方法来拆分。
- **支持提前编译**：Hooks 的设计支持提前编译，并减少由生命周期方法和类的限制所导致的、容易引发意外去优化的陷阱。

自发布以来，Hooks 在*组件之间共享代码*方面取得了成功。现在，Hooks 已成为组件之间共享逻辑的首选方式，render props 和高阶组件的使用场景也减少了。Hooks 也成功支持了像 Fast Refresh 这样的功能，而这些是类组件无法实现的。

### Effect 可能很难理解 {/*effects-can-be-hard*/}

不幸的是，一些 Hooks 仍然很难从函数而非生命周期的角度去思考。尤其是 Effects 仍然很难理解，也是我们从开发者那里听到的最常见痛点。去年，我们花了大量时间研究 Effects 是如何被使用的，以及这些使用场景如何简化并更容易理解。

我们发现，很多时候，困惑来自于在你其实不需要用 Effect 时却使用了它。[你可能不需要 Effect](/learn/you-might-not-need-an-effect) 这篇指南覆盖了许多不适合使用 Effect 的情况。然而，即使 Effect 是某个问题的正确选择，Effects 仍然可能比类组件生命周期更难理解。

我们认为，困惑的原因之一是，开发者往往从 _组件_ 的角度来理解 Effects（像生命周期一样），而不是从 _Effect_ 的角度（Effect 本身在做什么）。

让我们看一个[文档中的示例](/learn/lifecycle-of-reactive-effects#thinking-from-the-effects-perspective)：

```js
useEffect(() => {
  // 你的 Effect 连接到由 roomId 指定的房间...
  const connection = createConnection(serverUrl, roomId);
  connection.connect();
  return () => {
    // ...直到它断开连接
    connection.disconnect();
  };
}, [roomId]);
```

很多用户会把这段代码理解为：“在挂载时，连接到 roomId；每当 `roomId` 变化时，就断开旧房间并重新创建连接。”然而，这其实是在从组件的生命周期角度思考，这意味着你需要考虑组件生命周期中的每一种状态，才能正确编写 Effect。从组件视角来使用时，Effect 看起来比类生命周期更难理解，这也是可以理解的。

### 不带依赖项的 Effects {/*effects-without-dependencies*/}

相反，更好的方式是从 Effect 的角度去思考。Effect 并不知道组件生命周期。它只描述如何开始同步，以及如何停止同步。当用户以这种方式思考 Effects 时，他们往往更容易编写这些 Effects，而且即使需要多次启动和停止，它们也更具韧性。

我们花了一些时间研究为什么人们会从组件视角来理解 Effects，我们认为其中一个原因是依赖数组。因为你必须写它，它就在那里，映入眼帘，提醒你你在对什么“作出反应”，并诱导你进入“当这些值变化时就执行这个”的心智模型。

当我们发布 Hooks 时，我们就知道可以通过提前编译让它们更易用。借助 React Compiler，在大多数情况下你现在都可以不必自己编写 `useCallback` 和 `useMemo`。对于 Effects，编译器可以为你插入依赖项：

```js
useEffect(() => {
  const connection = createConnection(serverUrl, roomId);
  connection.connect();
  return () => {
    connection.disconnect();
  };
}); // 编译器插入的依赖项。
```

通过这段代码，React Compiler 可以为你推导依赖项并自动插入，这样你就不需要看到或编写它们了。结合[IDE 扩展](#compiler-ide-extension)和[`useEffectEvent`](/reference/react/useEffectEvent)等特性，我们可以提供 CodeLens，展示编译器在你需要调试时插入了什么，或者在你想通过移除某个依赖项来优化时展示出来。这有助于强化编写 Effects 时正确的心智模型：Effects 可以在任何时候运行，以将你的组件或 Hook 的状态与其他内容同步。

我们的希望是，自动插入依赖项不仅更易于编写，也能通过迫使你从 Effect 的作用而不是组件生命周期来思考，从而让它们更容易理解。

---

## Compiler IDE 扩展 {/*compiler-ide-extension*/}

在 2025 年晚些时候，[我们分享了](/blog/2025/10/07/react-compiler-1) React Compiler 的首个稳定版本，并且我们还在持续投入，发布更多改进。

我们也已经开始探索如何利用 React Compiler 提供能够提升代码理解和调试的信息。我们开始探索的一个想法，是一个新的、基于 LSP 的 React IDE 扩展，由 React Compiler 提供支持，类似于 [Lauren Tan 的 React Conf 演讲](https://conf2024.react.dev/talks/5)中使用的扩展。

我们的设想是，可以利用编译器的静态分析，在你的 IDE 中直接提供更多信息、建议和优化机会。例如，我们可以提供针对违反 React 规则代码的诊断信息，悬停提示用于显示组件和 Hooks 是否被编译器优化过，或者提供 CodeLens 来查看[自动插入的 Effect 依赖项](#automatic-effect-dependencies)。

这个 IDE 扩展仍处于早期探索阶段，但我们会在后续更新中分享进展。

---

## Fragment 引用 {/*fragment-refs*/}

许多 DOM API，例如事件管理、定位和焦点相关的 API，在使用 React 编写时都很难组合。这常常会导致开发者求助于 Effects、管理多个 Ref，或者使用像 `findDOMNode` 这样的 API（已在 React 19 中移除）。

我们正在探索为 Fragment 添加 refs，使其指向一组 DOM 元素，而不仅仅是单个元素。我们的希望是，这能简化对多个子元素的管理，并在调用 DOM API 时更容易编写可组合的 React 代码。

Fragment refs 仍在研究中。等我们更接近完成最终 API 时，会分享更多内容。

---

## 手势动画 {/*gesture-animations*/}

我们也在研究如何增强 View Transitions，以支持手势动画，例如通过滑动打开菜单，或在照片轮播中滚动浏览。

手势带来了新的挑战，原因有以下几点：

- **手势是连续的**：当你滑动时，动画与手指停留的位置和时间绑定，而不是触发后一直运行到完成。
- **手势不会单向完成**：当你松开手指时，手势动画可以继续运行到结束，也可以回到原始状态（例如你只部分打开菜单时），这取决于你滑动了多远。
- **手势会颠倒旧状态和新状态**：在动画过程中，你希望正在离开的页面保持“活着”并可交互。这颠倒了浏览器 View Transition 的模型，其中“旧”状态是快照，而“新”状态是实时 DOM。

我们认为我们已经找到了一个效果不错的方法，并可能引入一个用于触发手势过渡的新 API。就目前而言，我们的重点是发布 `<ViewTransition>`，之后再回过头来研究手势。

## 并发型 Store {/*concurrent-stores*/}

当我们随着并发渲染发布 React 18 时，我们也发布了 `useSyncExternalStore`，这样那些不使用 React state 或 context 的外部 store 库，就可以在 store 更新时强制同步渲染，从而[支持并发渲染](https://github.com/reactwg/react-18/discussions/70)。

不过，使用 `useSyncExternalStore` 也是有代价的，因为它会强制退出诸如 transitions 之类的并发特性，并强制现有内容显示 Suspense 回退。

现在 React 19 已经发布，我们正在重新审视这个问题领域，以创建一个原语，借助 `use` API 充分支持并发外部 store：

```js
const value = use(store);
```

我们的目标是在渲染期间读取外部状态而不会出现撕裂，并且能够与 React 提供的所有并发特性无缝协作。

这项研究仍处于早期阶段。当我们取得更多进展时，我们会分享更多内容，以及新的 API 会是什么样子。

---

_感谢 [Aurora Scharff](https://bsky.app/profile/aurorascharff.no)、[Dan Abramov](https://bsky.app/profile/danabra.mov)、[Eli White](https://twitter.com/Eli_White)、[Lauren Tan](https://bsky.app/profile/no.lol)、[Luna Wei](https://github.com/lunaleaps)、[Matt Carroll](https://twitter.com/mattcarrollcode)、[Jack Pope](https://jackpope.me)、[Jason Bonta](https://threads.net/someextent)、[Jordan Brown](https://github.com/jbrown215)、[Jordan Eldredge](https://bsky.app/profile/capt.dev)、[Mofei Zhang](https://threads.net/z_mofei)、[Sebastien Lorber](https://bsky.app/profile/sebastienlorber.com)、[Sebastian Markbåge](https://bsky.app/profile/sebmarkbage.calyptus.eu) 和 [Tim Yung](https://github.com/yungsters) 对这篇文章的审阅。_
