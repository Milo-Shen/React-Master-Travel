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

### 返回
`useState` 返回一个由两个值组成的数组：
1. 当前的 state。在首次渲染时，它将与你传递的 `initialState` 相匹配。
2. `set` 函数，它可以让你将 `state` 更新为不同的值并触发重新渲染。

### 注意事项 
`useState` 是一个 Hook，因此你只能在 组件的顶层 或自己的 Hook 中调用它。你不能在循环或条件语句中调用它。如果你需要这样做，请提取一个新组件并将状态移入其中。
在严格模式中，React 将 两次调用初始化函数，以 帮你找到意外的不纯性。这只是开发时的行为，不影响生产。如果你的初始化函数是纯函数（本该是这样），就不应影响该行为。其中一个调用的结果将被忽略。

`set` 函数，例如 `setSomething(nextState) `
`useState` 返回的 `set` 函数允许你将 `state` 更新为不同的值并触发重新渲染。你可以直接传递新状态，也可以传递一个根据先前状态来计算新状态的函数：

```jsx
const [name, setName] = useState('Edward');

function handleClick() {
    setName('Taylor');
    setAge(a => a + 1);
    // ...
}
```

### 参数 
+ `nextState`：你想要 `state` 更新为的值。它可以是任何类型的值，但对于函数有特殊的行为。
+ 如果你将函数作为 `nextState` 传递，它将被视为 更新函数。它必须是纯函数，只接受待定的 `state` 作为其唯一参数，并应返回下一个状态。React 将把你的更新函数放入队列中并重新渲染组件。在下一次渲染期间，React 将通过把队列中所有更新函数应用于先前的状态来计算下一个状态。

### 返回值 
`set` 函数没有返回值。

### 注意事项 