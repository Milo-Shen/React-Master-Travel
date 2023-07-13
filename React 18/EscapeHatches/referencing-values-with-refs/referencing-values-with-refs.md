# 使用 ref 引用值

当你希望组件“记住”某些信息，但又不想让这些信息 触发新的渲染 时，你可以使用 `ref` 。

## 你将会学习到
1. 如何向组件添加 `ref`
2. 如何更新 `ref` 的值
3. `ref` 与 `state` 有何不同
4. 如何安全地使用 `ref`

### 给你的组件添加 `ref` 

你可以通过从 React 导入 `useRef` Hook 来为你的组件添加一个 `ref`：

```jsx
import { useRef } from 'react';
```

在你的组件内，调用 `useRef` Hook 并传入你想要引用的初始值作为唯一参数。例如，这里的 `ref` 引用的值是 “0”：

```jsx
const ref = useRef(0);
```

`useRef` 返回一个这样的对象:

```jsx
{ 
  current: 0 // 你向 useRef 传入的值
}
```

你可以用 `ref.current` 属性访问该 `ref` 的当前值。这个值是有意被设置为可变的，意味着你既可以读取它也可以写入它。就像一个 React 追踪不到的、用来存储组件信息的秘密“口袋”。（这就是让它成为 React 单向数据流的“应急方案”的原因 —— 详见下文！）

这里，每次点击按钮时会使 `ref.current` 递增：

```jsx
import { useRef } from 'react';

export default function Counter() {
  let ref = useRef(0);

  function handleClick() {
    ref.current = ref.current + 1;
    alert('你点击了 ' + ref.current + ' 次！');
  }

  return (
    <button onClick={handleClick}>
      点击我！
    </button>
  );
}
```

这里的 `ref` 指向一个数字，但是，像 `state` 一样，你可以让它指向任何东西：字符串、对象，甚至是函数。与 `state` 不同的是，`ref` 是一个普通的 JavaScript 对象，具有可以被读取和修改的 `current` 属性。

请注意，组件不会在每次递增时重新渲染。 与 `state` 一样，React 会在每次重新渲染之间保留 `ref`。但是，设置 `state` 会重新渲染组件，更改 `ref` 不会！