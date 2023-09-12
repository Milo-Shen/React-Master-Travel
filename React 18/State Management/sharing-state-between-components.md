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

### 第 2 步: 从公共父组件传递硬编码数据 
为了实现状态提升，必须定位到你想协调的 两个 子组件最近的公共父组件：

+ Accordion (最近的公共父组件)
  + Panel
  + Panel

在这个例子中，公共父组件是 `Accordion`。因为它位于两个面板之上，可以控制它们的 props，所以它将成为当前激活面板的“控制之源”。通过 `Accordion` 组件将硬编码值 `isActive`（例如 `true` ）传递给两个面板：

```jsx
import { useState } from 'react';

export default function Accordion() {
  return (
    <>
      <h2>哈萨克斯坦，阿拉木图</h2>
      <Panel title="关于" isActive={true}>
        阿拉木图人口约200万，是哈萨克斯坦最大的城市。它在 1929 年到 1997 年间都是首都。
      </Panel>
      <Panel title="词源" isActive={true}>
        这个名字来自于 <span lang="kk-KZ">алма</span>，哈萨克语中“苹果”的意思，经常被翻译成“苹果之乡”。事实上，阿拉木图的周边地区被认为是苹果的发源地，<i lang="la">Malus sieversii</i> 被认为是现今苹果的祖先。
      </Panel>
    </>
  );
}

function Panel({ title, children, isActive }) {
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
```

你可以尝试修改 `Accordion` 组件中 `isActive` 的值，并在屏幕上查看结果。

### 第 3 步: 为公共父组件添加状态 
状态提升通常会改变原状态的数据存储类型。

在这个例子中，一次只能激活一个面板。这意味着 `Accordion` 这个父组件需要记录 哪个 面板是被激活的面板。我们可以用数字作为当前被激活 `Panel` 的索引，而不是 `boolean` 值：

```jsx
const [activeIndex, setActiveIndex] = useState(0);
```

当 `activeIndex` 为 `0` 时，激活第一个面板，为 `1` 时，激活第二个面板。

在任意一个 `Panel` 中点击“显示”按钮都需要更改 `Accordion` 中的激活索引值。 `Panel` 中无法直接设置状态 `activeIndex` 的值，因为该状态是在 `Accordion` 组件内部定义的。 `Accordion` 组件需要 显式允许 `Panel` 组件通过 将事件处理程序作为 prop 向下传递 来更改其状态：

```jsx
<>
  <Panel
    isActive={activeIndex === 0}
    onShow={() => setActiveIndex(0)}
  >
    ...
  </Panel>
  <Panel
    isActive={activeIndex === 1}
    onShow={() => setActiveIndex(1)}
  >
    ...
  </Panel>
</>
```

现在 `Panel` 组件中的` <button>` 将使用 `onShow` 这个属性作为其点击事件的处理程序：

```jsx
import { useState } from 'react';

export default function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);
  return (
    <>
      <h2>哈萨克斯坦，阿拉木图</h2>
      <Panel
        title="关于"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        阿拉木图人口约200万，是哈萨克斯坦最大的城市。它在 1929 年到 1997 年间都是首都。
      </Panel>
      <Panel
        title="词源"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        这个名字来自于 <span lang="kk-KZ">алма</span>，哈萨克语中“苹果”的意思，经常被翻译成“苹果之乡”。事实上，阿拉木图的周边地区被认为是苹果的发源地，<i lang="la">Malus sieversii</i> 被认为是现今苹果的祖先。
      </Panel>
    </>
  );
}

function Panel({
  title,
  children,
  isActive,
  onShow
}) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}>
          显示
        </button>
      )}
    </section>
  );
}
```

这样，我们就完成了对状态的提升！将状态移至公共父组件中可以让你更好的管理这两个面板。使用激活索引值代替之前的 是否显示 标识确保了一次只能激活一个面板。而通过向下传递事件处理函数可以让子组件修改父组件的状态。