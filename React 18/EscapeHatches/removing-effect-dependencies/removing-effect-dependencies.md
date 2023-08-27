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
