# createRoot

`createRoot` 允许在浏览器的 DOM 节点中创建根节点以显示 React 组件。

```jsx
const root = createRoot(domNode, options?)
```

## 参考 

## `createRoot(domNode, options?) `

调用 `createRoot` 以在浏览器 DOM 元素中创建根节点显示内容。

```jsx
import { createRoot } from 'react-dom/client';

const domNode = document.getElementById('root');
const root = createRoot(domNode);
```

React 将会为 `domNode` 创建一个根节点，并控制其中的 DOM。在已经创建根节点之后，需要调用 `root.render` 来显示 React 组件：

```jsx
root.render(<App />);
```

对于一个完全用 React 构建的应用程序，通常会调用一个 `createRoot` 来创建它的根节点。而对于一个使用了“少量” React 来创建部分内容的应用程序，则要按具体需求来确定根节点的数量。

### 参数 
+ `domNode`：一个 DOM 元素。React 将为这个 DOM 元素创建一个根节点然后允许你在这个根节点上调用函数，比如 `render` 来显示渲染的 React 内容。
+ 可选 `options`：用于配置这个 React 根节点的对象。
  + 可选 `onRecoverableError`：回调函数，在 React 从异常错误中恢复时自动调用。
  + 可选 `identifierPrefix`：一个 React 用来配合 useId 生成 id 的字符串前缀。在同一个页面上使用多个根节点的场景下，这将能有效避免冲突。

### 返回值 
`createRoot` 返回一个带有两个方法的的对象，这两个方法是：`render` 和 `unmount`。

### 注意 
+ 如果应用程序是服务端渲染的，那么不能使用 `createRoot()`。请使用 `hydrateRoot()` 替代它。
+ 在你的应用程序中，可能只调用了一次 `createRoot`。但如果你使用了框架，它可能已经自动帮你完成了这次调用。
+ 当你想要渲染一段 JSX，但是它存在于 DOM 树的其他位置，并非当前组件的子组件时（比如，一个弹窗或者提示框），使用 `createPortal` 替代 `createRoot`。

## `root.render(reactNode)`

调用 `root.render` 以将一段 JSX（“React 节点”）在 React 的根节点中渲染为 DOM 节点并显示。

```jsx
root.render(<App />);
```

React 将会在 根节点 中显示 `<App />` 组件，并且控制组件中的 DOM。

### 参数 
+ reactNode：一个你想要显示的 React 节点。它总是一段 JSX，就像 `<App />`，但是你也总是可以传递一个 `createElement()` 构造的 React 元素、一个字符串、一个数字、`null` 或者 `undefined`。

### 返回值 
`root.render` 返回 `undefined`。

### 注意 
+ 首次调用 `root.render` 时，React 会先清空根节点中所有已经存在的 HTML，然后才会渲染 React 组件。
+ 如果你的根节点包含了由 React 在构建期间通过服务端渲染生成的 HTML 内容，请使用 hydrateRoot() 替代这个方法，这样才能把事件处理程序和现有的 HTML 绑定。

如果你在一个根节点上多次调用了 render，React 仍然会更新 DOM，这样才能保证显示的内容是最新的。React 将会筛选出可复用的部分和需要更新的部分，对于需要更新的部分，是 React 通过与之前渲染的树进行 “比较” 得到的。在同一个根节点上再次调用 render 就和在根节点上调用 set 函数 类似：React 会避免没必要的 DOM 更新。