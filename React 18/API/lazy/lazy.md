# lazy

`lazy` 能够让你在组件第一次被渲染之前延迟加载组件的代码。

## 参考 

### lazy(load)

在组件外部调用 `lazy`，以声明一个懒加载的 React 组件:

```jsx
import { lazy } from 'react';

const MarkdownPreview = lazy(() => import('./MarkdownPreview.js'));
```

#### 参数
+ `load`: 一个返回 `Promise` 或另一个 `thenable`（具有 `then` 方法的类 `Promise` 对象）的函数。在你尝试第一次渲染返回的组件之前，React 是不会调用 `load` 函数的。在 React 首次调用 `load` 后，它将等待其解析，然后将解析值渲染成 React 组件。返回的 `Promise` 和 `Promise` 的解析值都将被缓存，因此 React 不会多次调用 `load` 函数。如果 `Promise` 被拒绝，则 React 将抛出拒绝原因给最近的错误边界处理。

#### 返回值 
`lazy` 返回一个 React 组件，你可以在 `fiber` 树中渲染。当懒加载组件的代码仍在加载时，尝试渲染它将会处于 暂停 状态。使用 `<Suspense>` 可以在其加载时显示一个正在加载的提示。

### load 函数 

#### 参数
`load` 不接收任何参数。

#### 返回值 
你需要返回一个 `Promise` 或其他 `thenable` （具有 `then` 方法的类 `Promise` 对象）。它最终需要解析为有效的 React 组件类型，例如函数、`memo` 或 `forwardRef` 组件。

## 使用方法 

### 使用 `Suspense` 实现懒加载组件 
通常，你可以使用静态 `import` 声明来导入组件：

```jsx
import MarkdownPreview from './MarkdownPreview.js';
```

如果想在组件第一次渲染前延迟加载这个组件的代码，请替换成以下导入方式：

```jsx
import { lazy } from 'react';

const MarkdownPreview = lazy(() => import('./MarkdownPreview.js'));
```

此代码依赖于 动态 `import()`，可能需要你的打包工具或框架提供支持。
现在你的组件代码可以按需加载，同时你需要指定在它正在加载时应该显示什么。你可以通过将懒加载组件或其任何父级包装到 `<Suspense>` 边界中来实现：

```jsx
<Suspense fallback={<Loading />}>
  <h2>Preview</h2>
  <MarkdownPreview />
 </Suspense>
```

在这个例子中，`MarkdownPreview` 的代码只有在你尝试渲染它时才会被加载。如果 `MarkdownPreview` 还没有加载完成，将显示 `Loading`。请尝试勾选复选框：

#### App.js
```jsx
import { useState, Suspense, lazy } from 'react';
import Loading from './Loading.js';

const MarkdownPreview = lazy(() => delayForDemo(import('./MarkdownPreview.js')));

export default function MarkdownEditor() {
  const [showPreview, setShowPreview] = useState(false);
  const [markdown, setMarkdown] = useState('Hello, **world**!');
  return (
    <>
      <textarea value={markdown} onChange={e => setMarkdown(e.target.value)} />
      <label>
        <input type="checkbox" checked={showPreview} onChange={e => setShowPreview(e.target.checked)} />
        Show preview
      </label>
      <hr />
      {showPreview && (
        <Suspense fallback={<Loading />}>
          <h2>Preview</h2>
          <MarkdownPreview markdown={markdown} />
        </Suspense>
      )}
    </>
  );
}

// 添加一个固定的延迟时间，以便你可以看到加载状态
function delayForDemo(promise) {
  return new Promise(resolve => {
    setTimeout(resolve, 2000);
  }).then(() => promise);
}
```

#### Loading.js
```jsx
export default function Loading() {
  return <p><i>Loading...</i></p>;
}
```

#### MarkdownPreview.js
```jsx
import { Remarkable } from 'remarkable';

const md = new Remarkable();

export default function MarkdownPreview({ markdown }) {
  return (
    <div
      className="content"
      dangerouslySetInnerHTML={{__html: md.render(markdown)}}
    />
  );
}
```

#### App.js
```jsx
import { useState, Suspense, lazy } from 'react';
import Loading from './Loading.js';

const MarkdownPreview = lazy(() => delayForDemo(import('./MarkdownPreview.js')));

export default function MarkdownEditor() {
  const [showPreview, setShowPreview] = useState(false);
  const [markdown, setMarkdown] = useState('Hello, **world**!');
  return (
    <>
      <textarea value={markdown} onChange={e => setMarkdown(e.target.value)} />
      <label>
        <input type="checkbox" checked={showPreview} onChange={e => setShowPreview(e.target.checked)} />
        Show preview
      </label>
      <hr />
      {showPreview && (
        <Suspense fallback={<Loading />}>
          <h2>Preview</h2>
          <MarkdownPreview markdown={markdown} />
        </Suspense>
      )}
    </>
  );
}

// 添加一个固定的延迟时间，以便你可以看到加载状态
function delayForDemo(promise) {
  return new Promise(resolve => {
    setTimeout(resolve, 2000);
  }).then(() => promise);
}
```

这个示例代码人为地延迟了加载。在你下次取消选中并重新选中复选框时，`Preview` 将被缓存，因此不会出现加载状态。要再次查看加载状态，请在示例中单击“Reset”。

## 疑难解答 

### 我的 `lazy` 组件状态意外重置
不要在其他组件 内部 声明 `lazy` 组件：

```jsx
import { lazy } from 'react';

function Editor() {
  // 🔴 Bad: 这将导致在重新渲染时重置所有状态
  const MarkdownPreview = lazy(() => import('./MarkdownPreview.js'));
  // ...
}
```

### 相反，总是在模块的顶层声明它们：
```jsx
import { lazy } from 'react';

// ✅ Good: 将 lazy 组件声明在组件外部
const MarkdownPreview = lazy(() => import('./MarkdownPreview.js'));

function Editor() {
  // ...
}
```