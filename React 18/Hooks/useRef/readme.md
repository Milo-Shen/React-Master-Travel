# useRef

`useRef` 是一个 React Hook，它能让你引用一个不需要渲染的值。

```
const ref = useRef(initialValue)
```

## 参考 

```jsx
useRef(initialValue);
```

在你组件的顶层调用 `useRef` 声明一个 `ref`。

```jsx
import { useRef } from 'react';

function MyComponent() {
    const intervalRef = useRef(0);
    const inputRef = useRef(null);
    // ...
}
```

### 参数 
+ `initialValue`：`ref` 对象的 `current` 属性的初始值。可以是任意类型的值。这个参数会首次渲染后被忽略。

### 返回值 

`useRef` 返回一个只有一个属性的对象:

+ `current`：最初，它被设置为你传递的 `initialValue`。之后你可以把它设置为其他值。如果你把 `ref` 对象作为一个 JSX 节点的 `ref` 属性传递给 React，React 将为它设置 `current` 属性。

在后续的渲染中，`useRef` 将返回同一个对象。

### 注意事项 

+ 你可以修改 `ref.current` 属性。与 `state` 不同，它是可变的。然而，如果它持有一个用于渲染的对象（例如，你的 `state` 的一部分），那么你就不应该修改这个对象。
+ 当你改变 `ref.current` 属性时，React 不会重新渲染你的组件。React 不知道你何时改变它，因为 `ref` 是一个普通的 JavaScript 对象。
+ 除了 初始化 外不要在渲染期间写入 或者读取 `ref.current`。这会使你的组件的行为不可预测。
+ 在严格模式下，React 将会 调用两次组件方法，这是为了 帮助你发现意外的问题。这只是开发模式下的行为，不影响生产模式。每个 `ref` 对象将会创建两次，但是其中一个版本将被丢弃。如果你的组件函数是纯的（应该如此），这不会影响其行为。

### 使用方法 

#### 用 `ref` 引用一个值 

在你的组件的顶层调用 useRef 声明一个或多个 refs。

```jsx
import { useRef } from 'react';

function Stopwatch() {
    const intervalRef = useRef(0);
    // ...
}
```

`useRef` 返回一个具有单个 `current` 属性 的 `ref` 对象，并初始化为你提供的 `initial value`

在后续的渲染中，`useRef` 将返回相同的对象。你可以改变它的 `current` 属性来存储信息，并在之后读取它。这会让你联想起 `state`，但是有一个重要的区别。

改变 `ref` 不会触发重新渲染。 这意味着 `ref` 是存储一些不影响组件视图输出的信息的完美选择。例如，如果你需要存储一个 `intervalID` 并在以后检索它，你可以把它放在一个 `ref` 中。如果要更新 `ref` 里面的值，你需要手动改变它的

```jsx
function handleStartClick() {
  const intervalId = setInterval(() => {
    // ...
  }, 1000);
  intervalRef.current = intervalId;
}
```

在之后，你可以从 `ref` 中读取 interval ID，这样你就可以 清除定时器：

```jsx
function handleStopClick() {
  const intervalId = intervalRef.current;
  clearInterval(intervalId);
}
```

通过使用 `ref`，你可以确保：
+ 你可以在重新渲染之间 存储信息（不像是普通对象，每次渲染都会重置）。
+ 改变它 不会触发重新渲染（不像是 `state` 变量，会触发重新渲染）。
+ 对于你的组件的每个副本来说，这些信息都是本地的（不像是外面的变量，是共享的）。

改变 `ref` 不会触发重新渲染，所以 `ref` 不适合用于存储期望显示在屏幕上的信息。如有需要，使用 `state` 代替。

### Examples of referencing a value with useRef

#### 第 1 个示例 共 2 个挑战: 点击计数器 
这个组件使用 `ref` 来记录按钮被点击的次数。注意，在这里使用 `ref` 而不是 `state` 是可以的，因为点击次数只在事件处理程序中被读取和写入。

```jsx
import { useRef } from 'react';

export default function Counter() {
  let ref = useRef(0);

  function handleClick() {
    ref.current = ref.current + 1;
    alert('You clicked ' + ref.current + ' times!');
  }

  return (
    <button onClick={handleClick}>
      Click me!
    </button>
  );
}
```

#### 第 2 个示例 共 2 个挑战: 秒表 
这个例子使用了 `state` 和 `ref` 的组合。`startTime` 和 `now` 都是 `state` 变量，因为它们是用来渲染的。但是我们还需要持有一个 interval ID，这样我们就可以在按下按钮时停止定时器。因为 interval ID 不用于渲染，所以应该把它保存在一个 `ref` 中，并且手动更新它。

```jsx
import { useState, useRef } from 'react';

export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);
  const intervalRef = useRef(null);

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current);
    intervalRef.current = setInterval(() => {
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }

  let secondsPassed = 0;
  if (startTime != null && now != null) {
    secondsPassed = (now - startTime) / 1000;
  }

  return (
    <>
      <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
      <button onClick={handleStart}>
        Start
      </button>
      <button onClick={handleStop}>
        Stop
      </button>
    </>
  );
}
```