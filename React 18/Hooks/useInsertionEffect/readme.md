# useInsertionEffect

## 陷阱
`useInsertionEffect` 是为 `CSS-in-JS` 库的作者特意打造的。除非你正在使用 `CSS-in-JS` 库并且需要注入样式，否则你应该使用 `useEffect` 或者 `useLayoutEffect`。

`useInsertionEffect` 可以在布局副作用触发之前将元素插入到 DOM 中。

```jsx
useInsertionEffect(setup, dependencies?)
```

## 参考 

```jsx
useInsertionEffect(setup, dependencies?)
```

调用 `useInsertionEffect` 在任何可能需要读取布局的副作用启动之前插入样式：

```jsx
import { useInsertionEffect } from 'react';

// 在你的 CSS-in-JS 库中
function useCSS(rule) {
  useInsertionEffect(() => {
    // ... 在此注入 <style> 标签 ...
  });
  return rule;
}
```

### 参数 
+ `setup`: 函数与你的 Effect 的逻辑。你的 `setup` 函数也可以选择返回一个 `cleanup` 函数。当你的组件被添加到DOM中时，在任何布局效果生效之前，React将运行你的 `setup` 函数。每次使用更改的依赖项重新渲染后，React将首先使用旧值运行清理函数(如果您提供了它)，然后使用新值运行 setup 函数。当你的组件从 DOM 中移除时，React会运行你的 `cleanup` 函数。
+ 可选 `dependencies`：`setup` 代码中引用的所有响应式值的列表。响应式值包括 `props`、`state` 以及所有直接在组件内部声明的变量和函数。如果你的代码检查工具 配置了 React，那么它将验证是否每个响应式值都被正确地指定为依赖项。依赖列表必须具有固定数量的项，并且必须像 `[dep1, dep2, dep3]` 这样内联编写。React 将使用 `Object.is` 来比较每个依赖项和它先前的值。如果省略此参数，则将在每次重新渲染组件之后重新运行 `Effect`。

### 返回值 

useInsertionEffect 返回 undefined。

+ Effects 只在客户端上运行。它们不会在服务器渲染期间运行。
+ 你不能从 `useInsertionEffect` 内部更新状态。
+ 在运行 `useInsertionEffect` 时，`refs` 还没有被附加。
+ `useInsertionEffect` 可以在 DOM 更新之前或之后运行。您不应该依赖于在任何特定时间更新的 DOM。
+ 与其他类型的 `Effects` 不同，它们先为每个 `Effect` 触发 `cleanup`，然后为每个 Effect 执行 `setup`，`useInsertionEffect` 将一次触发一个组件的 `cleanup` 和 `set`。这将导致 `cleanup` 和 `set` 函数的“交错”。
 