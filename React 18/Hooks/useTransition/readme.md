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

## 用法

### 将状态更新标记为非阻塞转换状态
在组件的顶层调用 `useTransition`，将状态更新标记为非阻塞的转换状态。

```jsx
import { useState, useTransition } from 'react';

function TabContainer() {
  const [isPending, startTransition] = useTransition();
  // ...
}
```

`useTransition` 返回一个具有两个项的数组：

1. `isPending` 标志，告诉你是否存在挂起的转换状态。
2. `startTransition` 方法 允许你将状态更新标记为转换状态。

你可以按照以下方式将状态更新标记为转换状态：

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

转换状态可以使用户界面更新在慢速设备上仍保持响应性。

通过转换状态，在重新渲染过程中你的用户界面保持响应。例如，如果用户单击一个选项卡，但改变了主意并单击另一个选项卡，他们可以在不等待第一个重新渲染完成的情况下完成操作。

##  `useTransition` 和常规 `state` update 的区别

### 第 1 个示例 共 2 个挑战: 在转换状态中更新当前选项卡 

在此示例中，“文章”选项卡被 *人为地减慢*，以便至少需要一秒钟才能呈现。

单击“文章”，然后立即单击“联系人”。请注意，这会中断“文章”的缓慢渲染。 “联系人”选项卡立即显示。因为此状态更新被标记为转换状态，因此缓慢的重新渲染不会冻结用户界面。

#### App.js
```jsx
import { useState, useTransition } from 'react';
import TabButton from './TabButton.js';
import AboutTab from './AboutTab.js';
import PostsTab from './PostsTab.js';
import ContactTab from './ContactTab.js';

export default function TabContainer() {
  const [isPending, startTransition] = useTransition();
  const [tab, setTab] = useState('about');

  function selectTab(nextTab) {
    startTransition(() => {
      setTab(nextTab);      
    });
  }

  return (
    <>
      <TabButton
        isActive={tab === 'about'}
        onClick={() => selectTab('about')}
      >
        About
      </TabButton>
      <TabButton
        isActive={tab === 'posts'}
        onClick={() => selectTab('posts')}
      >
        Posts (slow)
      </TabButton>
      <TabButton
        isActive={tab === 'contact'}
        onClick={() => selectTab('contact')}
      >
        Contact
      </TabButton>
      <hr />
      {tab === 'about' && <AboutTab />}
      {tab === 'posts' && <PostsTab />}
      {tab === 'contact' && <ContactTab />}
    </>
  );
}
```

#### TabButton.js
```jsx
import { useTransition } from 'react';

export default function TabButton({ children, isActive, onClick }) {
  if (isActive) {
    return <b>{children}</b>
  }
  return (
    <button onClick={() => {
      onClick();
    }}>
      {children}
    </button>
  )
}
```

#### AboutTab.js
```jsx
export default function AboutTab() {
    return (
        <p>Welcome to my profile!</p>
    );
}
```

#### PostsTab.js
```jsx
import { memo } from 'react';

const PostsTab = memo(function PostsTab() {
  // Log once. The actual slowdown is inside SlowPost.
  console.log('[ARTIFICIALLY SLOW] Rendering 500 <SlowPost />');

  let items = [];
  for (let i = 0; i < 500; i++) {
    items.push(<SlowPost key={i} index={i} />);
  }
  return (
    <ul className="items">
      {items}
    </ul>
  );
});

function SlowPost({ index }) {
  let startTime = performance.now();
  while (performance.now() - startTime < 1) {
    // Do nothing for 1 ms per item to emulate extremely slow code
  }

  return (
    <li className="item">
      Post #{index + 1}
    </li>
  );
}

export default PostsTab;
```

#### ContactTab.js
```jsx
export default function ContactTab() {
  return (
    <>
      <p>
        You can find me online here:
      </p>
      <ul>
        <li>admin@mysite.com</li>
        <li>+123456789</li>
      </ul>
    </>
  );
}
```

### 第 2 个示例 共 2 个挑战: 在不使用转换状态的情况下更新当前选项卡

在此示例中，“帖子”选项卡同样被 *人为地减慢*，以便至少需要一秒钟才能渲染。与之前的示例不同，这个状态更新 *不是一个转换状态*。

点击“帖子”，然后立即点击“联系人”。请注意，应用程序在渲染减速选项卡时会冻结，UI变得不响应。这个状态更新 *不是一个转换状态*，所以慢速的重新渲染会冻结用户界面。

```jsx
import { useState } from 'react';
import TabButton from './TabButton.js';
import AboutTab from './AboutTab.js';
import PostsTab from './PostsTab.js';
import ContactTab from './ContactTab.js';

export default function TabContainer() {
  const [tab, setTab] = useState('about');

  function selectTab(nextTab) {
    setTab(nextTab);
  }

  return (
    <>
      <TabButton
        isActive={tab === 'about'}
        onClick={() => selectTab('about')}
      >
        About
      </TabButton>
      <TabButton
        isActive={tab === 'posts'}
        onClick={() => selectTab('posts')}
      >
        Posts (slow)
      </TabButton>
      <TabButton
        isActive={tab === 'contact'}
        onClick={() => selectTab('contact')}
      >
        Contact
      </TabButton>
      <hr />
      {tab === 'about' && <AboutTab />}
      {tab === 'posts' && <PostsTab />}
      {tab === 'contact' && <ContactTab />}
    </>
  );
}
```