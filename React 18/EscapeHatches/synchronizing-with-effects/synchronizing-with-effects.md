# 使用 Effect 同步
有些组件需要与外部系统同步。例如，你可能希望根据 React state 控制非 React 组件、设置服务器连接或在组件出现在屏幕上时发送分析日志。Effects 会在渲染后运行一些代码，以便可以将组件与 React 之外的某些系统同步。

## 你将会学习到
+ 什么是 Effect
+ Effect 与事件（event）有何不同
+ 如何在组件中声明 Effect
+ 如何避免不必要地重新运行 Effect
+ 为什么 Effect 在开发环境中会影响两次，如何修复它们

## 什么是 Effect，它与事件（event）有何不同？ 
在谈到 Effect 之前，你需要熟悉 React 组件中的两种逻辑类型：
+ 渲染逻辑代码（在 描述 UI 中有介绍）位于组件的顶层。你将在这里接收 props 和 state，并对它们进行转换，最终返回你想在屏幕上看到的 JSX。渲染的代码必须是纯粹的——就像数学公式一样，它只应该“计算”结果，而不做其他任何事情。
+ 事件处理程序（在 添加交互性 中介绍）是嵌套在组件内部的函数，而不仅仅是计算函数。事件处理程序可能会更新输入字段、提交 HTTP POST 请求以购买产品，或者将用户导航到另一个屏幕。事件处理程序包含由特定用户操作（例如按钮点击或键入）引起的“副作用”（它们改变了程序的状态）。

有时这还不够。考虑一个 `ChatRoom` 组件，它在屏幕上可见时必须连接到聊天服务器。连接到服务器不是一个纯计算（它包含副作用），因此它不能在渲染过程中发生。然而，并没有一个特定的事件（比如点击）导致 ChatRoom 被显示。

*Effect 允许你指定由渲染本身，而不是特定事件引起的副作用*。在聊天中发送消息是一个“事件”，因为它直接由用户点击特定按钮引起。然而，建立服务器连接是 Effect，因为它应该发生无论哪种交互导致组件出现。Effect 在屏幕更新后的 提交阶段 运行。这是一个很好的时机，可以将 React 组件与某个外部系统（如网络或第三方库）同步。

## 注意
在本文和后续文本中，`Effect` 在 React 中是专有定义——由渲染引起的副作用。为了指代更广泛的编程概念，也可以将其称为“副作用（side effect）”。

## 你可能不需要 Effect 
*不要随意在你的组件中使用 Effect*。记住，Effect 通常用于暂时“跳出” React 代码并与一些 外部 系统进行同步。这包括浏览器 API、第三方小部件，以及网络等等。如果你想用 Effect 仅根据其他状态调整某些状态，那么 你可能不需要 Effect。

## 如何编写 Effect 
编写 Effect 需要遵循以下三个规则：

1. *声明 Effect*。默认情况下，Effect 会在每次渲染后都会执行。
2. *指定 Effect 依赖*。大多数 Effect 应该按需执行，而不是在每次渲染后都执行。例如，淡入动画应该只在组件出现时触发。连接和断开服务器的操作只应在组件出现和消失时，或者切换聊天室时执行。文章将介绍如何通过指定依赖来控制如何按需执行。
3. *必要时添加清理（`cleanup`）函数*。有时 Effect 需要指定如何停止、撤销，或者清除它的效果。例如，“连接”操作需要“断连”，“订阅”需要“退订”，“获取”既需要“取消”也需要“忽略”。你将学习如何使用 清理函数 来做到这一切。

以下是具体步骤。

### 第一步：声明 Effect 
首先在 React 中引入 `useEffect` Hook：

```jsx
import { useEffect } from 'react';
```

然后，在组件顶部调用它，并传入在每次渲染时都需要执行的代码：

```jsx
function MyComponent() {
  useEffect(() => {
    // 每次渲染后都会执行此处的代码
  });
  return <div />;
}
```

每当你的组件渲染时，React 将更新屏幕，然后运行 `useEffect` 中的代码。换句话说，`useEffect` 会把这段代码放到 *屏幕更新渲染之后* 执行。

让我们看看如何使用 Effect 与外部系统同步。考虑一个 `<VideoPlayer>` React 组件。通过传递布尔类型的 `isPlaying` prop 以控制是播放还是暂停：

```jsx
<VideoPlayer isPlaying={isPlaying} />;
```

自定义的 `VideoPlayer` 组件渲染了内置的 `<video>` 标签：

