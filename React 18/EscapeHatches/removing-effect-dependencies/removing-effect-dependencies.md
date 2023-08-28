# 移除 Effect 依赖
当编写 Effect 时，linter 会验证是否已经将 Effect 读取的每一个响应式值（如 props 和 state）包含在 Effect 的依赖中。这可以确保 Effect 与组件的 props 和 state 保持同步。不必要的依赖可能会导致 Effect 运行过于频繁，甚至产生无限循环。请按照本指南审查并移除 Effect 中不必要的依赖。

## 你将会学习到
+ 修复无限的 Effect 依赖性循环
+ 当你想移除依赖时，该怎么做
+ 从 Effect 中读取值而不对它作出“反应”
+ 为什么以及如何避免对象和函数的依赖？
+ 为什么抑制依赖代码检查器的检查是危险的，以及应该如何做？

### 依赖应该和代码保持一致
当你编写 Effect 时，无论这个 Effect 要做什么，你首先要明确其 生命周期：

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
    useEffect(() => {
        const connection = createConnection(serverUrl, roomId);
        connection.connect();
        return () => connection.disconnect();
        // ...
    })
}
```

如果你设置 Effect 的依赖是空数组（`[]`），那么 linter 将会建议合适的依赖：

```jsx
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // <-- 修复这里的依赖！
  return <h1>欢迎来到 {roomId} 房间！</h1>;
}

export default function App() {
  const [roomId, setRoomId] = useState('所有');
  return (
    <>
      <label>
        选择聊天室：
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="所有">所有</option>
          <option value="旅游">旅游</option>
          <option value="音乐">音乐</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

Linter 会显示下面的提示:

```
11:6 - React Hook useEffect has a missing dependency: 'roomId'. Either include it or remove the dependency array.
```

按照 linter 的建议，把它们填进去：

```jsx
function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ 所有依赖已声明
  // ...
}
```

Effect “反应”响应式值 因为这里的 `roomId` 是一个响应式值（它可能随重新渲染而改变），所以 `linter` 会验证你是否将它指定为依赖。如果 `roomId` 变成不同的值，React 将重新运行 Effect。这可以确保聊天界面与所选房间保持一致，并把变化“反馈”给下拉菜单：

#### App.js
```jsx
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);
  return <h1>欢迎来到 {roomId} 房间！</h1>;
}

export default function App() {
  const [roomId, setRoomId] = useState('所有');
  return (
    <>
      <label>
        选择聊天室：
        <select
          value={roomId}
          onChange={e => setRoomId(e.target.value)}
        >
          <option value="所有">所有</option>
          <option value="旅游">旅游</option>
          <option value="音乐">音乐</option>
        </select>
      </label>
      <hr />
      <ChatRoom roomId={roomId} />
    </>
  );
}
```

#### chat.js
```jsx
export function createConnection(serverUrl, roomId) {
  // 真正的实现实际上会连接到服务器
  return {
    connect() {
      console.log('✅ 连接到“' + roomId + '”房间，在 ' + serverUrl + '...');
    },
    disconnect() {
      console.log('❌ 断开“' + roomId + '”房间，在 ' + serverUrl);
    }
  };
}
```

### 当要移除一个依赖时，请证明它不是一个依赖 
注意，你不能“选择” Effect 的依赖。每个被 Effect 所使用的响应式值，*必须* 在依赖中声明。依赖是由 Effect 的代码决定的：

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) { // 这是一个响应式值
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId); // Effect 在这里读取响应式值
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ 所以你必须在依赖中声明 Effect 使用的响应式值
  // ...
}
```

响应式值 包括 `props` 以及所有你直接在组件中声明的变量和函数。由于 `roomId` 是响应式值，你不能把它从依赖中移除。linter 不允许这样做：

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // 🔴 React Hook useEffect 缺失依赖: 'roomId'
  // ...
}
```

linter 是对的！ 由于 `roomId` 可能会随时间变化，这会在代码中引入错误。

*移除一个依赖，你需要向 linter 证明其不需要这个依赖。* 例如，你可以将 `roomId` 移出组件，以证明它不是响应的，也不会在重新渲染时改变：

```jsx
const serverUrl = 'https://localhost:1234';
const roomId = '音乐'; // 不再是响应式值

function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []); // ✅ 所有依赖已声明
  // ...
}
```

现在 `roomId` 不是响应式值（并且不能在重新渲染时改变），那它不就不是依赖：

```jsx
import { useState, useEffect } from 'react';
import { createConnection } from './chat.js';

const serverUrl = 'https://localhost:1234';
const roomId = '音乐';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, []);
  return <h1>欢迎来到 {roomId} 房间！</h1>;
}
```

这就是为什么你现在可以指定 空（`[]`）依赖。Effect *真的不* 依赖任何响应式值了，也 *真的不* 需要在组件的 props 或 state 改变时重新运行。

### 要改变依赖，请改变代码 
你可能已经注意到工作流程中有一个模式：
1. 首先，你 *改变 Effect 的代码* 或响应式值的声明方式。
2. 然后，你采纳 linter 的建议，调整依赖，以 *匹配你所改变的代码*。
3. 如果你对依赖不满意，你可以 *回到第一步*（并再次修改代码）。

最后一部分很重要。*如果你想改变依赖，首先要改变所涉及到的代码。* 你可以把依赖看作是 Effect的代码所依赖的所有响应式值的列表。你不要 选择 把什么放在这个列表上。该列表 *描述了* 代码。要改变依赖，请改变代码。

