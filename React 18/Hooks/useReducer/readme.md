# useReducer

`useReducer` 是一个 React Hook，它允许你向组件里面添加一个 `reducer`。

## 参考 

```jsx
useReducer(reducer, initialArg, init?);
```

在组件的顶层作用域调用 `useReducer` 以创建一个用于管理状态的 `reducer`。

```jsx
import { useReducer } from 'react';

function reducer(state, action) {
  // ...
}

function MyComponent() {
    const [state, dispatch] = useReducer(reducer, {age: 42});
    // ...
}
```