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

### 从组件中提取自定义 Hook 
假设有一个内置 Hook `useOnlineStatus`，它与 `useState` 和 `useEffect` 相似。那么你就可以简化这两个组件并移除他们之间的重复部分：

```jsx
function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

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

尽管目前还没有这样的内置 Hook，但是你可以自己写。声明一个 `useOnlineStatus` 函数，并把组件里早前写的所有重复代码移入该函数：

```jsx
function useOnlineStatus() {
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
  return isOnline;
}
```

在函数结尾处返回 `isOnline`。这可以让组件读取到该值：

#### App.js
```jsx
import { useOnlineStatus } from './useOnlineStatus.js';

function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    console.log('✅ Progress saved');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? 'Save progress' : 'Reconnecting...'}
    </button>
  );
}

export default function App() {
  return (
    <>
      <SaveButton />
      <StatusBar />
    </>
  );
}
```

#### useOnlineStatus.js
```jsx
import { useState, useEffect } from 'react';

export function useOnlineStatus() {
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
  return isOnline;
}
```

切换网络状态验证一下是否会同时更新两个组件。

现在组件里没有那么多的重复逻辑了。*更重要的是，组件内部的代码描述的是想要做什么（使用在线状态！），而不是怎么做（通过订阅浏览器事件完成）。*

当提取逻辑到自定义 Hook 时，你可以隐藏如何处理外部系统或者浏览器 API 这些乱七八糟的细节。组件内部的代码表达的是目标而不是具体实现。

### Hook 的名称必须永远以 `use` 开头 
React 应用是由组件构成，而组件由内置或自定义 Hook 构成。可能你经常使用别人写的自定义 Hook，但偶尔也要自己写！

你必须遵循以下这些命名公约：
1. *React 组件名称必须以大写字母开头*，比如 `StatusBar` 和 `SaveButton`。React 组件还需要返回一些 React 能够显示的内容，比如一段 JSX。
2. *Hook 的名称必须以后跟一个大写字母的 `use` 开头*，像 `useState`（内置） 或者 `useOnlineStatus`（像本文早前的自定义 Hook）。Hook 可以返回任意值。

这个公约保证你始终能一眼识别出组件并且知道它的 `state`，Effect 以及其他的 React 特性可能“隐藏”在哪里。例如如果你在组件内部看见 `getColor()` 函数调用，就可以确定它里面不可能包含 React state，因为它的名称没有以 `use` 开头。但是像 `useOnlineStatus()` 这样的函数调用就很可能包含对内部其他 Hook 的调用！

### 注意
如果你为 React 配置了 代码检查工具，它会强制执行这个命名公约。现在滑动到上面的 `sandbox`，并将 `useOnlineStatus` 重命名为 `getOnlineStatus`。注意此时代码检查工具将不会再允许你在其内部调用 `useState` 或者 `useEffect`。只有 Hook 和组件可以调用其他 Hook！

### 渲染期间调用的所有函数都应该以 use 前缀开头么？ 
不。没有 *调用* Hook 的函数不需要 *变成* Hook。

如果函数没有调用任何 Hook，请避免使用 `use` 前缀。 而是 *不带* `use` 前缀把它当成常规函数去写。例如下面的 `useSorted`  没有调用 Hook，所以叫它 `getSorted`：

```jsx
// 🔴 Avoid: 没有调用其他Hook的Hook
function useSorted(items) {
  return items.slice().sort();
}

// ✅ Good: 没有使用Hook的常规函数
function getSorted(items) {
  return items.slice().sort();
}
```

这保证你的代码可以在包含条件语句在内的任何地方调用这个常规函数：

```jsx
function List({ items, shouldSort }) {
  let displayedItems = items;
  if (shouldSort) {
    // ✅ 在条件分支里调用getSorted()是没问题的，因为它不是Hook
    displayedItems = getSorted(items);
  }
  // ...
}
```

哪怕内部只使用了一个 Hook，你也应该给这个函数加 `use` 前缀（让它成为一个 Hook）：

```jsx
// ✅ Good: 一个使用了其他Hook的Hook
function useAuth() {
  return useContext(Auth);
}
```

技术上 React 对此并不强制要求。原则上你可以写出不调用其他 Hook 的 Hook。但这常常会难以理解且受限，所以最好避免这种方式。但是它在极少数场景下可能是有益的。例如函数目前也许并没有使用任何 Hook，但是你计划未来在该函数内部添加一些 Hook 调用。那么使用 `use` 前缀命名就很有意义：

```jsx
// ✅ Good: 之后可能使用其他Hook的Hook
function useAuth() {
  // TODO: 当认证功能实现以后，替换这一行：
  // 返回 useContext(Auth)；
  return TEST_USER;
}
```

接下来组件就不能在条件语句里调用这个函数。当你在内部实际添加了 Hook 调用时，这一点将变得很重要。如果你（现在或者之后）没有计划在内部使用 Hook，请不要让它变成 Hook。