# Suspense

`<Suspense>` 允许你显示一个退路方案（fallback）直到它的子组件完成加载。

## 参考 

```jsx
<Suspense>
```

### 参数 
+ `children`：实际的 UI 渲染内容。如果 `children` 在渲染中挂起（`suspend`），`Suspense` 边界将切换到渲染 `fallback`。
+ `fallback`：一个在实际的 UI 未渲染完成时代替其渲染的备用 UI。任何有效的 React Node 都被接受，但实际上退路方案（`fallback`）是一个轻量的占位符，例如加载中图标或者骨架屏。`Suspense` 将自动切换到 `fallback` 当 `children` 挂起（`suspend`）时，并在数据就位时切换回 `children`。如果 `fallback` 在渲染中挂起（`suspend`），它将自动激活最近的 `Suspense` 边界。

### 注意事项 
+ React 不会保留任何在首次挂载前被挂起（`suspend`）的渲染的任何状态。当组件完成加载后，React 将从头开始重新尝试渲染挂起（`suspend`）的组件树。
+ 如果 `Suspense` 正在显示 React 组件树中的内容，但是被再次挂起（`suspend`），`fallback` 将再次显示，除非导致它的更新是由 `startTransition` 或 `useDeferredValue` 发起的。
+ 如果 React 因已经可见的内容被再次挂起（`suspend`）而需要隐藏它，它将清理内容树中的 `layout Effects`。当内容可以被再次展示时，React 将重新触发 `layout Effects`。这确保了测量 DOM 布局的 `Effects` 不会在内容不可见时运行。
+ React 带有内置的优化，例如 *Streaming Server Rendering* 和 *Selective Hydration*，它们已经与 Suspense 集成。阅读 架构概述 并观看 技术讲座 以了解更多。

## 用法 

### 当内容正在加载时显示退路方案（fallback）
你可以使用 `Suspense` 边界包裹你应用的任何部分：

```jsx
<Suspense fallback={<Loading />}>
  <Albums />
</Suspense>
```

React 将展示你的  退路方案（`fallback`）  直到  子组件  需要的所有代码和数据都加载完成。

在下面的例子中，`Albums` 组件在获取专辑列表时被 挂起 。在它准备好渲染之前，React 切换到最近的 `Suspense` 边界来显示退路方案（`fallback`） —— 你的 `Loading` 组件。然后，当数据加载完成时，React 会隐藏 `Loading` 退路方案（`fallback`）并渲染带有数据的 `Albums` 组件。

```jsx
import { Suspense } from 'react';
import Albums from './Albums.js';

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Suspense fallback={<Loading />}>
        <Albums artistId={artist.id} />
      </Suspense>
    </>
  );
}

function Loading() {
  return <h2>🌀 Loading...</h2>;
}
```

### 只有启用了 Suspense 的数据源才会激活 Suspense 组件，它们包括：
+ 使用支持 `Suspense` 的框架 `Relay` 和 `Next.js`。
+ 使用 `lazy` 懒加载组件代码。

Suspense 无法 检测在 Effect 或事件处理程序中获取数据的情况。

在上面的 `Albums` 组件中，正确的数据加载方法取决于你使用的框架。如果你使用了支持 `Suspense` 的框架，你会在其数据获取文档中找到详细信息。

目前还不支持脱离框架使用支持 `Suspense` 的数据获取。实现支持 `Suspense` 的数据源的要求是不稳定的，也没有文档。用于将数据源与 `Suspense` 集成的官方 API 将在未来的 React 版本中发布。

### 同时展示内容 
默认情况下，`Suspense` 内部的整个组件树都被视为一个单独的单元。例如，即使 *只有一个* 组件挂起（`suspend`）等待某些数据，*所有* 的组件都将被替换为加载中指示器：

```jsx
<Suspense fallback={<Loading />}>
  <Biography />
  <Panel>
    <Albums />
  </Panel>
</Suspense>
```

然后，当它们都准备好显示时，它们将一起被显示。

