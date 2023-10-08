# hydrateRoot

`hydrateRoot` 函数允许你在先前由 react-dom/server 生成的浏览器 HTML DOM 节点中展示 React 组件。

```jsx
const root = hydrateRoot(domNode, reactNode, options?)
```

+ 参考
  + hydrateRoot(domNode, reactNode, options?)
  + root.render(reactNode)
  + root.unmount()
+ 用法
  + hydrate 服务端渲染的 HTML
  + hydrate 整个文档
  + 抑制不可避免的 hydrate 处理不匹配错误
  + 处理不同的客户端和服务端内容
  + 更新 hydrate 根组件

## 参考 

### `hydrateRoot(domNode, reactNode, options?)`

用 `hydrateRoot` 函数将 React 连接到由 React 在服务端环境中渲染的现有 HTML 中。

```jsx
import { hydrateRoot } from 'react-dom/client';

const domNode = document.getElementById('root');
const root = hydrateRoot(domNode, reactNode);
```

React 将会连接到内部有 domNode 的 HTML 上，然后接管其中的 `domNode`。一个完全由 React 构建的应用只会在其根组件中调用一次 `hydrateRoot` 方法。

#### 参数
+ `domNode`：一个在服务器端渲染时呈现为根元素的 DOM 元素。
+ `reactNode`：用于渲染已存在 HTML 的“React 节点”。这个节点通常是一些类似于 `<App />` 的 JSX，它会在 ReactDOM Server 端使用类似于 `renderToPipeableStream(<App />)` 的方法进行渲染。
+ *可选*  `options`：一个包含此 React 根元素选项的对象。
  + *可选* `onRecoverableError`：当 React 自动从错误中恢复时调用的回调函数。
  + *可选* `identifierPrefix`：字符串前缀，用于标识由 `useId` 生成的 ID ，可以避免在同一页面上使用多个 React 根元素时出现冲突。必须与服务端使用的前缀相同。

#### 返回值
hydrateRoot 返回一个包含两个方法的对象 `render` 和 `unmount`。

#### 警告 
+ `hydrateRoot()` 期望渲染内容与服务端渲染的内容完全相同。你应该将不匹配视为错误并进行修复。
+ 在开发模式下，React 会在 `hydrate` 期间发出不匹配警告。在不匹配的情况下，不能保证内容差异会被修补。出于性能原因，这很重要，因为在大多数应用程序中，不匹配很少见，因此验证所有标记将是昂贵而不可行的。
+ 你的应用程序可能只有一个 `hydrateRoot()` 函数调用。如果你使用框架，则可能会为你完成此调用。
+ 如果你的应用程序是客户端渲染，并且没有已渲染好的 HTML，则不支持使用 `hydrateRoot()`。请改用 `createRoot()`。

### `root.render(reactNode)`

使用 `root.render` 更新一个 `hydrate` 根组件中的 React 组件来渲染浏览器端 DOM 元素。

```jsx
root.render(<App />);
```

React 将会在 hydrate `root` 中更新 `<App />`。

#### 参数 
`reactNode`：你想要更新的 “React 节点”。通常这会是一段JSX代码，例如 `<App />`，但你也可以传递一个通过 `createElement()` 创建的 React 元素，一个字符串，一个数字，`null` 值 或者 `undefined` 值。

#### 返回值 
`root.render` 返回 `undefined` 值。

#### 警告 
如果你在根节点还没有完成 `hydrate` 的情况下调用了 `root.render`，React 将清除现有的服务端渲染 HTML 内容，并将整个根节点切换到客户端渲染。

### `root.unmount()`

调用 `root.unmount` 来销毁 React 根节点内的渲染树。

完全使用 React 构建的应用通常不会有任何调用 `root.unmount` 的情况。

这主要适用于 React 根节点的 DOM 节点（或其任何祖先节点）可能会被其他代码从 DOM 中移除的情况。例如，想象一下一个 `jQuery` 标签面板，它会将非活动标签从 DOM 中移除。如果一个标签被移除，其内部的所有内容（包括其中的 React 根节点）也将从 DOM 中移除。你需要调用 `root.unmount` 来告诉 React “停止”管理已移除根节点的内容。否则，已移除根节点内的组件将无法清理和释放已使用的资源，例如订阅。

