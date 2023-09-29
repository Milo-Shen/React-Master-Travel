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

### 参数 
+ `children`：React 可以渲染的任何内容，如 JSX 片段（`<div />` 或 `<SomeComponent />` 等等）、Fragment（`<>...</>`）、字符串或数字，以及这些内容构成的数组。
+ `domNode`：某个已经存在的 DOM 节点，例如由 `document.getElementById()` 返回的节点。在更新过程中传递不同的 DOM 节点将导致 portal 内容被重建。
+ 可选参数 `key`：用作 portal key 的独特字符串或数字。

### 返回值 
`createPortal` 返回一个可以包含在 JSX 中或从 React 组件中返回的 React 节点。如果 React 在渲染输出中遇见它，它将把提供的 `children` 放入提供的 `domNode` 中。

### 警告 
portal 中的事件传播遵循 React 树而不是 DOM 树。例如点击 `<div onClick>` 内部的 portal，将触发 `onClick` 处理程序。如果这会导致意外的问题，*请在 portal 内部停止事件传播，或将 portal 移动到 React 树中的上层。*

