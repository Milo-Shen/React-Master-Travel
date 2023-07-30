# useEffect

`useEffect` 是一个 React Hook，它允许你 将组件与外部系统同步。

```jsx
useEffect(setup, dependencies?);
```

## 参考 

```jsx
useEffect(setup, dependencies?);
```

在组件的顶层调用 `useEffect` 来声明一个 Effect：

```jsx
import { useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [serverUrl, roomId]);
  // ...
}
```

### 参数 
+ `setup`：处理 Effect 的函数。setup 函数选择性返回一个 清理（cleanup） 函数。在将组件首次添加到 DOM 之前，React 将运行 setup 函数。在每次依赖项变更重新渲染后，React 将首先使用旧值运行 cleanup 函数（如果你提供了该函数），然后使用新值运行 setup 函数。在组件从 DOM 中移除后，React 将最后一次运行 cleanup 函数。
+ 可选 `dependencies`：setup 代码中引用的所有响应式值的列表。响应式值包括 `props`、`state` 以及所有直接在组件内部声明的变量和函数。如果你的代码检查工具 配置了 React，那么它将验证是否每个响应式值都被正确地指定为一个依赖项。依赖项列表的元素数量必须是固定的，并且必须像 `[dep1, dep2, dep3]` 这样内联编写。React 将使用 `Object.is` 来比较每个依赖项和它先前的值。如果省略此参数，则在每次重新渲染组件之后，将重新运行 Effect 函数。如果你想了解更多，请参见 传递依赖数组、空数组和不传递依赖项之间的区别。

### 返回值 
`useEffect` 返回 `undefined`。

### 注意事项 
+ `useEffect` 是一个 Hook，因此只能在 组件的顶层 或自己的 Hook 中调用它，而不能在循环或者条件内部调用。如果需要，抽离出一个新组件并将 `state` 移入其中。
+ 如果你 没有打算与某个外部系统同步，那么你可能不需要 Effect。
+ 当严格模式启动时，React 将在真正的 setup 函数首次运行前，运行一个开发模式下专有的额外 setup + cleanup 周期。这是一个压力测试，用于确保 cleanup 逻辑“映射”到了 setup 逻辑，并停止或撤消 setup 函数正在做的任何事情。如果这会导致一些问题，请实现 cleanup 函数。
+ 如果你的一些依赖项是组件内部定义的对象或函数，则存在这样的风险，即它们将 导致 Effect 过多地重新运行。要解决这个问题，请删除不必要的 对象 和 函数 依赖项。你还可以 抽离状态更新 和 非响应式的逻辑 到 Effect 之外。
+ 如果你的 Effect 不是由交互（比如点击）引起的，那么 React 会让浏览器 在运行 Effect 前先绘制出更新后的屏幕。如果你的 Effect 正在做一些视觉相关的事情（例如，定位一个 tooltip），并且有显著的延迟（例如，它会闪烁），那么将 `useEffect` 替换为 `useLayoutEffect`。
+ 即使你的 Effect 是由一个交互（比如点击）引起的，浏览器也可能在处理 Effect 内部的状态更新之前重新绘制屏幕。通常，这就是你想要的。但是，如果你一定要阻止浏览器重新绘制屏幕，则需要用 `useLayoutEffect` 替换 `useEffect`。
+ *Effect* 只在客户端上运行，在服务端渲染中不会运行。

## 用法 

### 连接到外部系统 

有些组件需要与网络、某些浏览器 API 或第三方库保持连接，当它们显示在页面上时。这些系统不受 React 控制，所以称为外部系统。

要 将组件连接到某个外部系统，请在组件的顶层调用 useEffect：

```jsx
import { useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
  	const connection = createConnection(serverUrl, roomId);
    connection.connect();
  	return () => {
      connection.disconnect();
  	};
  }, [serverUrl, roomId]);
  // ...
}
```

你需要向 `useEffect` 传递两个参数：
1. 一个 setup 函数 ，其 setup 代码 用来连接到该系统。
   + 它应该返回一个 清理函数（cleanup），其 cleanup 代码 用来与该系统断开连接。
2. 一个 依赖项列表，包括这些函数使用的每个组件内的值。

