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
如果 React 没有在父树中找到该特定 `context` 的任何 `provider`，`useContext()` 返回的 `context` 值将等于你在 创建 `context` 时指定的 默认值：

```jsx
const ThemeContext = createContext(null);
```

默认值 从不改变。如果你想要更新 `context`，请按 上述方式 将其与状态一起使用。

通常，除了 `null`，还有一些更有意义的值可以用作默认值，例如：

```jsx
const ThemeContext = createContext('light');
```

这样，如果你不小心渲染了没有相应 `provider` 的某个组件，它也不会出错。这也有助于你的组件在测试环境中很好地运行，而无需在测试中设置许多 `provider`。

在下面的例子中，`“Toggle theme”` 按钮总是处于 `light` 状态，因为它位于 任何主题的 `context provider` 之外，且 `context` 主题的默认值是 `'light'`。试着编辑默认主题为 `'dark'`。

```jsx
import { createContext, useContext, useState } from 'react';

const ThemeContext = createContext('light');

export default function MyApp() {
  const [theme, setTheme] = useState('light');
  return (
    <>
      <ThemeContext.Provider value={theme}>
        <Form />
      </ThemeContext.Provider>
      <Button onClick={() => {
        setTheme(theme === 'dark' ? 'light' : 'dark');
      }}>
        Toggle theme
      </Button>
    </>
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

function Button({ children, onClick }) {
  const theme = useContext(ThemeContext);
  const className = 'button-' + theme;
  return (
    <button className={className} onClick={onClick}>
      {children}
    </button>
  );
}
```

### 覆盖组件树一部分的 context 

通过在 `provider` 中使用不同的值包装树的某个部分，可以覆盖该部分的 `context`。

```jsx
<ThemeContext.Provider value="dark">
  ...
  <ThemeContext.Provider value="light">
    <Footer />
  </ThemeContext.Provider>
  ...
</ThemeContext.Provider>
```

你可以根据需要多次嵌套和覆盖 `provider`。

#### 第 1 个示例 共 2 个挑战: 覆盖主题 
这里，与 `Footer` 外的值为（`"dark"`）的按钮相比，里面 的按钮接收到一个不一样的 `context` 值（`"light"`）。

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
      <ThemeContext.Provider value="light">
        <Footer />
      </ThemeContext.Provider>
    </Panel>
  );
}

function Footer() {
  return (
    <footer>
      <Button>Settings</Button>
    </footer>
  );
}

function Panel({ title, children }) {
  const theme = useContext(ThemeContext);
  const className = 'panel-' + theme;
  return (
    <section className={className}>
      {title && <h1>{title}</h1>}
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

#### 第 2 个示例 共 2 个挑战: 自动嵌套标题

在嵌套使用 `context provider` 时，可以“累积”信息。在此示例中，`Section` 组件记录了 `LevelContext`，该 `context` 指定了 `section` 嵌套的深度。它从父级 `section` 中读取 `LevelContext`，然后把 `LevelContext` 的数值加一传递给子级。因此，`Heading` 组件可以根据被 `Section` 组件嵌套的层数自动决定使用 `<h1>`，`<h2>`，`<h3>`，…，中的哪种标签。

##### LevelContext.js

```jsx
import { createContext } from 'react';

export const LevelContext = createContext(0);
```

##### App.js

```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading>Title</Heading>
      <Section>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

##### Section.js

```jsx
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext.Provider value={level + 1}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

##### Heading.js

```jsx
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Heading({ children }) {
  const level = useContext(LevelContext);
  switch (level) {
    case 0:
      throw Error('Heading must be inside a Section!');
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('Unknown level: ' + level);
  }
}
```

### 在传递对象和函数时优化重新渲染
你可以通过 `context` 传递任何值，包括对象和函数。

```jsx
function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  function login(response) {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }

  return (
    <AuthContext.Provider value={{ currentUser, login }}>
      <Page />
    </AuthContext.Provider>
  );
}
```

此处，`context value` 是一个具有两个属性的 JavaScript 对象，其中一个是函数。每当 `MyApp` 出现重新渲染（例如，路由更新）时，这里将会是一个 不同的 对象指向 不同的 函数，因此 React 还必须重新渲染树中调用 `useContext(AuthContext)` 的所有组件。

在较小的应用程序中，这不是问题。但是，如果基础数据如 `currentUser` 没有更改，则不需要重新渲染它们。为了帮助 React 利用这一点，你可以使用 `useCallback` 包装 login 函数，并将对象创建包装到 `useMemo` 中。这是一个性能优化的例子：

```jsx
import { useCallback, useMemo } from 'react';

function MyApp() {
  const [currentUser, setCurrentUser] = useState(null);

  const login = useCallback((response) => {
    storeCredentials(response.credentials);
    setCurrentUser(response.user);
  }, []);

  const contextValue = useMemo(() => ({
    currentUser,
    login
  }), [currentUser, login]);

  return (
    <AuthContext.Provider value={contextValue}>
      <Page />
    </AuthContext.Provider>
  );
}
```

根据以上改变，即使 `MyApp` 需要重新渲染，调用 `useContext(AuthContext)` 的组件也不需要重新渲染，除非 `currentUser` 发生了变化。