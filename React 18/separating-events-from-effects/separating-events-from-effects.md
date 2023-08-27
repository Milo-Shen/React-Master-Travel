# 将事件从 Effect 中分开
事件处理函数只有在你再次执行同样的交互时才会重新运行。Effect 和事件处理函数不一样，它只有在读取的 props 或 state 值和上一次渲染不一样时才会重新同步。有时你需要这两种行为的混合体：即一个 Effect 只在响应某些值时重新运行，但是在其他值变化时不重新运行。本章将会教你怎么实现这一点。

## 你将会学习到
+ 怎么在事件处理函数和 Effect 之间做选择
+ 为什么 Effect 是响应式的，而事件处理函数不是
+ 当你想要 Effect 的部分代码变成非响应式时要做些什么
+ Effect Event 是什么，以及怎么从 Effect 中提取
+ 怎么使用 Effect Event 读取最新的 props 和 state

### 在事件处理函数和 Effect 中做选择 
首先让我们回顾一下事件处理函数和 Effect 的区别。

假设你正在实现一个聊天室组件，需求如下：

1. 组件应该自动连接选中的聊天室。
2. 每当你点击 “Send” 按钮，组件应该在当前聊天界面发送一条消息。

假设你已经实现了这部分代码，但是还没有确定应该放在哪里。你是应该用事件处理函数还是 Effect 呢？每当你需要回答这个问题时，请考虑一下 为什么代码需要运行。

### 事件处理函数只在响应特定的交互操作时运行 
从用户角度出发，发送消息是 *因为* 他点击了特定的“Send”按钮。如果在任意时间或者因为其他原因发送消息，用户会觉得非常混乱。这就是为什么发送消息应该使用事件处理函数。事件处理函数是让你处理特定的交互操作的：

```jsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');
  // ...
  function handleSendClick() {
    sendMessage(message);
  }
  // ...
  return (
    <>
      <input value={message} onChange={e => setMessage(e.target.value)} />
      <button onClick={handleSendClick}>Send</button>;
    </>
  );
}
```

借助事件处理函数，你可以确保 `sendMessage(message)` *只* 在用户点击按钮的时候运行。

### 每当需要同步，Effect 就会运行 
回想一下，你还需要让组件和聊天室保持连接。代码放哪里呢？

运行这个代码的 *原因* 不是特定的交互操作。用户为什么或怎么导航到聊天室屏幕的都不重要。既然用户正在看它并且能够和它交互，组件就要和选中的聊天服务器保持连接。即使聊天室组件显示的是应用的初始屏幕，用户根本还没有执行任何交互，仍然应该需要保持连接。这就是这里用 Effect 的原因：

```jsx
function ChatRoom({ roomId }) {
  // ...
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

无论 用户是否执行指定交互操作，这段代码都可以保证当前选中的聊天室服务器一直有一个活跃连接。用户是否只启动了应用，或选中了不同的聊天室，又或者导航到另一个屏幕后返回，Effect 都可以确保组件和当前选中的聊天室保持同步，并在必要时 重新连接。

#### App.js
```jsx
import { useState, useEffect } from 'react';
import { createConnection, sendMessage } from './chat.js';

const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]);

  function handleSendClick() {
    sendMessage(message);
  }

  return (
    <>
      <h1>Welcome to the {roomId} room!</h1>
      <input value={message} onChange={e => setMessage(e.target.value)} />
      <button onClick={handleSendClick}>Send</button>
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

#### chat.js
```jsx
export function sendMessage(message) {
  console.log('🔵 You sent: ' + message);
}

export function createConnection(serverUrl, roomId) {
  // 真正的实现实际上会连接到服务器
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

### 响应式值和响应式逻辑 
直观上，你可以说事件处理函数总是“手动”触发的，例如点击按钮。另一方面， Effect 是自动触发：每当需要保持同步的时候他们就会开始运行和重新运行。

有一个更精确的方式来考虑这个问题。

组件内部声明的 `state` 和 `props` 变量被称为  响应式值。本示例中的 `serverUrl` 不是响应式值，但 `roomId` 和 `message` 是。他们参与组件的渲染数据流：

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  // ...
}
```

像这样的响应式值可以因为重新渲染而变化。例如用户可能会编辑 message 或者在下拉菜单中选中不同的 roomId。事件处理函数和 Effect 对于变化的响应是不一样的：

*事件处理函数内部的逻辑是非响应式的。* 除非用户又执行了同样的操作（例如点击），否则这段逻辑不会再运行。事件处理函数可以在“不响应”他们变化的情况下读取响应式值。
*Effect 内部的逻辑是响应式的。* 如果 Effect 要读取响应式值，你必须将它指定为依赖项。如果接下来的重新渲染引起那个值变化，React 就会使用新值重新运行 Effect 内的逻辑。

让我们重新看看前面的示例来说明差异。

### 事件处理函数内部的逻辑是非响应式的
看这行代码。这个逻辑是响应式的吗？

```jsx
    // ...
    sendMessage(message);
    // ...
```

从用户角度出发，*message 的变化并不意味着他们想要发送消息。* 它只能表明用户正在输入。换句话说，发送消息的逻辑不应该是响应式的。它不应该仅仅因为 响应式值 变化而再次运行。这就是应该把它归入事件处理函数的原因：

```jsx
  function handleSendClick() {
    sendMessage(message);
  }
```

事件处理函数是非响应式的，所以 `sendMessage(message)` 只会在用户点击“Send”按钮的时候运行。

### Effect 内部的逻辑是响应式的 
现在让我们返回这几行代码：

```jsx
    // ...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    // ...
```

从用户角度出发，* `roomId` 的变化意味着他们的确想要连接到不同的房间。* 换句话说，连接房间的逻辑应该是响应式的。你 *需要* 这几行代码和响应式值“保持同步”，并在值不同时再次运行。这就是它被归入 Effect 的原因：

```jsx
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect()
    };
  }, [roomId]);
```

Effect 是响应式的，所以 `createConnection(serverUrl, roomId)` 和 `connection.connect()` 会因为 `roomId` 每个不同的值而运行。Effect 让聊天室连接和当前选中的房间保持了同步。