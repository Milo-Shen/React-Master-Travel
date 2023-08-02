# useReducer

`useReducer` 是一个 React Hook，它允许你向组件里面添加一个 `reducer`。

## 参考 

```jsx
useReducer(reducer, initialArg, init?);
```

在组件的顶层作用域调用 `useReducer` 以创建一个用于管理状态的 `reducer`。

```jsx
import { useReducer } from 'react';

function reducer(state, action) {
  // ...
}

function MyComponent() {
    const [state, dispatch] = useReducer(reducer, {age: 42});
    // ...
}
```

### 参数 
+ `reducer`：用于更新 `state` 的纯函数。参数为 `state` 和 `action`，返回值是更新后的 `state`。`state` 与 `action` 可以是任意合法值。
+ `initialArg`：用于初始化 `state` 的任意值。初始值的计算逻辑取决于接下来的 `init` 参数。
+ 可选参数 `init`：用于计算初始值的函数。如果存在，使用 `init(initialArg)` 的执行结果作为初始值，否则使用 `initialArg`。

### 返回值 
`useReducer` 返回一个由两个值组成的数组：

1. 当前的 `state`。初次渲染时，它是 `init(initialArg`) 或 `initialArg` （如果没有 init 函数）。
2. `dispatch` 函数。用于更新 `state` 并触发组件的重新渲染。

### 注意事项 
`useReducer` 是一个 Hook，所以只能在 组件的顶层作用域 或自定义 Hook 中调用，而不能在循环或条件语句中调用。如果你有这种需求，可以创建一个新的组件，并把 `state` 移入其中。
严格模式下 React 会 调用两次 `reducer` 和初始化函数，这可以 帮助你发现意外的副作用。这只是开发模式下的行为，并不会影响生产环境。只要 `reducer` 和初始化函数是纯函数（理应如此）就不会改变你的逻辑。其中一个调用结果会被忽略。

## `dispatch` 函数 

`useReducer` 返回的 `dispatch` 函数允许你更新 `state` 并触发组件的重新渲染。它需要传入一个 `action` 作为参数：

```jsx
const [state, dispatch] = useReducer(reducer, { age: 42 });

function handleClick() {
    dispatch({type: 'incremented_age'});
    // ...
}
```

React 会调用 `reducer` 函数以更新 state，reducer 函数的参数为当前的 state 与传递的 action。

### 参数 
+ `action`：用户执行的操作。可以是任意类型的值。通常来说 `action` 是一个对象，其中 `type` 属性标识类型，其它属性携带额外信息。

### 返回值 
`dispatch` 函数没有返回值。

### 注意 
+ `dispatch` 函数 *是为下一次渲染而更新 `state`*。因此在调用 `dispatch` 函数后读取 `state` 并不会拿到更新后的值，也就是说只能获取到调用前的值。
+ 如果你提供的新值与当前的 `state` 相同（使用 `Object.is` 比较），React 会 *跳过组件和子组件的重新渲染，这是一种优化手段*。虽然在跳过重新渲染前 React 可能会调用你的组件，但是这不应该影响你的代码。
+ React 会批量更新 `state`。`state` 会在 *所有事件函数执行完毕* 并且已经调用过它的 `set` 函数后进行更新，这可以防止在一个事件中多次进行重新渲染。如果在访问 DOM 等极少数情况下需要强制 React 提前更新，可以使用 `flushSync`。

## 用法 

### 向组件添加 reducer 
在组件的顶层作用域调用 `useReducer` 来创建一个用于管理状态（state）的 `reducer`。

```jsx
import { useReducer } from 'react';

function reducer(state, action) {
  // ...
}

function MyComponent() {
    const [state, dispatch] = useReducer(reducer, {age: 42});
    // ...
}
```

`useReducer` 返回一个由两个值组成的数组：

1. 当前的 `state`，首次渲染时为你提供的 *初始值*。
2. `dispatch` 函数，让你可以根据交互修改 `state`。

为了更新屏幕上的内容，使用一个表示用户操作的 `action` 来调用 `dispatch` 函数：

```jsx
function handleClick() {
  dispatch({ type: 'incremented_age' });
}
```

React `会把当前的` state 和这个 `action` 一起作为参数传给 `reducer` 函数，然后 `reducer` 计算并返回新的 `state`，最后 React 保存新的 `state`，并使用它渲染组件和更新 UI。

```jsx
import { useReducer } from 'react';

function reducer(state, action) {
  if (action.type === 'incremented_age') {
    return {
      age: state.age + 1
    };
  }
  throw Error('Unknown action.');
}

export default function Counter() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });

  return (
    <>
      <button onClick={() => {
        dispatch({ type: 'incremented_age' })
      }}>
        Increment age
      </button>
      <p>Hello! You are {state.age}.</p>
    </>
  );
}
```


