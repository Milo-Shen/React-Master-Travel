# forwardRef

`forwardRef` 允许你的组件使用 `ref` 将一个 DOM 节点暴露给父组件。

## 参考 

```jsx
forwardRef(render) 
```

使用 `forwardRef()` 来让你的组件接收一个 `ref` 并将其传递给一个子组件：

```jsx
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  // ...
});
```

### 参数 
+ `render`：组件的渲染函数。React 会调用该函数并传入父组件传递来的参数和 `ref`。返回的 `JSX` 将成为组件的输出。

### 返回值 
`forwardRef` 返回一个可以在 JSX 中渲染的 React 组件。与作为纯函数定义的 React 组件不同，`forwardRef` 返回的组件还能够接收 `ref` 属性。

### 警告 
在严格模式中，为了 帮助你找到意外的副作用，React 将会 调用两次你的渲染函数。不过这仅限于开发环境，并不会影响生产环境。如果你的渲染函数是纯函数（也应该是），这应该不会影响你的组件逻辑。其中一个调用的结果将被忽略。