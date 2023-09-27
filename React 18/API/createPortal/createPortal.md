# createPortal
`createPortal` 允许你将 JSX 作为 children 渲染至 DOM 的不同部分。

```jsx
<div>
  <SomeComponent />
  {createPortal(children, domNode, key?)}
</div>
```

+ 参考
  + `createPortal(children, domNode, key?)`
+ 用法
  + 渲染到 DOM 的不同部分
  + 使用 portal 渲染模态对话框
  + 将 React 组件渲染到非 React 服务器标记中
  + 将 React 组件渲染到非 React DOM 节点

## 参考 

```jsx
createPortal(children, domNode, key?) 
```

调用 `createPortal` 创建 `portal`，并传入 JSX 与实际渲染的目标 DOM 节点：

```jsx
import { createPortal } from 'react-dom';

// ...

<div>
  <p>这个子节点被放置在父节点 div 中。</p>
  {createPortal(
    <p>这个子节点被放置在 document body 中。</p>,
    document.body
  )}
</div>
```

portal 只改变 DOM 节点的所处位置。在其他方面，渲染至 portal 的 JSX 的行为表现与作为 React 组件的子节点一致。该子节点可以访问由父节点树提供的 context 对象、事件将从子节点依循 React 树冒泡到父节点。