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