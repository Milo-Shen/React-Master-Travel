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

你会发现 React 报错：`React Hook useEffect has a missing dependency: 'isPlaying'`：

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
  }, []); // 这将产生错误

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? 'Pause' : 'Play'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

问题出现在 Effect 中使用了 `isPlaying` prop 以控制逻辑，但又没有直接告诉 Effect 需要依赖这个属性。为了解决这个问题，将 `isPlaying` 添加至依赖数组中：

```jsx
  useEffect(() => {
    if (isPlaying) { // isPlaying 在此处使用……
      // ...
    } else {
      // ...
    }
  }, [isPlaying]); // ……所以它必须在此处声明！
```

现在所有的依赖都已经声明，所以没有错误了。指定 `[isPlaying]` 会告诉 React，如果 `isPlaying` 在上一次渲染时与当前相同，它应该跳过重新运行 Effect。通过这个改变，输入框的输入不会导致 Effect 重新运行，但是按下播放/暂停按钮会重新运行 Effect。

```jsx
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log('Calling video.play()');
      ref.current.play();
    } else {
      console.log('Calling video.pause()');
      ref.current.pause();
    }
  }, [isPlaying]);

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState('');
  return (
    <>
      <input value={text} onChange={e => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? 'Pause' : 'Play'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

依赖数组可以包含多个依赖项。当指定的所有依赖项在上一次渲染期间的值与当前值完全相同时，React 会跳过重新运行该 Effect。React 使用 Object.is 比较依赖项的值。有关详细信息，请参阅 useEffect 参考文档。

*请注意，不能随意选择依赖项*。如果你指定的依赖项不能与 Effect 代码所期望的相匹配时，lint 将会报错，这将帮助你找到代码中的问题。如果你希望重复执行，那么你应当 重新编辑 Effect 代码本身，使其不需要该依赖项。

### 陷阱
没有依赖数组作为第二个参数，与依赖数组位空数组 `[]` 的行为是不一致的：

```jsx
useEffect(() => {
  // 这里的代码会在每次渲染后执行
});
```

```jsx
useEffect(() => {
  // 这里的代码只会在组件挂载后执行
}, []);
```

```jsx
useEffect(() => {
  //这里的代码只会在每次渲染后，并且 a 或 b 的值与上次渲染不一致时执行
}, [a, b]);
```

接下来，我们将进一步介绍什么是 *挂载（mount）*。

### 为什么依赖数组中可以省略 ref? 
下面的 Effect 同时使用了 `ref` 与 `isPlaying` prop，但是只有 `isPlaying` 被声明为了依赖项：

```jsx
function VideoPlayer({ src, isPlaying }) {
    const ref = useRef(null);
    useEffect(() => {
        if (isPlaying) {
            ref.current.play();
        } else {
            ref.current.pause();
        }
    }, [isPlaying]);
}
```

这是因为 `ref` 具有 稳定 的标识：React 保证 每轮渲染中调用 `useRef` 所产生的引用对象时，获取到的对象引用总是相同的，即获取到的对象引用永远不会改变，所以它不会导致重新运行 Effect。因此，依赖数组中是否包含它并不重要。当然也可以包括它，这样也可以：

```jsx
function VideoPlayer({ src, isPlaying }) {
    const ref = useRef(null);
    useEffect(() => {
        if (isPlaying) {
            ref.current.play();
        } else {
            ref.current.pause();
        }
    }, [isPlaying, ref]);
}
```

`useState` 返回的 `set` 函数 也有稳定的标识符，所以也可以把它从依赖数组中忽略掉。如果在忽略某个依赖项时 linter 不会报错，那么这么做就是安全的。
但是，仅在 linter 可以“看到”对象稳定时，忽略稳定依赖项的规则才会起作用。例如，如果 `ref` 是从父组件传递的，则必须在依赖项数组中指定它。这样做是合适的，因为无法确定父组件是否始终是传递相同的 `ref`，或者可能是有条件地传递几个 `ref` 之一。因此，你的 Effect 将取决于传递的是哪个 `ref`。

### 第三部：按需添加清理（cleanup）函数 
考虑一个不同的例子。你正在编写一个 `ChatRoom` 组件，该组件出现时需要连接到聊天服务器。现在为你提供了 `createConnection()` API，该 API 返回一个包含 `connect()` 与 `disconnection()` 方法的对象。考虑当组件展示给用户时，应该如何保持连接？

从编写 `Effect` 逻辑开始：

```jsx
useEffect(() => {
  const connection = createConnection();
  connection.connect();
});
```

每次重新渲染后连接到聊天室会很慢，因此可以添加依赖数组：

```jsx
useEffect(() => {
  const connection = createConnection();
  connection.connect();
}, []);
```

在这个例子中，Effect 中的代码没有使用任何 `props` 或 `state`，此时指定依赖数组为空数组 `[]`。这告诉 React 仅在组件“挂载”时运行此代码，即首次出现在屏幕上这一阶段。

试试运行下面的代码：

#### App.js
```jsx
import { useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
  }, []);
  return <h1>欢迎来到聊天室！</h1>;
}
```

#### chat.js
```jsx
export function createConnection() {
  // 真实的实现会将其连接到服务器，此处代码只是示例
  return {
    connect() {
      console.log('✅ 连接中……');
    },
    disconnect() {
      console.log('❌ 连接断开。');
    }
  };
}
```

这里的 Effect 仅在组件挂载时执行，所以 "✅ 连接中……" 在控制台中只会打印一次。然而控制台实际打印 "✅ 连接中……" 了两次！为什么会这样？

想象 `ChatRoom` 组件是一个大规模的 App 中许多界面中的一部分。用户切换到含有 `ChatRoom` 组件的页面上时，该组件被挂载，并调用 `connection.connect()` 方法连接服务器。然后想象用户此时突然导航到另一个页面，比如切换到“设置”页面。这时，`ChatRoom` 组件就被卸载了。接下来，用户在“设置”页面忙完后，单击“返回”，回到上一个页面，并再次挂载 `ChatRoom`。这将建立第二次连接，但是，第一次时创建的连接从未被销毁！当用户在应用程序中不断切换界面再返回时，与服务器的连接会不断堆积。

如果不进行大量的手动测试，这样的错误很容易被遗漏。为了帮助你快速发现它们，在开发环境中，React 会在初始挂载组件后，立即再挂载一次。

观察到 "✅ 连接中……" 出现了两次，可以帮助找到问题所在：在代码中，组件被卸载时没有关闭连接。

为了解决这个问题，可以在 Effect 中返回一个 *清理（cleanup）* 函数。

```jsx
useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []);
```

每次重新执行 Effect 之前，React 都会调用清理函数；组件被卸载时，也会调用清理函数。让我们看看执行清理函数会做些什么：

#### App.js

```jsx
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => connection.disconnect();
  }, []);
  return <h1>欢迎来到聊天室！</h1>;
}
```

现在在开发模式下，控制台会打印三条记录：
1. "✅ 连接中……"
2. "❌ 连接断开。"
3. "✅ 连接中……"

*在开发环境下，出现这样的结果才是符合预期的。* 重复挂载组件，可以确保在 React 中离开和返回页面时不会导致代码运行出现问题。上面的代码中规定了挂载组件时连接服务器、卸载组件时断连服务器。所以断开、连接再重新连接是符合预期的行为。当为 Effect 正确实现清理函数时，无论 Effect 执行一次，还是执行、清理、再执行，用户都不会感受到明显的差异。所以，在开发环境下，出现额外的连接、断连时，这是 React 正在调试你的代码。这是很正常的现象，不要试图消除它！

*在生产环境下，"✅ 连接中……" 只会被打印一次。* 也就是说仅在开发环境下才会重复挂载组件，以帮助你找到需要清理的 Effect。你可以选择关闭 严格模式 来关闭开发环境下特有的行为，但我们建议保留它。这可以帮助发现许多上面这样的错误。

## 如何处理在开发环境中 Effect 执行两次？ 
在开发环境中，React 有意重复挂载你的组件，以查找像上面示例中的错误。*正确的态度是“如何修复 Effect 以便它在重复挂载后能正常工作”，而不是“如何只运行一次 Effect”。*

通常的解决办法是实现清理函数。清理函数应该停止或撤销 Effect 正在执行的任何操作。简单来说，用户不应该感受到 Effect 只执行一次（如在生产环境中）和执行“挂载 → 清理 → 挂载”过程（如在开发环境中）之间的差异。

下面提供一些常用的 Effect 应用模式。

### 控制非 React 组件 
有时需要添加不是使用 React 编写的 UI 小部件。例如，假设你要向页面添加地图组件，并且它有一个 `setZoomLevel()` 方法，你希望调整缩放级别（zoom level）并与 React 代码中的 `zoomLevel` `state` 变量保持同步。Effect 看起来应该与下面类似：

```jsx
useEffect(() => {
  const map = mapRef.current;
  map.setZoomLevel(zoomLevel);
}, [zoomLevel]);
```

请注意，在这种情况下不需要清理。在开发环境中，React 会调用 Effect 两次，但这两次挂载时依赖项 setZoomLevel 都是相同的，所以会跳过执行第二次挂载时的 Effect。开发环境中它可能会稍微慢一些，但这问题不大，因为它在生产中不会进行不必要的重复挂载。

某些 API 可能不允许连续调用两次。例如，内置的 `<dialog>` 元素的 `showModal` 方法在连续调用两次时会抛出异常，此时实现清理函数并使其关闭对话框：

```jsx
useEffect(() => {
    const dialog = dialogRef.current;
    dialog.showModal();
    return () => dialog.close();
}, []);
```

在开发环境中，Effect 将调用 `showModal()`，然后立即调用 `close()`，然后再次调用 `showModal()`。这与调用只一次 `showModal()` 的效果相同。也正如在生产环境中看到的那样。

### 订阅事件 
如果 Effect 订阅了某些事件，清理函数应该退订这些事件：

```jsx
useEffect(() => {
  function handleScroll(e) {
    console.log(window.scrollX, window.scrollY);
  }
  window.addEventListener('scroll', handleScroll);
  return () => window.removeEventListener('scroll', handleScroll);
}, []);
```

在开发环境中，Effect 会调用 `addEventListener()`，然后立即调用 `removeEventListener()`，然后再调用相同的 `addEventListener()`，这与只订阅一次事件的 Effect 等效；这也与用户在生产环境中只调用一次 `addEventListener()` 具有相同的感知效果。