*React 在必要时会调用 setup 和 cleanup，这可能会发生多次：*
1. 将组件挂载到页面时，将运行 setup 代码。
2. 重新渲染 依赖项 变更的组件后：
    + 首先，使用旧的 props 和 state 运行 cleanup 代码。
    + 然后，使用新的 props 和 state 运行 setup 代码。
3. 当组件从页面卸载后，cleanup 代码 将运行最后一次。

*用上面的代码作为例子来解释这个顺序。*+

当 `ChatRoom` 组件添加到页面中时，它将使用 `serverUrl` 和 `roomId` 初始值连接到聊天室。如果 `serverUrl` 或者 `roomId` 发生改变并导致重新渲染（比如用户在下拉列表中选择了一个不同的聊天室），那么 `Effect` 就会 断开与前一个聊天室的连接，并连接到下一个聊天室。当 `ChatRoom` 组件从页面中卸载时，你的 `Effect` 将最后一次断开连接。

为了 帮助你发现 bug，在开发环境下，React 在运行 `setup` 之前会额外运行一次 `setup` 和 `cleanup`。这是一个压力测试，用于验证 `Effect` 逻辑是否正确实现。如果这会导致可见的问题，那么你的 `cleanup` 函数就缺少一些逻辑。`cleanup` 函数应该停止或撤消 `setup` 函数正在执行的任何操作。一般来说，用户不应该能够区分只调用一次 `setup`（在生产环境中）与调用 `setup` → `cleanup` → `setup` 序列（在开发环境中）。查看常见解决方案。

尽量 将每个 `Effect` 作为一个独立的过程编写，并且 每次只考虑一个单独的 `setup/cleanup` 周期。组件是否正在挂载、更新或卸载并不重要。当你的 `cleanup` 逻辑正确地“映射”到 `setup` 逻辑时，你的 Effect 是可复原的，因此可以根据需要多次运行 `setup` 和 `cleanup` 函数。

### 注意 Effect 可以让你的组件与某些外部系统（比如聊天服务）保持同步。在这里，外部系统是指任何不受 React 控制的代码，例如：
1. 由 `setInterval()` 和 `clearInterval()` 管理的定时器。
2. 使用 `window.addEventListener()` 和 `window.removeEventListener()` 的事件订阅。
3. 一个第三方的动画库，它有一个类似 `animation.start()` 和 `animation.reset()` 的 API。

*如果你没有连接到任何外部系统，你或许不需要 Effect。*

### 连接到外部系统的示例

#### 第 1 个示例 共 5 个挑战: 连接到聊天服务器 
在这个示例中，`ChatRoom` 组件使用一个 Effect 来保持与 `chat.js` 中定义的外部系统的连接。点击“打开聊天”以显示 `ChatRoom` 组件。这个沙盒在开发模式下运行，因此有一个额外的“连接并断开”的周期，就像 这里描述的 一样。尝试使用下拉菜单和输入框更改 `roomId` 和 `serverUrl`，并查看 `Effect` 如何重新连接到聊天。点击“关闭聊天”可以看到 `Effect` 最后一次断开连接。

##### App.js
```jsx
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]);

  return (
    <>
      <label>
        Server URL:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

##### chat.js

```jsx
export function createConnection(serverUrl, roomId) {
  // 真正的实现会实际连接到服务器
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    }
  };
}
```

#### 第 2 个示例 共 5 个挑战: 监听全局的浏览器事件 
在这个例子中，外部系统就是浏览器 DOM 本身。通常，你会使用 JSX 指定事件监听，但是你不能以这种方式监听全局的 `window` 对象。你可以通过 `Effect` 连接到 `window` 对象并监听其事件。如监听 `pointermove` 事件可以让你追踪光标（或手指）的位置，并更新红点以随之移动。

```jsx
import { useState, useEffect } from 'react';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useEffect(() => {
    function handleMove(e) {
      setPosition({ x: e.clientX, y: e.clientY });
    }
    window.addEventListener('pointermove', handleMove);
    return () => {
      window.removeEventListener('pointermove', handleMove);
    };
  }, []);

  return (
    <div style={{
      position: 'absolute',
      backgroundColor: 'pink',
      borderRadius: '50%',
      opacity: 0.6,
      transform: `translate(${position.x}px, ${position.y}px)`,
      pointerEvents: 'none',
      left: -20,
      top: -20,
      width: 40,
      height: 40,
    }} />
  );
}
```

#### 第 3 个示例 共 5 个挑战: 触发动画效果
在这个例子中，外部系统是 `animation.js` 中的动画库。它提供了一个名为 `FadeInAnimation` 的 JavaScript 类，该类接受一个 DOM 节点作为参数，并暴露 `start()` 和 `stop()` 方法来控制动画。此组件 使用 `ref` 访问底层 DOM 节点。`Effect` 从 `ref` 中读取 DOM 节点，并在组件出现时自动开启该节点的动画。

##### App.js

```jsx
import { useState, useEffect, useRef } from 'react';
import { FadeInAnimation } from './animation.js';