```jsx
function VideoPlayer({ src, isPlaying }) {
  // TODO：使用 isPlaying 做一些事情
  return <video src={src} />;
}
```

但是，浏览器的 `<video>` 标签没有 `isPlaying` 属性。控制它的唯一方式是在 DOM 元素上调用 `play()` 和 `pause()` 方法。因此，你需要将 `isPlaying` prop 的值与 `play()` 和 `pause()` 等函数的调用进行同步，该属性用于告知当前视频是否应该播放。

首先要获取 `<video>` DOM 节点的 对象引用。

你可能会尝试在渲染期间调用 `play()` 或 `pause()`，但这种做法是错的：

```jsx
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  if (isPlaying) {
    ref.current.play();  // 渲染期间不能调用 `play()`。 
  } else {
    ref.current.pause(); // 同样，调用 `pause()` 也不行。
  }

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  return (
    <>
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? '暂停' : '播放'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

这段代码之所以不正确，是因为它试图在渲染期间对 DOM 节点进行操作。在 React 中，JSX 的渲染必须是纯粹操作，不应该包含任何像修改 DOM 的副作用。

而且，当第一次调用 `VideoPlayer` 时，对应的 DOM 节点甚至还不存在！如果连 DOM 节点都没有，那么如何调用 `play()` 或 `pause()` 方法呢！在返回 JSX 之前，React 不知道要创建什么 DOM。

解决办法是 使用 `useEffect` 包裹副作用，*把它分离到渲染逻辑的计算过程之外*：

```jsx
import { useEffect, useRef } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}
```

把调用 DOM 方法的操作封装在 Effect 中，你可以让 React 先更新屏幕，确定相关 DOM 创建好了以后然后再运行 Effect。

当 `VideoPlayer` 组件渲染时（无论是否为首次渲染），都会发生以下事情。首先，React 会刷新屏幕，确保 `<video>` 元素已经正确地出现在 DOM 中；然后，React 将运行 Effect；最后，Effect 将根据 `isPlaying` 的值调用 `play()` 或 `pause()`。

试试按下几次播放和暂停操作，观察视频播放器的播放、暂停行为是如何与 `isPlaying` prop 同步的：

```jsx
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  return (
    <>
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? '暂停' : '播放'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

在这个示例中，你同步到 React `state` 的“外部系统”是浏览器媒体 API；也可以使用类似的方法将旧的非 React 代码（如 jQuery 插件）封装成声明性的 React 组件。

请注意，控制视频播放器在实际应用中复杂得多：比如调用 `play()` 可能会失败，用户可能会使用内置浏览器控件播放或暂停等等。这只是一个简化了很多具体细节的例子。

### 陷阱
一般来说，Effect 会在 *每次* 渲染后执行，*而以下代码会陷入死循环中*：

```jsx
const [count, setCount] = useState(0);
useEffect(() => {
  setCount(count + 1);
});
```

每次渲染结束都会执行 Effect；而更新 `state` 会触发重新渲染。但是新一轮渲染时又会再次执行 Effect，然后 Effect 再次更新 `state`…… 如此周而复始，从而陷入死循环。
Effect 通常应该使组件与 外部 系统保持同步。如果没有外部系统，你只想根据其他状态调整一些状态，那么 你也许不需要 Effect。

### 第二步：指定 Effect 依赖 
一般来说，Effect 会在 *每次* 渲染时执行。*但更多时候，并不需要每次渲染的时候都执行 Effect*。
+ 有时这会拖慢运行速度。因为与外部系统的同步操作总是有一定时耗，在非必要时可能希望跳过它。例如，没有人会希望每次用键盘打字时都重新连接聊天服务器。
+ 有时这会导致程序逻辑错误。例如，组件的淡入动画只需要在第一轮渲染出现时播放一次，而不是每次触发新一轮渲染后都播放。

为了演示这个问题，我们在前面的示例中加入一些 `console.log` 语句和更新父组件 `state` 的文本输入。请注意键入是如何导致 Effect 重新运行的：

```jsx
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log('调用 video.play()');
      ref.current.play();
    } else {
      console.log('调用 video.pause()');
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? '暂停' : '播放'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

将 *依赖数组* 传入 `useEffect` 的第二个参数，以告诉 React 跳过不必要地重新运行 Effect。在上面示例的第 14 行中传入一个空数组 `[]`：

```jsx
  useEffect(() => {
    // ...
  }, []);
```