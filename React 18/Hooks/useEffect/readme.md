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