调用 `root.unmount` 将卸载根节点中的所有组件，并“分离” React 与根 DOM 节点之间的连接，包括删除树中的任何事件处理程序或状态。

#### 参数 
`root.unmount` 不接受任何参数。

#### 返回值 
`render` 返回 `undefined` 值。

#### 警告 
+ 调用 `root.unmount` 将卸载树中的所有组件，并“分离” React 与根 DOM 节点之间的连接
+ 一旦你调用 `root.unmount`，就不能再在根节点上调用 `root.render`。在未挂载的根节点上尝试调用 `root.render` 将抛出“不能更新未挂载的根节点”的错误。

## 用法 

### hydrate 服务端渲染的 HTML 
如果你的应用程序的 HTML 是由 react-dom/server 生成的，你需要在客户端上进行 hydrate。

```jsx
import { hydrateRoot } from 'react-dom/client';

hydrateRoot(document.getElementById('root'), <App />);
```

对于你的应用程序来说，这将  hydrate 你的服务端 HTML 来复苏里面的 浏览器 DOM 节点和 React 组件。通常，你只需要在启动时执行一次。如果你使用框架，则可能会自动在幕后执行此操作。

为了进行 hydrate，React 将把你的组件逻辑连接到服务器上生成的初始 HTML 中。hydrate 可以将来自服务器的初始 HTML 快照转换为在浏览器中运行的完全可交互应用。

#### index.html
```hrml
<!--
  在 <div id="root">...</div> 中的 HTML 内容
  由 react-dom/server 生成
-->
<div id="root"><h1>Hello, world!</h1><button>You clicked me <!-- -->0<!-- --> times</button></div>
```

#### index.js
```jsx
import './styles.css';
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(
  document.getElementById('root'),
  <App />
);
```

#### App.js
```jsx
import { useState } from 'react';

export default function App() {
  return (
    <>
      <h1>Hello, world!</h1>
      <Counter />
    </>
  );
}

function Counter() {
  const [count, setCount] = useState(0);
  return (
    <button onClick={() => setCount(count + 1)}>
      You clicked me {count} times
    </button>
  );
}
```

你不需要再次调用 `hydrateRoot` 或者在其他地方调用它。从现在开始，React 将管理你的应用程序的 DOM。想要更新 UI 请使用 useState 替代。

### 陷阱
传递给 `hydrateRoot` 的 React 树必须生成与服务端 *相同的输出*。

这对于用户体验非常重要。用户会在你的 JavaScript 代码加载前花费一些时间来查看服务端生成的 HTML。服务端渲染通过显示应用输出的 HTML 快照来产生了应用程序加速加载的错觉。突然出现不同的内容会破坏这种错觉。这就是为什么服务端渲染输出必须与客户端初始渲染输出匹配。

导致 hydrate 错误的最常见原因包括：

+ 根节点 React 生成的 HTML 周围存在额外的空白符（如换行符）。
+ 在渲染逻辑中使用 `typeof window !== 'undefined'` 这样的判断。
+ 在渲染逻辑中使用仅限于浏览器端的 API，例如 `window.matchMedia`。
+ 在服务器和客户端渲染不同的数据。

React 可以从一些 hydrate 错误中恢复，但 *你必须像处理其他 bug 一样修复它们*。在最好的情况下，它们会导致应用程序加载变慢；在最坏的情况下，事件处理程序可能会附加到错误的元素上。

## hydrate 整个文档 
完全使用 React 构建的应用程序可以将整个文档作为 JSX 渲染，包括 `<html>` 标签：

```jsx
function App() {
  return (
    <html>
      <head>
        <meta charSet="utf-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1" />
        <link rel="stylesheet" href="/styles.css"></link>
        <title>My app</title>
      </head>
      <body>
        <Router />
      </body>
    </html>
  );
}
```

要对整个文档进行 hydrate 处理，将全局的 `document` 作为 `hydrateRoot` 的第一个参数传递：

```jsx
import { hydrateRoot } from 'react-dom/client';
import App from './App.js';

hydrateRoot(document, <App />);
```

