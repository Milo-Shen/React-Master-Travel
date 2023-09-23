# flushSync

## 陷阱
使用 `flushSync` 是不常见的行为，并且可能损伤应用程序的性能。

`flushSync` 允许你强制 React 在提供的回调函数内同步刷新任何更新，这将确保 DOM 立即更新。

## 参考 
```jsx
flushSync(callback) 
```

调用 `flushSync` 强制 React 刷新所有挂起的工作，并同步更新 DOM。

```jsx
import { flushSync } from 'react-dom';

flushSync(() => {
  setSomething(123);
});
```

大多数时候都不需要使用 flushSync，请将其作为最后的手段使用。

### 参数 