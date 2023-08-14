# 使用 ref 操作 DOM
由于 React 会自动处理更新 DOM 以匹配你的渲染输出，因此你在组件中通常不需要操作 DOM。但是，有时你可能需要访问由 React 管理的 DOM 元素 —— 例如，让一个节点获得焦点、滚动到它或测量它的尺寸和位置。在 React 中没有内置的方法来做这些事情，所以你需要一个指向 DOM 节点的 `ref` 来实现。

## 你将会学习到
+ 如何使用 `ref` 属性访问由 React 管理的 DOM 节点
+ `ref` JSX 属性如何与 `useRef` Hook 相关联
+ 如何访问另一个组件的 DOM 节点
+ 在哪些情况下修改 React 管理的 DOM 是安全的

### 获取指向节点的 `ref` 
要访问由 React 管理的 DOM 节点，首先，引入 `useRef` Hook：

```jsx
import { useRef } from 'react';
```

然后，在你的组件中使用它声明一个 `ref`：

```jsx
const myRef = useRef(null);
```

最后，把你的 `ref` 作为 `ref` 属性传递给 JSX 标签，你想要得到它的 DOM 节点:

```jsx
<div ref={myRef}>
```

`useRef` Hook 返回一个对象，该对象有一个名为 `current` 的属性。最初，`myRef.current` 是 `null`。当 React 为这个 `<div>` 创建一个 DOM 节点时，React 会把对该节点的引用放入 `myRef.current`。然后，你可以从 事件处理器 访问此 DOM 节点，并使用在其上定义的内置浏览器 API。

```jsx
// 你可以使用任意浏览器 API，例如：
myRef.current.scrollIntoView();
```

### 示例: 使文本输入框获得焦点 
在本例中，单击按钮将使输入框获得焦点：

```jsx
import { useRef } from 'react';

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} />
      <button onClick={handleClick}>
        聚焦输入框
      </button>
    </>
  );
}
```

要实现这一点：
1. 使用 `useRef` Hook 声明 `inputRef`。
2. 像 `<input ref={inputRef}>` 这样传递它。这告诉 React 将这个 `<input>` 的 DOM 节点放入 `inputRef.current`。
3. 在 `handleClick` 函数中，从 `inputRef.current` 读取 `input` DOM 节点并使用 `inputRef.current.focus()` 调用它的 `focus(`)。
4. 用 `onClick` 将 `handleClick` 事件处理器传递给 `<button>`。

虽然 DOM 操作是 `ref` 最常见的用例，但 `useRef` Hook 可用于存储 React 之外的其他内容，例如计时器 ID 。与 `state` 类似，`ref` 能在渲染之间保留。你甚至可以将 `ref` 视为设置它们时不会触发重新渲染的 `state` 变量！你可以在使用 Ref 引用值中了解有关 `ref` 的更多信息。

### 示例: 滚动至一个元素 
一个组件中可以有多个 `ref`。在这个例子中，有一个由三张图片和三个按钮组成的轮播，点击按钮会调用浏览器的 `scrollIntoView()` 方法，在相应的 DOM 节点上将它们居中显示在视口中：

```jsx
import { useRef } from 'react';

export default function CatFriends() {
  const firstCatRef = useRef(null);
  const secondCatRef = useRef(null);
  const thirdCatRef = useRef(null);

  function handleScrollToFirstCat() {
    firstCatRef.current.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  function handleScrollToSecondCat() {
    secondCatRef.current.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  function handleScrollToThirdCat() {
    thirdCatRef.current.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  return (
    <>
      <nav>
        <button onClick={handleScrollToFirstCat}>
          Tom
        </button>
        <button onClick={handleScrollToSecondCat}>
          Maru
        </button>
        <button onClick={handleScrollToThirdCat}>
          Jellylorum
        </button>
      </nav>
      <div>
        <ul>
          <li>
            <img
              src="https://placekitten.com/g/200/200"
              alt="Tom"
              ref={firstCatRef}
            />
          </li>
          <li>
            <img
              src="https://placekitten.com/g/300/200"
              alt="Maru"
              ref={secondCatRef}
            />
          </li>
          <li>
            <img
              src="https://placekitten.com/g/250/200"
              alt="Jellylorum"
              ref={thirdCatRef}
            />
          </li>
        </ul>
      </div>
    </>
  );
}
```

### 如何使用 ref 回调管理 ref 列表 
在上面的例子中，`ref` 的数量是预先确定的。但有时候，你可能需要为列表中的每一项都绑定 `ref` ，而你又不知道会有多少项。像下面这样做是行不通的：

```jsx
<ul>
  {items.map((item) => {
    // 行不通！
    const ref = useRef(null);
    return <li ref={ref} />;
  })}
</ul>
```

这是因为 Hook 只能在组件的顶层被调用。不能在循环语句、条件语句或 `map()` 函数中调用 `useRef` 。

一种可能的解决方案是用一个 `ref` 引用其父元素，然后用 DOM 操作方法如 `querySelectorAll` 来寻找它的子节点。然而，这种方法很脆弱，如果 DOM 结构发生变化，可能会失效或报错。

另一种解决方案是将函数传递给 `ref` 属性。这称为 `ref` 回调。当需要设置 `ref` 时，React 将传入 DOM 节点来调用你的 `ref` 回调，并在需要清除它时传入 `null` 。这使你可以维护自己的数组或 `Map`，并通过其索引或某种类型的 ID 访问任何 `ref`。

此示例展示了如何使用此方法滚动到长列表中的任意节点：

