# useId

`useId` 是一个 React Hook，可以生成传递给无障碍属性的唯一 ID。

```jsx
const id = useId()
```

## 参数 
`useId` 不带任何参数。

## 返回值 
`useId` 返回一个唯一的字符串 `ID`，与此特定组件中的 `useId` 调用相关联。

## 注意事项 
+ `useId` 是一个 Hook，因此你只能 在组件的顶层 或自己的 Hook 中调用它。你不能在内部循环或条件判断中调用它。如果需要，可以提取一个新组件并将 `state` 移到该组件中。
+ `useId` 不应该被用来生成列表中的 key。key 应该由你的数据生成。

## 用法
### 为无障碍属性生成唯一 ID 

```jsx
import { useId } from 'react';

function PasswordField() {
    const passwordHintId = useId();
    return (
        <>
            <label>
                密码:
                <input
                    type="password"
                    aria-describedby={passwordHintId}
                />
            </label>
            <p id={passwordHintId}>
                密码应该包含至少 18 个字符
            </p>
        </>
    );
}

export default function App() {
    return (
        <>
            <h2>输入密码</h2>
            <PasswordField />
            <h2>验证密码</h2>
            <PasswordField />
        </>
    );
}
```

### 陷阱
使用服务端渲染时，useId 需要在服务器和客户端上有相同的组件树。如果你在服务器和客户端上渲染的树不完全匹配，生成的 ID 将不匹配。

### 为什么 useId 比递增计数器更好？ 

你可能想知道为什么使用 `useId` 比增加全局变量（如 `nextId++`）更好。

`useId` 的主要好处是 React 确保它能够与 服务端渲染一起工作。 在服务器渲染期间，你的组件生成输出 HTML。随后，在客户端，hydration 会将你的事件处理程序附加到生成的 HTML 上。由于 hydration，客户端必须匹配服务器输出的 HTML。

使用递增计数器非常难以保证这一点，因为客户端组件被 hydrate 处理后的顺序可能与服务器 HTML 发出的顺序不匹配。通过调用 `useId`，你可以确保 hydration 正常工作，并且服务器和客户端之间的输出将匹配。

在 React 内部，调用组件的“父路径”生成 `useId`。这就是为什么如果客户端和服务器的树相同，不管渲染顺序如何，“父路径”始终都匹配。

### 为所有生成的 ID 指定共享前缀 

#### index.html

```html
<!DOCTYPE html>
<html>
  <head><title>My app</title></head>
  <body>
    <div id="root1"></div>
    <div id="root2"></div>
  </body>
</html>
```

#### App.js

```jsx
import { useId } from 'react';

function PasswordField() {
  const passwordHintId = useId();
  console.log('生成的 ID：', passwordHintId)
  return (
    <>
      <label>
        密码:
        <input
          type="password"
          aria-describedby={passwordHintId}
        />
      </label>
      <p id={passwordHintId}>
        密码应该包含至少 18 个字符
      </p>
    </>
  );
}

export default function App() {
  return (
    <>
      <h2>输入密码</h2>
      <PasswordField />
    </>
  );
}
```

#### index.js

```jsx
import { createRoot } from 'react-dom/client';
import App from './App.js';
import './styles.css';

const root1 = createRoot(document.getElementById('root1'), {
  identifierPrefix: 'my-first-app-'
});
root1.render(<App />);

const root2 = createRoot(document.getElementById('root2'), {
  identifierPrefix: 'my-second-app-'
});
root2.render(<App />);
```