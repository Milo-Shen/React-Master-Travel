# 在组件间共享状态
有时候，你希望两个组件的状态始终同步更改。要实现这一点，可以将相关 state 从这两个组件上移除，并把 state 放到它们的公共父级，再通过 props 将 state 传递给这两个组件。这被称为“状态提升”，这是编写 React 代码时常做的事。

## 你将会学习到
+ 如何使用状态提升在组件之间共享状态
+ 什么是受控组件和非受控组件

## 举例说明一下状态提升 
在这个例子中，父组件 `Accordion` 渲染了 2 个独立的 `Panel` 组件。

+ Accordion
  + Panel
  + Panel

每个 `Panel` 组件都有一个布尔值 `isActive`，用于控制其内容是否可见。

请点击 2 个面板中的显示按钮：

```jsx
import { useState } from 'react';

function Panel({ title, children }) {
  const [isActive, setIsActive] = useState(false);
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>
          显示
        </button>
      )}
    </section>
  );
}

export default function Accordion() {
  return (
    <>
      <h2>哈萨克斯坦，阿拉木图</h2>
      <Panel title="关于">
        阿拉木图人口约200万，是哈萨克斯坦最大的城市。它在 1929 年到 1997 年间都是首都。
      </Panel>
      <Panel title="词源">
        这个名字来自于 <span lang="kk-KZ">алма</span>，哈萨克语中“苹果”的意思，经常被翻译成“苹果之乡”。事实上，阿拉木图的周边地区被认为是苹果的发源地，<i lang="la">Malus sieversii</i> 被认为是现今苹果的祖先。
      </Panel>
    </>
  );
}
```

我们发现点击其中一个面板中的按钮并不会影响另外一个，他们是独立的。

*假设现在您想改变这种行为，以便在任何时候只展开一个面板。* 在这种设计下，展开第 2 个面板应会折叠第 1 个面板。您该如何做到这一点呢？”

要协调好这两个面板，我们需要分 3 步将状态“提升”到他们的父组件中。

1. 从子组件中 移除 state 。
2. 从父组件 传递 硬编码数据。
3. 为共同的父组件添加 state ，并将其与事件处理函数一起向下传递。

这样，`Accordion` 组件就可以控制 2 个 `Panel` 组件，保证同一时间只能展开一个。

### 第 1 步: 从子组件中移除状态 
你将把 `Panel` 组件对 `isActive` 的控制权交给他们的父组件。这意味着，父组件会将 `isActive` 作为 prop 传给子组件 `Panel`。我们先从 `Panel` 组件中 删除下面这一行：

```jsx
const [isActive, setIsActive] = useState(false);
```

然后，把 `isActive` 加入 `Panel` 组件的 `props` 中：

```jsx
function Panel({ title, children, isActive }) {
}
```

现在 `Panel` 的父组件就可以通过 向下传递 prop 来 控制 `isActive`。但相反地，`Panel` 组件对 `isActive` 的值 没有控制权 —— 现在完全由父组件决定！

