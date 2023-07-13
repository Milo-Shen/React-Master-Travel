# 使用 ref 引用值

当你希望组件“记住”某些信息，但又不想让这些信息 触发新的渲染 时，你可以使用 `ref` 。

## 你将会学习到
1. 如何向组件添加 `ref`
2. 如何更新 `ref` 的值
3. `ref` 与 `state` 有何不同
4. 如何安全地使用 `ref`

### 给你的组件添加 `ref` 

你可以通过从 React 导入 `useRef` Hook 来为你的组件添加一个 `ref`：

```jsx
import { useRef } from 'react';
```

在你的组件内，调用 `useRef` Hook 并传入你想要引用的初始值作为唯一参数。例如，这里的 `ref` 引用的值是 “0”：

```jsx
const ref = useRef(0);
```

`useRef` 返回一个这样的对象:

```jsx
{ 
  current: 0 // 你向 useRef 传入的值
}
```