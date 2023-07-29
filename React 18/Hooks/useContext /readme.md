# useContext

`useContext` 是一个 React Hook，可以让你读取和订阅组件中的 `context`。

```jsx
const value = useContext(SomeContext)
```

## 参考 

```jsx
useContext(SomeContext)
```

```jsx
import { useContext } from 'react';

function MyComponent() {
    const theme = useContext(ThemeContext);
    // ...
}
```

### 参数
+ `SomeContext`：先前用 `createContext` 创建的 `context`。`context` 本身不包含信息，它只代表你可以提供或从组件中读取的信息类型。

### 返回值 
`useContext` 为调用组件返回 `context` 的值。它被确定为传递给树中调用组件上方最近的 `SomeContext.Provider` 的 `value`。如果没有这样的 `provider`，那么返回值将会是为创建该 `context` 传递给 `createContext` 的 `defaultValue`。返回的值始终是最新的。如果 `context` 发生变化，React 会自动重新渲染读取 `context` 的组件。

### 注意事项 
+ 组件中的 `useContext()` 调用不受 *同一* 组件返回的 `provider` 的影响。相应的 `<Context.Provider>` 需要位于调用 `useContext()` 的组件 之上。
+ 从 `provider` 接收到不同的 `value` 开始，React 自动重新渲染使用了该特定 `context` 的所有子级。先前的值和新的值会使用 `Object.is` 来做比较。使用 `memo` 来跳过重新渲染并不妨碍子级接收到新的 `context` 值。
+ 如果您的构建系统在输出中产生重复的模块（可能发生在符号链接中），这可能会破坏 `context`。通过 `context` 传递数据只有在用于传递 `context` 的 `SomeContext` 和用于读取数据的 `SomeContext` 是完全相同的对象时才有效，这是由 `===` 比较决定的。

## 用法

### 向组件树深层传递数据 
在组件的最顶级调用 useContext 来读取和订阅 context。

```jsx
import { useContext } from 'react';

function Button() {
    const theme = useContext(ThemeContext);
    // ...
}
```

`useContext` 返回你向 `context` 传递的 `context value`。为了确定 `context` 值，React 搜索组件树，为这个特定的 `context` 向上查找最近的 `context provider`。

若要将 `context` 传递给 `Button`，请将其或其父组件之一包装到相应的 `context provider`：

```jsx
function MyPage() {
  return (
    <ThemeContext.Provider value="dark">
      <Form />
    </ThemeContext.Provider>
  );
}

function Form() {
  // ... 在内部渲染 buttons ...
}
```

`provider` 和 `Button` 之间有多少层组件并不重要。当 `Form` 中的任何位置的 `Button` 调用 `useContext(ThemeContext)` 时，它都将接收 `"dark"` 作为值。

### 陷阱
`useContext()` 总是在调用它的组件 上面 寻找最近的 `provider`。它向上搜索， 不考虑 调用 `useContext()` 的组件中的 `provider`。

```jsx
import { createContext, useContext } from 'react';

const ThemeContext = createContext(null);

export default function MyApp() {
  return (
    <ThemeContext.Provider value="dark">
      <Form />
    </ThemeContext.Provider>
  )
}

function Form() {
  return (
    <Panel title="Welcome">
      <Button>Sign up</Button>
      <Button>Log in</Button>
    </Panel>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ children }) {
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button className={className}>
      {children}
    </button>
  );
}
```