function Welcome() {
  const ref = useRef(null);

  useEffect(() => {
    const animation = new FadeInAnimation(ref.current);
    animation.start(1000);
    return () => {
      animation.stop();
    };
  }, []);

  return (
    <h1
      ref={ref}
      style={{
        opacity: 0,
        color: 'white',
        padding: 50,
        textAlign: 'center',
        fontSize: 50,
        backgroundImage: 'radial-gradient(circle, rgba(63,94,251,1) 0%, rgba(252,70,107,1) 100%)'
      }}
    >
      Welcome
    </h1>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? 'Remove' : 'Show'}
      </button>
      <hr />
      {show && <Welcome />}
    </>
  );
}
```

##### animation.js

```jsx
export class FadeInAnimation {
  constructor(node) {
    this.node = node;
  }
  start(duration) {
    this.duration = duration;
    if (this.duration === 0) {
      // 立刻跳转到最后
      this.onProgress(1);
    } else {
      this.onProgress(0);
      // 开始动画
      this.startTime = performance.now();
      this.frameId = requestAnimationFrame(() => this.onFrame());
    }
  }
  onFrame() {
    const timePassed = performance.now() - this.startTime;
    const progress = Math.min(timePassed / this.duration, 1);
    this.onProgress(progress);
    if (progress < 1) {
      // 仍然有更多的帧要绘制
      this.frameId = requestAnimationFrame(() => this.onFrame());
    }
  }
  onProgress(progress) {
    this.node.style.opacity = progress;
  }
  stop() {
    cancelAnimationFrame(this.frameId);
    this.startTime = null;
    this.frameId = null;
    this.duration = 0;
  }
}
```

#### 第 4 个示例 共 5 个挑战: 控制模态对话框
在这个例子中，外部系统是浏览器 DOM。`ModalDialog` 组件渲染一个 `<dialog>` 元素。它使用 `Effect` 将 `isOpen` prop 同步到 `showModal()` 和 `close()` 方法调用。

##### App.js
```jsx
import { useState } from 'react';
import ModalDialog from './ModalDialog.js';

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(true)}>
        Open dialog
      </button>
      <ModalDialog isOpen={show}>
        Hello there!
        <br />
        <button onClick={() => {
          setShow(false);
        }}>Close</button>
      </ModalDialog>
    </>
  );
}
```

##### ModalDialog.js

```jsx
import { useEffect, useRef } from 'react';

export default function ModalDialog({ isOpen, children }) {
  const ref = useRef();

  useEffect(() => {
    if (!isOpen) {
      return;
    }
    const dialog = ref.current;
    dialog.showModal();
    return () => {
      dialog.close();
    };
  }, [isOpen]);

  return <dialog ref={ref}>{children}</dialog>;
}
```

#### 第 5 个示例 共 5 个挑战: 跟踪元素可见性 
在这个例子中，外部系统仍然是浏览器 DOM。`App` 组件展示一个长列表，然后是 `Box` 组件，然后是另一个长列表。试试向下滚动列表。请注意，当 `Box` 组件出现在视野中时，背景色会变成黑色。为了实现这一点，`Box` 组件使用 `Effect` 来管理 `IntersectionObserver`。这个浏览器 API 会在视野中出现指定 DOM 元素时通知你。

##### App.js
```jsx
import Box from './Box.js';

export default function App() {
  return (
    <>
      <LongSection />
      <Box />
      <LongSection />
      <Box />
      <LongSection />
    </>
  );
}