在下面的例子中，`Biography` 和 `Albums` 都会获取一些数据。但是，因为它们都被分组在一个单独的 `Suspense` 边界下，这些组件总是一起“浮现”。

#### ArtistPage.js
```jsx
import { Suspense } from 'react';
import Albums from './Albums.js';
import Biography from './Biography.js';
import Panel from './Panel.js';

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Suspense fallback={<Loading />}>
        <Biography artistId={artist.id} />
        <Panel>
          <Albums artistId={artist.id} />
        </Panel>
      </Suspense>
    </>
  );
}

function Loading() {
  return <h2>🌀 Loading...</h2>;
}
```

#### Panel.js
```jsx
export default function Panel({ children }) {
  return (
    <section className="panel">
      {children}
    </section>
  );
}
```

加载数据的组件不必是 `Suspense` 边界的直接子组件。例如，你可以将 `Biography` 和 `Albums` 移动到一个新的 `Details` 组件中。这不会改变其行为。`Biography` 和 `Albums` 共享最近的父级 `<Suspense>` 边界，因此它们的显示是协同进行的。

```jsx
<Suspense fallback={<Loading />}>
  <Details artistId={artist.id} />
</Suspense>

function Details({ artistId }) {
  return (
    <>
      <Biography artistId={artistId} />
      <Panel>
        <Albums artistId={artistId} />
      </Panel>
    </>
  );
}
```

### 逐步加载内容 

当一个组件挂起（`suspend`）时，最近的父级 `Suspense` 组件会显示退路方案（`fallback`）。这允许你嵌套多个 `Suspense` 组件来创建一个加载序列。每个 `Suspense` 边界的退路方案（`fallback`）都会在下一级内容可用时填充。例如，你可以给专辑列表设置自己的退路方案（`fallback`）

```jsx
<Suspense fallback={<BigSpinner />}>
  <Biography />
  <Suspense fallback={<AlbumsGlimmer />}>
    <Panel>
      <Albums />
    </Panel>
  </Suspense>
</Suspense>
```

通过这个改变，显示 `Biography` 不需要“等待” `Albums` 加载。
1. 如果 `Biography` 没有加载完成，`BigSpinner` 会显示在整个内容区域的位置。
2. 一旦 `Biography` 加载完成，`BigSpinner` 会被内容替换。
3. 如果 `Albums` 没有加载完成，`AlbumsGlimmer` 会显示在 `Albums` 和它的父级 `Panel` 的位置。
4. 最后，一旦 `Albums` 加载完成，它会替换 `AlbumsGlimmer`。

#### ArtistPage.js
```jsx
import { Suspense } from 'react';
import Albums from './Albums.js';
import Biography from './Biography.js';
import Panel from './Panel.js';

export default function ArtistPage({ artist }) {
  return (
    <>
      <h1>{artist.name}</h1>
      <Suspense fallback={<BigSpinner />}>
        <Biography artistId={artist.id} />
        <Suspense fallback={<AlbumsGlimmer />}>
          <Panel>
            <Albums artistId={artist.id} />
          </Panel>
        </Suspense>
      </Suspense>
    </>
  );
}

function BigSpinner() {
  return <h2>🌀 Loading...</h2>;
}

function AlbumsGlimmer() {
  return (
    <div className="glimmer-panel">
      <div className="glimmer-line" />
      <div className="glimmer-line" />
      <div className="glimmer-line" />
    </div>
  );
}
```

#### Panel.js
```jsx
export default function Panel({ children }) {
  return (
    <section className="panel">
      {children}
    </section>
  );
}
```

`Suspense` 边界允许你协调 UI 的哪些部分应该总是一起“浮现”，以及哪些部分应该按照加载状态的序列逐步显示更多内容。你可以在树的任何位置添加、移动或删除 `Suspense` 边界，而不会影响应用程序的其余的行为。

不要在每个组件周围都放置 `Suspense` 边界。为了提供更好的用户体验，`Suspense` 边界的粒度应该与期望的加载粒度相匹配。如果你与设计师合作，请询问他们应该放置加载状态的位置——他们很可能已经在设计线框图中包含了它们。

