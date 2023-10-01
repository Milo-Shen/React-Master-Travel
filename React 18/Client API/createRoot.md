# createRoot

`createRoot` 允许在浏览器的 DOM 节点中创建根节点以显示 React 组件。

```jsx
const root = createRoot(domNode, options?)
```

## 参考 
```jsx
createRoot(domNode, options?) 
```

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

## 参数 