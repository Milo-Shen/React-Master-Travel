# useState

`useState` 是一个 React Hook，它允许你向组件添加一个 状态变量。

```jsx
const [state, setState] = useState(initialState);
```

## 参考 

```jsx
useState(initialState);
```

在组件的顶层调用 `useState` 来声明一个 状态变量。

```jsx
import { useState } from 'react';

function MyComponent() {
    const [age, setAge] = useState(28);
    const [name, setName] = useState('Taylor');
    
    // react 会调用，后方的回调函数
    const [todos, setTodos] = useState(() => createTodos());
    // ...
}
```

按照惯例使用 数组解构 来命名状态变量，例如 `[something, setSomething]`。

### 参数 
+ `initialState`：你希望 s`tate 初始化的值。它可以是任何类型的值，但对于函数有特殊的行为。在初始渲染后，此参数将被忽略。
+ 如果传递函数作为 `initialState`，则它将被视为 初始化函数。它应该是纯函数，不应该接受任何参数，并且应该返回一个任何类型的值。当初始化组件时，React 将调用你的初始化函数，并将其返回值存储为初始状态。请参见下面的示例。
