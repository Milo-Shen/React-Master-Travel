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

### 通过 context 更新传递的数据 
通常，你会希望 `context` 随着时间的推移而改变。要更新 `context`，请将其与 `state` 结合。在父组件中声明一个状态变量，并将当前状态作为 `context value` 传递给 `provider`。

```jsx
function MyPage() {
  const [theme, setTheme] = useState('dark');
  return (
    <ThemeContext.Provider value={theme}>
      <Form />
      <Button onClick={() => {
        setTheme('light');
      }}>
       Switch to light theme
      </Button>
    </ThemeContext.Provider>
  );
}
```

现在 `provider` 中的任何一个 `Button` 都会接收到当前的 `theme` 值。如果调用 `setTheme` 来更新传递给 `provider` 的 `theme` 值，则所有 `Button` 组件都将使用新的值 `'light'` 来重新渲染。

### 更新 context 的例子

#### 第 1 个示例 共 5 个挑战: 通过 context 来更新数据
在这个示例中，`MyApp` 组件包含一个状态变量，然后该变量被传递给 `ThemeContext provider`。选中 `“Dark mode”` 复选框更新状态。更改提供的值将重新渲染使用该 `context` 的所有组件。

```jsx
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext(null);

export default function MyApp() {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext.Provider value={theme}>
      <Form />
      <label>
        <input
          type="checkbox"
          checked={theme === 'dark'}
          onChange={(e) => {
            setTheme(e.target.checked ? 'dark' : 'light')
          }}
        />
        Use dark mode
      </label>
    </ThemeContext.Provider>
  )
}

function Form({ children }) {
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

#### 第 2 个示例 共 5 个挑战: 通过 context 更新对象
在这个例子中，有一个 `currentUser` 状态变量，它包含一个对象。将 `{ currentUser, setCurrentUser }` 组合成一个对象，并通过 `context` 在 `value={}` 中向下传递。这允许下面的任何组件，如 `LoginButton`，同时读取 `currentUser` 和 `setCurrentUser`，然后在需要时调用 `setCurrentUser`。

```jsx
import { createContext, useContext, useState } from 'react';

const CurrentUserContext = createContext(null);

export default function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);
  return (
    <CurrentUserContext.Provider
      value={{
        currentUser,
        setCurrentUser
      }}
    >
      <Form />
    </CurrentUserContext.Provider>
  );
}

function Form({ children }) {
  return (
    <Panel title="Welcome">
      <LoginButton />
    </Panel>
  );
}

function LoginButton() {
  const {
    currentUser,
    setCurrentUser
  } = useContext(CurrentUserContext);

  if (currentUser !== null) {
    return <p>You logged in as {currentUser.name}.</p>;
  }

  return (
    <Button onClick={() => {
      setCurrentUser({ name: 'Advika' })
    }}>Log in as Advika</Button>
  );
}

function Panel({ title, children }) {
  return (
    <section className="panel">
      <h1>{title}</h1>
      {children}
    </section>
  )
}

function Button({ children, onClick }) {
  return (
    <button className="button" onClick={onClick}>
      {children}
    </button>
  );
}
```

### 指定回退默认值 
