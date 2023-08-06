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

### 陷阱

#### 不要在渲染期间写入 或者读取 `ref.current`。

React 期望你的组件的主体 表现得像一个纯函数：
+ 如果输入的（`props`、`state` 和 `context`）都是一样的，那么就应该返回一样的 JSX。
+ 以不同的顺序或用不同的参数调用它，不应该影响其他调用的结果。

在 *渲染期间* 读取或写入 `ref` 会破坏这些预期行为。

```jsx
function MyComponent() {
  // ...
  // 🚩 不要在渲染期间写入 ref
  myRef.current = 123;
  // ...
  // 🚩 不要在渲染期间读取 ref
  return <h1>{myOtherRef.current}</h1>;
}
```

你可以在 事件处理程序或者 effects 中读取和写入 ref。

```jsx
function MyComponent() {
  // ...
  useEffect(() => {
    // ✅ 你可以在 effects 中读取和写入 ref
    myRef.current = 123;
  });
  // ...
  function handleClick() {
    // ✅ 你可以在事件处理程序中读取和写入 ref
    doSomething(myOtherRef.current);
  }
  // ...
}
```

如果 *不得不* 在渲染期间读取 或者写入，使用 state 代替。

当你打破这些规则时，你的组件可能仍然可以工作，但是我们为 React 添加的大多数新功能将依赖于这些预期行为。

### 通过 ref 操作 DOM 
使用 `ref` 操作 DOM 是非常常见的。React 内置了对它的支持。

首先，声明一个 `initial value` 为 `null` 的 `ref` 对象

```jsx
import { useRef } from 'react';

function MyComponent() {
    const inputRef = useRef(null);
    // ...
}
```

然后将你的 `ref` 对象作为 `ref` 属性传递给你想要操作的 DOM 节点的 JSX：

```jsx
  // ...
  return <input ref={inputRef} />;
```

当 React 创建 DOM 节点并将其渲染到屏幕时，React 将会把 DOM 节点设置为你的 `ref` 对象的 `current` 属性。现在你可以访问 `<input>` 的 DOM 节点，并且可以调用类似于 `focus()` 的方法：

```jsx
  function handleClick() {
    inputRef.current.focus();
  }
```

当节点从屏幕上移除时，React 将把 `current` 属性设回 `null`。

### Examples of manipulating the DOM with useRef

#### 第 1 个示例 共 4 个挑战: 聚焦文字输入框 

在这个示例中，点击按钮将会聚焦 `input`：

```jsx
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

#### 第 2 个示例 共 4 个挑战: 滚动图片到视图 

在这个示例中，点击按钮将会把图片滚动到视图。这里使用 ref 绑定到列表的 DOM 节点，然后调用 DOM 的 `querySelectorAll` API 找到我们想要滚动的图片。

```jsx
import { useRef } from 'react';

export default function CatFriends() {
  const listRef = useRef(null);

  function scrollToIndex(index) {
    const listNode = listRef.current;
    // This line assumes a particular DOM structure:
    const imgNode = listNode.querySelectorAll('li > img')[index];
    imgNode.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  return (
    <>
      <nav>
        <button onClick={() => scrollToIndex(0)}>
          Tom
        </button>
        <button onClick={() => scrollToIndex(1)}>
          Maru
        </button>
        <button onClick={() => scrollToIndex(2)}>
          Jellylorum
        </button>
      </nav>
      <div>
        <ul ref={listRef}>
          <li>
            <img
              src="https://placekitten.com/g/200/200"
              alt="Tom"
            />
          </li>
          <li>
            <img
              src="https://placekitten.com/g/300/200"
              alt="Maru"
            />
          </li>
          <li>
            <img
              src="https://placekitten.com/g/250/200"
              alt="Jellylorum"
            />
          </li>
        </ul>
      </div>
    </>
  );
}
```

#### 第 3 个示例 共 4 个挑战: 播放和暂停视频

这个示例使用 `ref` 调用 `<video>` DOM 节点的 `play()` 和 `pause()` 方法。

```jsx
import { useState, useRef } from 'react';

export default function VideoPlayer() {
  const [isPlaying, setIsPlaying] = useState(false);
  const ref = useRef(null);

  function handleClick() {
    const nextIsPlaying = !isPlaying;
    setIsPlaying(nextIsPlaying);

    if (nextIsPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  }

  return (
    <>
      <button onClick={handleClick}>
        {isPlaying ? 'Pause' : 'Play'}
      </button>
      <video
        width="250"
        ref={ref}
        onPlay={() => setIsPlaying(true)}
        onPause={() => setIsPlaying(false)}
      >
        <source
          src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
          type="video/mp4"
        />
      </video>
    </>
  );
}
```

### 第 4 个示例 共 4 个挑战: 向你的组件暴露 ref 

有时，你可能想让父级组件在你的组件中操纵 DOM。例如，也许你正在写一个 `MyInput` 组件，但你希望父组件能够聚焦 `input`（父组件无法访问）。你可以使用一个组合，通过 `useRef` 持有 `input` 并且通过 `forwardRef` 来将其暴露给父组件。

```jsx
import { forwardRef, useRef } from 'react';

const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

### 避免重复创建 ref 的内容

React 保存首次的 `ref` 初始值，并在后续的渲染中忽略它。

```jsx
function Video() {
    const playerRef = useRef(new VideoPlayer());
    // ...
}
```

虽然 `new VideoPlayer()` 的结果只会在首次渲染时使用，但是你依然在每次渲染时都在调用这个方法。如果是创建昂贵的对象，这可能是一种浪费。

为了解决这个问题，你可以像这样初始化 `ref`：

```jsx
function Video() {
    const playerRef = useRef(null);
    if (playerRef.current === null) {
        playerRef.current = new VideoPlayer();
    }
    // ...
}
```

通常情况下，在渲染过程中写入或读取 `ref.current` 是不允许的。然而，在这种情况下是可以的，因为结果总是一样的，而且条件只在初始化时执行，所以是完全可以预测的。

### 如何避免在初始化 useRef 之后进行 null 的类型检查 

如果你使用一个类型检查器，并且不想总是检查 `null`，你可以尝试用这样的模式来代替：

```jsx
function Video() {
    const playerRef = useRef(null);

    function getPlayer() {
        if (playerRef.current !== null) {
            return playerRef.current;
        }
        const player = new VideoPlayer();
        playerRef.current = player;
        return player;
    }

    // ...
}
```

在这里，`playerRef` 本身是可以为空的。然而，你应该能够使你的类型检查器确信，不存在 `getPlayer()` 返回 `null` 的情况。然后在你的事件处理程序中使用 `getPlayer()`。

## 疑难解答 

### 我无法获取自定义组件的 ref 

如果你尝试像这样传递 `ref` 到你自己的组件：

```jsx
const inputRef = useRef(null);

return <MyInput ref={inputRef} />;
```

默认情况下，你自己的组件不会暴露它们内部 DOM 节点的 `ref`。

为了解决这个问题，首先，找到你想获得 `ref` 的组件：

然后像这样将其包装在 `forwardRef` 里：

```jsx
import { forwardRef } from 'react';

const MyInput = forwardRef(({ value, onChange }, ref) => {
  return (
    <input
      value={value}
      onChange={onChange}
      ref={ref}
    />
  );
});

export default MyInput;
```

最后，父级组件就可以得到它的 `ref`。