这可能感觉就像解方程一样。你有一个目标（例如，移除一个依赖），你需要“找到”与该目标相匹配的代码。不是每个人都觉得解方程很有趣，写 Effect 也是如此！幸运的是，下面有一些常见的解决方案你可以去尝试。

### 陷阱
如果你有一个已经存在的代码库，你可能会有一些像这样抑制 linter 的代码：

```jsx
useEffect(() => {
  // ...
  // 🔴 避免像这样抑制 linter 的警告或错误提示：
  // eslint-ignore-next-line react-hooks/exhaustive-deps
}, []);
```

*当依赖与代码不匹配时，极有可能引入 `bug`。* 通过抑制 linter，你是在 Effect 所依赖的值上对 React “撒谎”。

你可以使用如下技术。

### 为什么抑制 linter 对依赖的检查如此危险？
抑制 linter 会导致非常不直观的 bug，这将很难发现和修复。这里有一个例子：

```jsx
import { useState, useEffect } from 'react';

export default function Timer() {
  const [count, setCount] = useState(0);
  const [increment, setIncrement] = useState(1);

  function onTick() {
	setCount(count + increment);
  }

  useEffect(() => {
    const id = setInterval(onTick, 1000);
    return () => clearInterval(id);
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);

  return (
    <>
      <h1>
        计数器：{count}
        <button onClick={() => setCount(0)}>重制</button>
      </h1>
      <hr />
      <p>
        每秒递增：
        <button disabled={increment === 0} onClick={() => {
          setIncrement(i => i - 1);
        }}>–</button>
        <b>{increment}</b>
        <button onClick={() => {
          setIncrement(i => i + 1);
        }}>+</button>
      </p>
    </>
  );
}
```

比方说，你想“只在 mount 时”运行 Effect。你已经知道可以通过设置 空（`[]`）依赖 来达到这种效果，所以你决定忽略 linter 的检查，强行指定 `[]` 为依赖。

上面的计数器例子，本应该每秒递增，递增量可以通过两个按钮来控制。然而，由于你对 React “撒谎”，说这个 Effect 不依赖于任何东西，React 便一直使用初次渲染时的 `onTick` 函数。在后续渲染中， `count` 总是 `0` ，`increment` 总是 `1`。为什么？因为定时器每秒调用 `onTick` 函数，实际运行的是 `setCount(0 + 1)`，所以你总是看到 `1`。像这样的错误，当它们分散在多个组件中时，就更难解决了。

这里有一个比忽略 linter 更好的解决方案! 那便是将 `onTick` 添加到依赖中。(为了确保 `interval` 只设置一次，使 `onTick` 成为 Effect Event。)

*我们建议将依赖性 lint 错误作为一个编译错误来处理。如果你不抑制它，你将永远不会遇到像上面这样的错误*。 本页面的剩下部分将介绍这个和其他情况的替代方案。

### 移除非必需的依赖 
每当你调整 Effect 的依赖以适配代码时，请注意一下当前的依赖。当这些依赖发生变化时，让 Effect 重新运行是否有意义？有时，答案是“不”：
+ 你可能想在不同的条件下重新执行 Effect 的 *不同部分*。
+ 你可能想只读取某个依赖的 *最新值*，而不是对其变化做出“反应”。
+ 依赖可能会因为它的类型是对象或函数而 *无意间* 改变太频繁。

为了找到正确的解决方案，你需要回答关于 Effect 的几个问题。让我们来看看这些问题。

### 这段代码应该移到事件处理程序中吗？ 
你应该考虑的第一件事是，这段代码是否应该成为 Effect。

想象一个表单，在提交时你将 `submitted` 状态变量设置为 `true`，并在 `submitted` 为 `true` 时，需要发送 POST 请求并显示通知。你把这个逻辑放在 Effect 内，并根据 `submitted` 为 `true` “反应”。

```jsx
function Form() {
  const [submitted, setSubmitted] = useState(false);

  useEffect(() => {
    if (submitted) {
      // 🔴 避免: Effect 中有特定事件的逻辑
      post('/api/register');
      showNotification('Successfully registered!');
    }
  }, [submitted]);

  function handleSubmit() {
    setSubmitted(true);
  }

  // ...
}
```

后来，你想通过读取当前的主题值来调整通知信息的样式。因为 `theme` 是在组件中声明的，所以它是响应式值，你决定把它作为依赖加入：

```jsx
function Form() {
  const [submitted, setSubmitted] = useState(false);
  const theme = useContext(ThemeContext);

  useEffect(() => {
    if (submitted) {
      // 🔴 避免: Effect 中有特定事件的逻辑
      post('/api/register');
      showNotification('Successfully registered!', theme);
    }
  }, [submitted, theme]); // ✅ 所有依赖已声明

  function handleSubmit() {
    setSubmitted(true);
  }  

  // ...
}
```

如果这么做，你将引入一个错误。想象一下，你先提交表单，然后切换暗亮主题。当 `theme` 改变后，Effect 重新运行，这将导致显示两次相同的通知！

*首先，这里的问题是，代码不应该以 Effect 实现*。你想发送这个 POST 请求，并在 *提交表单时显示通知*，这是一个特定的交互。特定的交互请将该逻辑直接放到相应的事件处理程序中：

```jsx
function Form() {
  const theme = useContext(ThemeContext);

  function handleSubmit() {
    // ✅ 好：从事件处理程序调用特定于事件的逻辑
    post('/api/register');
    showNotification('Successfully registered!', theme);
  }  

  // ...
}
```

现在，代码在事件处理程序中，它不是响应式的 —— 所以它只在用户提交表单时运行。阅读更多关于 在事件处理程序和 Effect 之间做出选择 和 如何删除不必要的 Effect。