function LongSection() {
  const items = [];
  for (let i = 0; i < 50; i++) {
    items.push(<li key={i}>Item #{i} (keep scrolling)</li>);
  }
  return <ul>{items}</ul>
}
```

##### Box.js
```jsx
import { useRef, useEffect } from 'react';

export default function Box() {
  const ref = useRef(null);

  useEffect(() => {
    const div = ref.current;
    const observer = new IntersectionObserver(entries => {
      const entry = entries[0];
      if (entry.isIntersecting) {
        document.body.style.backgroundColor = 'black';
        document.body.style.color = 'white';
      } else {
        document.body.style.backgroundColor = 'white';
        document.body.style.color = 'black';
      }
    });
    observer.observe(div, {
      threshold: 1.0
    });
    return () => {
      observer.disconnect();
    }
  }, []);

  return (
    <div ref={ref} style={{
      margin: 20,
      height: 100,
      width: 100,
      border: '2px solid black',
      backgroundColor: 'blue'
    }} />
  );
}
```

### 在自定义 Hook 中封装 Effect 
Effect 是一个 “逃生出口”：当你需要“走出 React 之外”或者当你的使用场景没有更好的内置解决方案时，你可以使用它们。如果你发现自己经常需要手动编写 Effect，那么这通常表明你需要为组件所依赖的通用行为提取一些 自定义 Hook。

例如，这个 `useChatRoom` 自定义 Hook 把 Effect 的逻辑“隐藏”在一个更具声明性的 API 之后：

```jsx
function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

```jsx
function ChatRoom({ roomId }) {
    const [serverUrl, setServerUrl] = useState('https://localhost:1234');

    useChatRoom({
        roomId: roomId,
        serverUrl: serverUrl
    });
    // ...
}
```

在 React 生态系统中，还有许多用于各种用途的优秀的自定义 Hook。

### 自定义 Hook 中封装 Effect 示例

#### 第 1 个示例 共 3 个挑战: 定制 `useChatRoom` Hook 
此示例与 前面的一个示例 相同，但是逻辑被提取到一个自定义 Hook 中。

##### App.js
```jsx
import { useState } from 'react';
import { useChatRoom } from './useChatRoom.js';

function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl
  });

  return (
    <>
      <label>
        Server URL:{' '}
        <input
          value={serverUrl}
          onChange={e => setServerUrl(e.target.value)}
        />
      </label>
      <h1>Welcome to the {roomId} room!</h1>
    </>
  );
}

export default function App() {
  const [roomId, setRoomId] = useState('general');
  const [show, setShow] = useState(false);
  return (
    <>
      <label>
        Choose the chat room:{' '}
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="general">general</option>
          <option value="travel">travel</option>
          <option value="music">music</option>
        </select>
      </label>
      <button onClick={() => setShow(!show)}>
        {show ? 'Close chat' : 'Open chat'}
      </button>
      {show && <hr />}
      {show && <ChatRoom roomId={roomId} />}
    </>
  );
}
```

##### useChatRoom.js
```jsx
import { useEffect } from 'react';
import { createConnection } from './chat.js';

export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId, serverUrl]);
}
```

##### chat.js
```jsx
export function createConnection(serverUrl, roomId) {
  // 真正的实现将实际连接到服务器
  return {
    connect() {
      console.log('✅ Connecting to "' + roomId + '" room at ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ Disconnected from "' + roomId + '" room at ' + serverUrl);
    }
  };
}
```

#### 第 2 个示例 共 3 个挑战: 定制 useWindowListener Hook 
此示例与 前面的一个示例 相同，但是逻辑被提取到一个自定义 Hook 中。

##### App.js
```jsx
import { useState } from 'react';
import { useWindowListener } from './useWindowListener.js';

export default function App() {
  const [position, setPosition] = useState({ x: 0, y: 0 });

  useWindowListener('pointermove', (e) => {
    setPosition({ x: e.clientX, y: e.clientY });
  });

  return (
    <div style={{
      position: 'absolute',
      backgroundColor: 'pink',
      borderRadius: '50%',
      opacity: 0.6,
      transform: `translate(${position.x}px, ${position.y}px)`,
      pointerEvents: 'none',
      left: -20,
      top: -20,
      width: 40,
      height: 40,
    }} />
  );
}
```

##### useWindowListener.js
```jsx
import { useState, useEffect } from 'react';

export function useWindowListener(eventType, listener) {
  useEffect(() => {
    window.addEventListener(eventType, listener);
    return () => {
      window.removeEventListener(eventType, listener);
    };
  }, [eventType, listener]);
}
```

#### 第 3 个示例 共 3 个挑战: 定制 useIntersectionObserver Hook
此示例与 前面的一个示例 相同，但是部分逻辑被提取到自定义 Hook 中。

##### App.js
```jsx
import Box from './Box.js';

export default function App() {
  return (
    <>
      <LongSection />
      <Box />
      <LongSection />
      <Box />
      <LongSection />
    </>
  );
}

function LongSection() {
  const items = [];
  for (let i = 0; i < 50; i++) {
    items.push(<li key={i}>Item #{i} (keep scrolling)</li>);
  }
  return <ul>{items}</ul>
}
```

##### Box.js
```jsx
import { useRef, useEffect } from 'react';
import { useIntersectionObserver } from './useIntersectionObserver.js';

export default function Box() {
  const ref = useRef(null);
  const isIntersecting = useIntersectionObserver(ref);

  useEffect(() => {
   if (isIntersecting) {
      document.body.style.backgroundColor = 'black';
      document.body.style.color = 'white';
    } else {
      document.body.style.backgroundColor = 'white';
      document.body.style.color = 'black';
    }
  }, [isIntersecting]);

  return (
    <div ref={ref} style={{
      margin: 20,
      height: 100,
      width: 100,
      border: '2px solid black',
      backgroundColor: 'blue'
    }} />
  );
}
```

##### useIntersectionObserver.js
```jsx
import { useState, useEffect } from 'react';

export function useIntersectionObserver(ref) {
  const [isIntersecting, setIsIntersecting] = useState(false);

  useEffect(() => {
    const div = ref.current;
    const observer = new IntersectionObserver(entries => {
      const entry = entries[0];
      setIsIntersecting(entry.isIntersecting);
    });
    observer.observe(div, {
      threshold: 1.0
    });
    return () => {
      observer.disconnect();
    }
  }, [ref]);

  return isIntersecting;
}
```

### 控制非 React 小部件 
有时，你希望外部系统与你组件的某些 `props` 或 `state` 保持同步。

例如，如果你有一个没有使用 React 编写的第三方地图小部件或视频播放器组件，你可以使用 Effect 调用该组件上的方法，使其状态与 React 组件的当前状态相匹配。此 Effect 创建了在 `map-widget.js` 中定义的 `MapWidget` 类的实例。当你更改 `Map` 组件的 `zoomLevel` prop 时，`Effect` 调用类实例上的 `setZoom()` 来保持同步：

##### App.js
```jsx
import { useState } from 'react';
import Map from './Map.js';

export default function App() {
  const [zoomLevel, setZoomLevel] = useState(0);
  return (
    <>
      Zoom level: {zoomLevel}x
      <button onClick={() => setZoomLevel(zoomLevel + 1)}>+</button>
      <button onClick={() => setZoomLevel(zoomLevel - 1)}>-</button>
      <hr />
      <Map zoomLevel={zoomLevel} />
    </>
  );
}
```

##### Map.js
```jsx
import { useRef, useEffect } from 'react';
import { MapWidget } from './map-widget.js';

export default function Map({ zoomLevel }) {
  const containerRef = useRef(null);
  const mapRef = useRef(null);

  useEffect(() => {
    if (mapRef.current === null) {
      mapRef.current = new MapWidget(containerRef.current);
    }

    const map = mapRef.current;
    map.setZoom(zoomLevel);
  }, [zoomLevel]);

  return (
    <div
      style={{ width: 200, height: 200 }}
      ref={containerRef}
    />
  );
}
```

##### map-widget.js
```jsx
import 'leaflet/dist/leaflet.css';
import * as L from 'leaflet';

export class MapWidget {
  constructor(domNode) {
    this.map = L.map(domNode, {
      zoomControl: false,
      doubleClickZoom: false,
      boxZoom: false,
      keyboard: false,
      scrollWheelZoom: false,
      zoomAnimation: false,
      touchZoom: false,
      zoomSnap: 0.1
    });
    L.tileLayer('https://tile.openstreetmap.org/{z}/{x}/{y}.png', {
      maxZoom: 19,
      attribution: '© OpenStreetMap'
    }).addTo(this.map);
    this.map.setView([0, 0], 0);
  }
  setZoom(level) {
    this.map.setZoom(level);
  }
}
```