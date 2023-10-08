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

`#### 返回值
`hydrateRoot 返回一个包含两个方法的对象 `render` 和 `unmount`。