```jsx
import { useRef } from 'react';

export default function CatFriends() {
  const itemsRef = useRef(null);

  function scrollToId(itemId) {
    const map = getMap();
    const node = map.get(itemId);
    node.scrollIntoView({
      behavior: 'smooth',
      block: 'nearest',
      inline: 'center'
    });
  }

  function getMap() {
    if (!itemsRef.current) {
      // 首次运行时初始化 Map。
      itemsRef.current = new Map();
    }
    return itemsRef.current;
  }

  return (
    <>
      <nav>
        <button onClick={() => scrollToId(0)}>
          Tom
        </button>
        <button onClick={() => scrollToId(5)}>
          Maru
        </button>
        <button onClick={() => scrollToId(9)}>
          Jellylorum
        </button>
      </nav>
      <div>
        <ul>
          {catList.map(cat => (
            <li
              key={cat.id}
              ref={(node) => {
                const map = getMap();
                if (node) {
                  map.set(cat.id, node);
                } else {
                  map.delete(cat.id);
                }
              }}
            >
              <img
                src={cat.imageUrl}
                alt={'Cat #' + cat.id}
              />
            </li>
          ))}
        </ul>
      </div>
    </>
  );
}

const catList = [];
for (let i = 0; i < 10; i++) {
  catList.push({
    id: i,
    imageUrl: 'https://placekitten.com/250/200?image=' + i
  });
}
```

在这个例子中，`itemsRef` 保存的不是单个 DOM 节点，而是保存了包含列表项 ID 和 DOM 节点的 `Map`。(Ref 可以保存任何值！) 每个列表项上的 `ref` 回调负责更新 Map：

```jsx
<li
  key={cat.id}
  ref={node => {
    const map = getMap();
    if (node) {
      // 添加到 Map
      map.set(cat.id, node);
    } else {
      // 从 Map 删除
      map.delete(cat.id);
    }
  }}
>
```

这使你可以之后从 Map 读取单个 DOM 节点。

### 访问另一个组件的 DOM 节点

当你将 `ref` 放在像 `<input />` 这样输出浏览器元素的内置组件上时，React 会将该 `ref` 的 `current` 属性设置为相应的 DOM 节点（例如浏览器中实际的 `<input />` ）。

但是，如果你尝试将 `ref` 放在 你自己的 组件上，例如 `<MyInput /`>，默认情况下你会得到 `null`。这个示例演示了这种情况。请注意单击按钮 并不会 聚焦输入框：

```jsx
import { useRef } from 'react';

function MyInput(props) {
  return <input {...props} />;
}

export default function MyForm() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current?.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        聚焦输入框
      </button>
    </>
  );
}
```

为了帮助您注意到这个问题，React 还会向控制台打印一条错误消息：

```
Warning: Function components cannot be given refs. Attempts to access this ref will fail. Did you mean to use React.forwardRef()?
```

发生这种情况是因为默认情况下，React 不允许组件访问其他组件的 DOM 节点。甚至自己的子组件也不行！这是故意的。Refs 是一个应急方案，应该谨慎使用。手动操作 另一个 组件的 DOM 节点会使你的代码更加脆弱。

相反，想要 暴露其 DOM 节点的组件必须选择该行为。一个组件可以指定将它的 `ref` “转发”给一个子组件。下面是 `MyInput` 如何使用 `forwardRef` API：

```jsx
const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});
```

它是这样工作的:
1. `<MyInput ref={inputRef} />` 告诉 React 将对应的 DOM 节点放入 `inputRef.current` 中。但是，这取决于 `MyInput` 组件是否允许这种行为， 默认情况下是不允许的。
2. `MyInput` 组件是使用 `forwardRef` 声明的。 这让从上面接收的 `inputRef` 作为第二个参数 `ref` 传入组件，第一个参数是 `props` 。
3. `MyInput` 组件将自己接收到的 `ref` 传递给它内部的 `<input>`。

现在，单击按钮聚焦输入框起作用了：

```jsx
import { forwardRef, useRef } from 'react';

const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        聚焦输入框
      </button>
    </>
  );
}
```

在设计系统中，将低级组件（如按钮、输入框等）的 `ref` 转发到它们的 DOM 节点是一种常见模式。另一方面，像表单、列表或页面段落这样的高级组件通常不会暴露它们的 DOM 节点，以避免对 DOM 结构的意外依赖。

### 使用命令句柄暴露一部分 API

在上面的例子中，`MyInput` 暴露了原始的 DOM 元素 `input`。这让父组件可以对其调用 `focus()`。然而，这也让父组件能够做其他事情 —— 例如，改变其 CSS 样式。在一些不常见的情况下，你可能希望限制暴露的功能。你可以用 `useImperativeHandle` 做到这一点：

```jsx
import {
  forwardRef, 
  useRef, 
  useImperativeHandle
} from 'react';

const MyInput = forwardRef((props, ref) => {
  const realInputRef = useRef(null);
  useImperativeHandle(ref, () => ({
    // 只暴露 focus，没有别的
    focus() {
      realInputRef.current.focus();
    },
  }));
  return <input {...props} ref={realInputRef} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        聚焦输入框
      </button>
    </>
  );
}
```

这里，`MyInput` 中的 `realInputRef` 保存了实际的 `input` DOM 节点。 但是，`useImperativeHandle` 指示 React 将你自己指定的对象作为父组件的 `ref` 值。 所以 `Form` 组件内的 `inputRef.current` 将只有 `focus` 方法。在这种情况下，`ref` “句柄”不是 DOM 节点，而是你在 `useImperativeHandle` 调用中创建的自定义对象。