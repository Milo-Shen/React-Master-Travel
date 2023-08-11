# useTransition

`useTransition` 是一个让你在不阻塞 UI 的情况下来更新状态的 React Hook。

```jsx
const [isPending, startTransition] = useTransition();
```

## 参考 

```jsx
useTransition();
```

在组件的顶层调用 `useTransition`，将某些状态更新标记为转换状态。

```jsx
import { useTransition } from 'react';

function TabContainer() {
  const [isPending, startTransition] = useTransition();
  // ...
}
```

### 参数 
`useTransition` 不需要任何参数。

### 返回值 
`useTransition` 返回一个由两个元素组成的数组：

1. `isPending` 标志，告诉你是否存在待处理的转换。
2. `startTransition` 函数 允许你将状态更新标记为转换状态。

### startTransition 函数 
`useTransition` 返回的 `startTransition` 函数允许你将状态更新标记为转换状态。

```jsx
function TabContainer() {
  const [isPending, startTransition] = useTransition();
  const [tab, setTab] = useState('about');

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);
    });
  }
  // ...
}
```

### 参数 
作用域（`scope`）：一个通过调用一个或多个 `set` 函数。 函数更新某些状态的函数。`React` 会立即不带参数地调用此函数，并将在作用域函数调用期间计划同步执行的所有状态更新标记为转换状态。它们将是非阻塞的，并且 不会显示不想要的加载指示器。

### 返回值 
`startTransition` 不会返回任何值。

### 注意事项 
+ `useTransition` 是一个 Hook，因此只能在组件或自定义 Hook 内部调用。如果你需要在其他地方启动转换（例如从数据库），请调用独立的 `startTransition` 函数。
+ 只有在你可以访问该状态的 `set` 函数时，才能将更新包装为转换状态。如果你想响应某个 `prop` 或自定义 Hook 值启动转换，请尝试使用 `useDeferredValue`。
+ 你传递给 `startTransition` 的函数必须是同步的。React 立即执行此函数，标记其执行期间发生的所有状态更新为转换状态。如果你稍后尝试执行更多的状态更新（例如在一个定时器中），它们将不会被标记为转换状态。
+ 标记为转换状态的状态更新将被其他状态更新打断。例如，如果你在转换状态中更新图表组件，但在图表正在重新渲染时开始在输入框中输入，React 将在处理输入更新后重新启动对图表组件的渲染工作。
+ 转换状态更新不能用于控制文本输入。
+ 如果有多个正在进行的转换状态，React 目前会将它们批处理在一起。这是一个限制，可能会在未来的版本中被删除。