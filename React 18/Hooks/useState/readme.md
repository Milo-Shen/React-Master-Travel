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

+ `set` 函数 仅更新 下一次 渲染的状态变量。如果在调用 `set` 函数后读取状态变量，则 仍会得到在调用之前显示在屏幕上的旧值。

+ 如果你提供的新值与当前 `state` 相同（由 `Object.is` 比较确定），React 将 跳过重新渲染该组件及其子组件。这是一种优化。虽然在某些情况下 React 仍然需要在跳过子组件之前调用你的组件，但这不应影响你的代码。

+ React 会 批量处理状态更新。它会在所有 事件处理函数运行 并调用其 set 函数后更新屏幕。这可以防止在单个事件期间多次重新渲染。在某些罕见情况下，你需要强制 React 更早地更新屏幕，例如访问 DOM，你可以使用 `flushSync`。

+ 在渲染期间，只允许在当前渲染组件内部调用 `set` 函数。React 将丢弃其输出并立即尝试使用新状态重新渲染。这种方式很少需要，但你可以使用它来存储 先前渲染中的信息。请参见下面的示例。

+ 在严格模式中，React 将 两次调用你的更新函数，以帮助你找到 意外的不纯性。这只是开发时的行为，不影响生产。如果你的更新函数是纯函数（本该是这样），就不应影响该行为。其中一次调用的结果将被忽略。

## 用法

### 为组件添加状态 

在组件的顶层调用 `useState` 来声明一个或多个 状态变量。

```jsx
import { useState } from 'react';

function MyComponent() {
    const [age, setAge] = useState(42);
    const [name, setName] = useState('Taylor');
    // ...
}
```

按照惯例使用 数组解构 来命名状态变量，例如 `[something, setSomething]`。

`useState` 返回一个只包含两个项的数组：

1. 该状态变量 当前的 `state`，最初设置为你提供的 初始化 `state`。
2. `set` 函数，它允许你在响应交互时将 `state` 更改为任何其他值。

要更新屏幕上的内容，请使用新状态调用 `set` 函数：

```jsx
function handleClick() {
  setName('Robin');
}
```

React 会存储新状态，使用新值重新渲染组件，并更新 UI。

#### 陷阱
调用 `set` 函数 不会 改变已经执行的代码中当前的 `state`：

```jsx
function handleClick() {
  setName('Robin');
  console.log(name); // Still "Taylor"!
}
```

它只影响 下一次 渲染中 `useState` 返回的内容。

#### 基本的 useState 示例

```jsx
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      You pressed me {count} times
    </button>
  );
}
```

### 根据先前的 state 更新 state 

假设 `age` 为 `42`，这个处理函数三次调用 `setAge(age + 1)`：

```jsx
function handleClick() {
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
}
```

为了解决这个问题，你可以向 `setAge` 传递一个 更新函数，而不是下一个状态：

```jsx
function handleClick() {
  setAge(a => a + 1); // setAge(42 => 43)
  setAge(a => a + 1); // setAge(43 => 44)
  setAge(a => a + 1); // setAge(44 => 45)
}
```

这里，`a => a + 1` 是更新函数。它获取 待定状态 并从中计算 下一个状态。

React 将更新函数放入 队列 中。然后，在下一次渲染期间，它将按照相同的顺序调用它们：

1. `a => a + 1` 将接收 `42` 作为待定状态，并返回 `43` 作为下一个状态。
2. `a => a + 1` 将接收 `43` 作为待定状态，并返回 `44` 作为下一个状态。
3. `a => a + 1` 将接收 `44` 作为待定状态，并返回 `45` 作为下一个状态。

现在没有其他排队的更新，因此 React 最终将存储 `45` 作为当前状态。

按照惯例，通常将待定状态参数命名为状态变量名称的第一个字母，如 `age` 为 `a`。然而，你也可以把它命名为 `prevAge` 或者其他你觉得更清楚的名称。

React 在开发环境中可能会 两次调用你的更新函数 来验证其是否为 纯函数。

#### 是否总是优先使用更新函数？ 

你可能会听到这样的建议，如果要设置的状态是根据先前的状态计算得出的，则应始终编写类似于 `setAge(a => a + 1)` 的代码。这样做没有害处，但也不总是必要的。

在大多数情况下，这两种方法没有区别。React 始终确保对于有意的用户操作，如单击，age 状态变量将在下一次单击之前被更新。这意味着单击事件处理函数在事件处理开始没有得到“过时” `age` 的风险。

但是，如果在同一事件中进行多个更新，则更新函数可能会有帮助。如果访问状态变量本身不方便（在优化重新渲染时可能会遇到这种情况），它们也很有用。

如果比起轻微的冗余你更喜欢语法的一致性，你正设置的状态又是根据先前的状态计算出来的，那么总是编写一个更新函数是合理的。如果它是从某个其他状态变量的先前状态计算出的，则你可能希望将它们结合成一个对象然后 使用 `reducer`。

### 传递更新函数和直接传递下一个状态之间的区别

#### 1. 传递更新函数
这个例子传递了更新函数，因此“+3”按钮可以正常工作。

```jsx
import { useState } from 'react';

export default function Counter() {
  const [age, setAge] = useState(42);

  function increment() {
    setAge(a => a + 1);
  }

  return (
    <>
      <h1>Your age: {age}</h1>
      <button onClick={() => {
        increment();
        increment();
        increment();
      }}>+3</button>
      <button onClick={() => {
        increment();
      }}>+1</button>
    </>
  );
}
```

#### 2. 直接传递下一个状态
这个示例 没有 传递更新函数，所以“+3”按钮 不能按预期的方式工作。

```jsx
import { useState } from 'react';

export default function Counter() {
  const [age, setAge] = useState(42);

  function increment() {
    setAge(age + 1);
  }

  return (
    <>
      <h1>Your age: {age}</h1>
      <button onClick={() => {
        increment();
        increment();
        increment();
      }}>+3</button>
      <button onClick={() => {
        increment();
      }}>+1</button>
    </>
  );
}
```