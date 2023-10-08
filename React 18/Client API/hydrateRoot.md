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


