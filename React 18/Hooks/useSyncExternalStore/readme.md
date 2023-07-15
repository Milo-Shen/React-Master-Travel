# useSyncExternalStore
`useSyncExternalStore` 是一个可以让开发者订阅外部 store 的 React Hook
`const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)`

## 参考
### `useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)`

在组件顶层调用 useSyncExternalStore 以从外部 store 读取值。

```jsx
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

function TodosApp() {
  const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
  // ...
}
```

它返回 store 中数据的快照。你需要传两个函数作为参数：
1. `subscribe` 函数应当订阅该 store 并返回一个取消订阅的函数。
2. `getSnapshot` 函数应当从该 store 读取数据的快照。

### 参数:
1. `subscribe`：一个函数，接收一个单独的 callback 参数并把它订阅到 store 上。当 store 发生改变，它应当调用被提供的 callback。这会导致组件重新渲染。subscribe 函数会返回清除订阅的函数。
2. `getSnapshot`：一个函数，返回组件需要的 store 中的数据快照。在 store 不变的情况下，重复调用 getSnapshot 必须返回同一个值。如果 store 改变，并且返回值也不同了（ 用 `Object.is` 比较 ），React 就会重新渲染组件。
3. 可选 `getServerSnapshot`：一个函数，返回 store 中数据的初始快照。它只会在服务端渲染时，以及在客户端进行服务端渲染内容的 hydration 时被用到。快照在服务端与客户端之间必须相同，它通常是从服务端序列化并传到客户端的。如果你忽略此参数，在服务端渲染这个组件会抛出一个错误。

### 返回值: 
该 store 的当前快照，可以在你的渲染逻辑中使用。

### 警告:
1. `getSnapshot` 返回的 store 快照必须是不可变的。如果底层 store 有可变数据，要在数据改变时返回一个新的不可变快照。否则，返回上次缓存的快照。
2. 如果在重新渲染时传入一个不同的 subscribe 函数，React 会用新传入的 subscribe 函数重新订阅该 store。你可以通过在组件外声明 subscribe 来避免。

## 使用

### 订阅外部 store 
你的多数组件只会从它们的 props，state，以及 context 读取数据。然而，有时一个组件需要从一些 React 之外的 store 读取一些随时间变化的数据。这包括：
1. 在 React 之外持有状态的第三方状态管理库
2. 暴露出一个可变值及订阅其改变事件的浏览器 API
在组件顶层调用 useSyncExternalStore 以从外部 store 读取值。

```jsx
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

function TodosApp() {
  const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
  // ...
}
```

它返回 store 中数据的 快照。你需要传两个函数作为参数：
1. subscribe 函数应当订阅 store 并返回一个取消订阅函数。
2. getSnapshot 函数应当从 store 中读取数据的快照。
React 会用这些函数来保持你的组件订阅到 store 并在它改变时重新渲染。
例如，在下面的沙盒中，`todosStore` 被实现为一个保存 React 之外数据的外部 store。`TodosApp` 组件通过 `useSyncExternalStore` Hook 连接到外部 store。

#### App.js
```jsx
export default function TodosApp() {
  const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
  return (
    <>
      <button onClick={() => todosStore.addTodo()}>Add todo</button>
      <hr />
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>{todo.text}</li>
        ))}
      </ul>
    </>
  );
}
```

#### todoStore.js
```jsx

let nextId = 0;
let todos = [{ id: nextId++, text: 'Todo #1' }];
let listeners = [];

export const todosStore = {
  addTodo() {
    todos = [...todos, { id: nextId++, text: 'Todo #' + nextId }]
    emitChange();
  },
    
  subscribe(listener) {
    listeners = [...listeners, listener];
    return () => {
      listeners = listeners.filter(l => l !== listener);
    };
  },
    
  getSnapshot() {
    return todos;
  }
};

function emitChange() {
  for (let listener of listeners) {
    listener();
  }
}
```

### 订阅浏览器 API 

```jsx
import { useSyncExternalStore } from 'react';

export default function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

function getSnapshot() {
  return navigator.onLine;
}

function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}
```

### 把逻辑抽取到自定义 Hook 
通常不会在组件里直接用 `useSyncExternalStore`，而是在自定义 Hook 里调用它。这使得你可以在不同组件里使用相同的外部 store。
例如：这里自定义的 `useOnlineStatus` Hook 追踪网络是否在线：

#### seOnlineStatus.js
```jsx
import { useSyncExternalStore } from 'react';

export function useOnlineStatus() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  return isOnline;
}

function getSnapshot() {
  return navigator.onLine;
}

function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}
```

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

### 添加服务端渲染支持 
如果你的 React 应用使用 服务端渲染，你的 React 组件也会运行在浏览器环境之外来生成初始 HTML。这给连接到外部 store 造成了一些挑战：
1. 如果你连接到一个浏览器特有的 API，因为它在服务端不存在，所以是不可行的。
2. 如果你连接到一个第三方 store，数据要在服务端和客户端之间相匹配。
为了解决这些问题，要传一个 getServerSnapshot 函数作为第三个参数给 useSyncExternalStore：

```jsx
import { useSyncExternalStore } from 'react';

export function useOnlineStatus() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot);
  return isOnline;
}

function getSnapshot() {
  return navigator.onLine;
}

function getServerSnapshot() {
  return true; // 服务端生成的 HTML 总是显示“在线”
}

function subscribe(callback) {
  // ...
}
```

getServerSnapshot 函数与 getSnapshot 相似，但它只在两种情况下才运行：
1. 在服务端生成 HTML 时。
2. 在客户端 hydration 时，即：当 React 拿到服务端的 HTML 并使其可交互。

这使得你能提供在应用可交互前可用的初始快照值。如果没有对服务器端渲染来说有意义的初始值，就省略这个参数来 强制客户端渲染。

#### 注意
确保客户端初始渲染与服务端渲染时 `getServerSnapshot` 返回完全相同的数据。例如，如果在服务端 `getServerSnapshot` 返回一些预先载入的 store 内容，你就需要把这些内容也传给客户端。一种方法是在服务端渲染时，生成 `<script>` 标签来设置像 `window.MY_STORE_DATA` 这样的全局变量，并在客户端 `getServerSnapshot` 内读取此全局变量。你的外部 store 应当提供如何这样做的说明。

## 疑难解答 

### 我的 subscribe 函数每次重新渲染都被调用 
这里的 subscribe 函数是在组件 内部 定义的，所以它每次渲染都不同：

```jsx
function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  
  // 🚩 总是不同的函数，所以 React 每次重新渲染都会重新订阅
  function subscribe() {
    // ...
  }

  // ...
}
```

如果重新渲染时你传一个不同的 `subscribe` 函数，React 会重新订阅你的 store。如果这造成了性能问题，因而你想避免重新订阅，就把 `subscribe` 函数移到外面：

```jsx
function ChatIndicator() {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  // ...
}

// ✅ 总是相同的函数，所以 React 不需要重新订阅
function subscribe() {
  // ...
}
```

或者，把 subscribe 包在 useCallback 里面以便只在某些参数改变时重新订阅：

```jsx
function ChatIndicator({ userId }) {
  const isOnline = useSyncExternalStore(subscribe, getSnapshot);
  
  // ✅ 只要 userId 不变，都是同一个函数
  const subscribe = useCallback(() => {
    // ...
  }, [userId]);

  // ...
}
```