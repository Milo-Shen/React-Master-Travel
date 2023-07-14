# useSyncExternalStore
`useSyncExternalStore` 是一个可以让开发者订阅外部 store 的 React Hook
`const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)`

## Reference
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

参数:
1. `subscribe`：一个函数，接收一个单独的 callback 参数并把它订阅到 store 上。当 store 发生改变，它应当调用被提供的 callback。这会导致组件重新渲染。subscribe 函数会返回清除订阅的函数。
2. `getSnapshot`：一个函数，返回组件需要的 store 中的数据快照。在 store 不变的情况下，重复调用 getSnapshot 必须返回同一个值。如果 store 改变，并且返回值也不同了（ 用 `Object.is` 比较 ），React 就会重新渲染组件。
3. 可选 `getServerSnapshot`：一个函数，返回 store 中数据的初始快照。它只会在服务端渲染时，以及在客户端进行服务端渲染内容的 hydration 时被用到。快照在服务端与客户端之间必须相同，它通常是从服务端序列化并传到客户端的。如果你忽略此参数，在服务端渲染这个组件会抛出一个错误。

返回值: 
该 store 的当前快照，可以在你的渲染逻辑中使用。

警告:
1. `getSnapshot` 返回的 store 快照必须是不可变的。如果底层 store 有可变数据，要在数据改变时返回一个新的不可变快照。否则，返回上次缓存的快照。
2. 如果在重新渲染时传入一个不同的 subscribe 函数，React 会用新传入的 subscribe 函数重新订阅该 store。你可以通过在组件外声明 subscribe 来避免。
