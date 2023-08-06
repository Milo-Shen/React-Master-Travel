# useRef

`useRef` 是一个 React Hook，它能让你引用一个不需要渲染的值。

```
const ref = useRef(initialValue)
```

## 参考 

```jsx
useRef(initialValue);
```

在你组件的顶层调用 `useRef` 声明一个 `ref`。

```jsx
import { useRef } from 'react';

function MyComponent() {
    const intervalRef = useRef(0);
    const inputRef = useRef(null);
    // ...
}
```

### 参数 
+ `initialValue`：`ref` 对象的 `current` 属性的初始值。可以是任意类型的值。这个参数会首次渲染后被忽略。

### 返回值 

`useRef` 返回一个只有一个属性的对象:

+ `current`：最初，它被设置为你传递的 `initialValue`。之后你可以把它设置为其他值。如果你把 `ref` 对象作为一个 JSX 节点的 `ref` 属性传递给 React，React 将为它设置 `current` 属性。

在后续的渲染中，`useRef` 将返回同一个对象。

### 注意事项 

+ 你可以修改 `ref.current` 属性。与 `state` 不同，它是可变的。然而，如果它持有一个用于渲染的对象（例如，你的 `state` 的一部分），那么你就不应该修改这个对象。
+ 当你改变 `ref.current` 属性时，React 不会重新渲染你的组件。React 不知道你何时改变它，因为 `ref` 是一个普通的 JavaScript 对象。