# 使用自定义 Hook 复用逻辑
React 有一些内置 Hook，例如 `useState`，`useContext` 和 `useEffect`。有时你需要一个用途更特殊的 Hook：例如获取数据，记录用户是否在线或者连接聊天室。虽然 React 中可能没有这些 Hook，但是你可以根据应用需求创建自己的 Hook。

## 你将会学习到
+ 什么是自定义 Hook，以及如何编写
+ 如何在组件间重用逻辑
+ 如何给自定义 Hook 命名以及如何构建
+ 提取自定义 Hook 的时机和原因

### 自定义 Hook：组件间共享逻辑 
假设你正在开发一款重度依赖网络的应用（和大多数应用一样）。当用户使用应用时网络意外断开，你需要提醒他。你会怎么处理呢？看上去组件需要两个东西：
1. 一个追踪网络是否在线的 `state`。
2. 一个订阅全局 `online` 和 `offline` 事件并更新上述 `state` 的 Effect。

这会让组件与网络状态保持 同步。你也许可以像这样开始：

```jsx
import { useState, useEffect } from 'react';

export default function StatusBar() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}
```

试着开启和关闭网络，注意观察 `StatusBar` 组件应对你的行为是如何更新的。

假设现在你想在另一个不同的组件里 *也* 使用同样的逻辑。你希望实现一个保存按钮，每当网络断开这个按钮就会不可用并且显示 “Reconnecting…” 而不是 “Save progress”。

你可以从复制粘贴 `isOnline` state 和 Effect 到 `SaveButton` 组件开始：

```jsx
import { useState, useEffect } from 'react';

export default function SaveButton() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  function handleSaveClick() {
    console.log('✅ Progress saved');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? 'Save progress' : 'Reconnecting...'}
    </button>
  );
}
```

如果你关闭网络，可以发现这个按钮的外观变了。

这两个组件都能很好地工作，但不幸的是他们的逻辑重复了。他们看上去有不同的 *视觉外观*，但你依然想复用他们的逻辑。