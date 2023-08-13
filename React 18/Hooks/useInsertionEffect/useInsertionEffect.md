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

`useInsertionEffect` 返回 `undefined`。

+ Effects 只在客户端上运行。它们不会在服务器渲染期间运行。
+ 你不能从 `useInsertionEffect` 内部更新状态。
+ 在运行 `useInsertionEffect` 时，`refs` 还没有被附加。
+ `useInsertionEffect` 可以在 DOM 更新之前或之后运行。您不应该依赖于在任何特定时间更新的 DOM。
+ 与其他类型的 `Effects` 不同，它们先为每个 `Effect` 触发 `cleanup`，然后为每个 Effect 执行 `setup`，`useInsertionEffect` 将一次触发一个组件的 `cleanup` 和 `set`。这将导致 `cleanup` 和 `set` 函数的“交错”。
 
## 用法

### 从 CSS-in-JS 库中注入动态样式 

传统上，你会使用纯 CSS 为 React 组件设置样式。

```
// 在你的 JS 文件中：
<button className="success" />

// 在你的 CSS 文件中：
.success { color: green; }
```

有些团队更喜欢直接在 JavaScript 代码中编写样式，而不是编写 CSS 文件。这通常需要使用 CSS-in-JS 库或工具。以下是 CSS-in-JS 三种常见的实现方法：
1. 使用编译器静态提取到 CSS 文件
2. 内联样式，例如 `<div style={{ opacity: 1 }}>`
3. 运行时注入 `<style>` 标签

如果你使用 CSS-in-JS，我们建议结合使用前两种方法（静态样式使用 CSS 文件，动态样式使用内联样式）。我们不建议运行时注入 `<style>` 标签有两个原因：
1. 运行时注入会使浏览器频繁地重新计算样式。
2. 如果在 React 生命周期中某个错误的时机进行运行时注入，它可能会非常慢。

第一个问题无法解决，但是 `useInsertionEffect` 可以帮助你解决第二个问题。

调用 `useInsertionEffect` 在任何 layout effects 触发之前插入样式:

```jsx
// 在你的 CSS-in-JS 库中
let isInserted = new Set();
function useCSS(rule) {
  useInsertionEffect(() => {
    // 同前所述，我们不建议在运行时注入 <style> 标签。
    // 如果你必须这样做，那么应当在 useInsertionEffect 中进行。
    if (!isInserted.has(rule)) {
      isInserted.add(rule);
      document.head.appendChild(getStyleForRule(rule));
    }
  });
  return rule;
}

function Button() {
  const className = useCSS('...');
  return <div className={className} />;
}
```

与 `useEffect` 类似，`useInsertionEffect` 不在服务端运行。如果你需要收集在服务端上使用了哪些 CSS 规则，你可以在渲染期间进行：

```jsx
let collectedRulesSet = new Set();

function useCSS(rule) {
  if (typeof window === 'undefined') {
    collectedRulesSet.add(rule);
  }
  useInsertionEffect(() => {
    // ...
  });
  return rule;
}
```

[阅读更多使用 useInsertionEffect 升级 CSS-in-JS 库的相关指南。](https://github.com/reactwg/react-18/discussions/110)

### 这与在渲染期间或 `useLayoutEffect` 中注入样式相比有何优势？ 

如果你在渲染期间注入样式并且 React 正在处理 非阻塞更新，那么浏览器将在渲染组件树时每一帧都会重新计算样式，这可能会 非常慢。

`useInsertionEffect` 比在 `useLayoutEffect` 或 `useEffect` 期间注入样式更好。因为它会确保 `<style>` 标签在其它 `Effect` 运行前被注入。否则，正常的 `Effect` 中的布局计算将由于过时的样式而出错。