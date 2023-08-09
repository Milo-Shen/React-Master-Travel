# StrictMode

```jsx
<StrictMode> 帮助你在开发过程中尽早地发现组件中的常见错误。
```

```jsx
<StrictMode>
  <App />
</StrictMode>
```

## 参考 

使用 `StrictMode` 来启用组件树内部的额外开发行为和警告：

```jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

严格模式启用了以下仅在开发环境下有效的行为：
1. 组件将 重新渲染一次，以查找由于非纯渲染而引起的错误。
2. 组件将 重新运行 Effect 一次，以查找由于缺少 Effect 清理而引起的错误。
3. 组件将被 检查是否使用了已弃用的 API。

### 参数
`StrictMode` 不接受任何参数。

### 注意事项
+ 在由 `<StrictMode>` 包裹的树中，无法选择退出严格模式。这可以确保在 `<StrictMode>` 内的所有组件都经过检查。如果两个团队在一个产品上工作，并且对于这些检查是否有价值存在分歧，他们需要达成共识或将 `<StrictMode>` 下移到树的较低层级。

## 用法

### 为整个应用启用严格模式
严格模式为 `<StrictMode>` 组件内的整个组件树启用额外的开发环境检查，这些检查有助于在开发过程中尽早地发现组件中的常见错误。
如果要为整个应用启用严格模式，请在渲染根组件时使用 `<StrictMode>` 包裹它：

```jsx
import { StrictMode } from 'react';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'));
root.render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

我们建议将整个应用程序包裹在严格模式中，特别是对于新创建的应用程序。如果你使用的是一个调用 `createRoot` 的框架，请查阅其文档以了解如何启用严格模式。

尽管严格模式的检查 *仅在开发环境* 下运行，但它们有助于找出已经存在于代码中但在生产环境中可能难以复现的错误。严格模式让你在用户反馈之前就可以修复这些错误。

## 严格模式启用了以下仅在开发环境下有效的行为：
1. 组件将 重新渲染一次，以查找由于非纯渲染而引起的错误。
2. 组件将 重新运行 Effect 一次，以查找由于缺少 Effect 清理而引起的错误。
3. 组件将被 检查是否使用了已弃用的 API。

*所有这些检查仅在开发环境中进行，不会影响生产构建。*

### 为应用的一部分启用严格模式 