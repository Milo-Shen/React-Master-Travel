# Suspense

`<Suspense>` 允许你显示一个退路方案（fallback）直到它的子组件完成加载。

## 参考 

```jsx
<Suspense>
```

### 参数 
+ `children`：实际的 UI 渲染内容。如果 `children` 在渲染中挂起（`suspend`），`Suspense` 边界将切换到渲染 `fallback`。
+ `fallback`：一个在实际的 UI 未渲染完成时代替其渲染的备用 UI。任何有效的 React Node 都被接受，但实际上退路方案（`fallback`）是一个轻量的占位符，例如加载中图标或者骨架屏。`Suspense` 将自动切换到 `fallback` 当 `children` 挂起（`suspend`）时，并在数据就位时切换回 `children`。如果 `fallback` 在渲染中挂起（`suspend`），它将自动激活最近的 `Suspense` 边界。