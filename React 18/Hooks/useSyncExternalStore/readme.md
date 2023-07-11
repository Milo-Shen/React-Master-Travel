# useSyncExternalStore
`useSyncExternalStore` 是一个可以让开发者订阅外部存储的钩子
`const snapshot = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)`

## Reference
### `useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?)`

调用位于组件最顶上的 useSyncExternalStore 从外部数据存储读取你想要的值

```jsx
import { useSyncExternalStore } from 'react';
import { todosStore } from './todoStore.js';

function TodosApp() {
  const todos = useSyncExternalStore(todosStore.subscribe, todosStore.getSnapshot);
  // ...
}
```