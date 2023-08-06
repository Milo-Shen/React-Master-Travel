# useImperativeHandle

`useImperativeHandle` 是 React 中的一个 Hook，它能让你自定义由 `ref` 暴露出来的句柄。

```jsx
useImperativeHandle(ref, createHandle, dependencies?)
```

## 参考 

```jsx
useImperativeHandle(ref, createHandle, dependencies?)
```

在组件顶层通过调用 `useImperativeHandle` 来自定义 `ref` 暴露出来的句柄：

```jsx
import { forwardRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
    useImperativeHandle(ref, () => {
        return {
            // ... 你的方法 ...
        };
    }, []);
    // ...
})
```

### 参数 

+ `ref`：该 `ref` 是你从 `forwardRef` 渲染函数 中获得的第二个参数。
+ `createHandle`：该函数无需参数，它返回你想要暴露的 `ref` 的句柄。该句柄可以包含任何类型。通常，你会返回一个包含你想暴露的方法的对象。
+ 可选的 `dependencies`：函数 `createHandle` 代码中所用到的所有反应式的值的列表。反应式的值包含 `props`、状态和其他所有直接在你组件体内声明的变量和函数。倘若你的代码检查器已 为 React 配置好，它会验证每一个反应式的值是否被正确指定为依赖项。该列表的长度必须是一个常数项，并且必须按照 `[dep1, dep2, dep3]` 的形式罗列各依赖项。React 会使用 `Object.is` 来比较每一个依赖项与其对应的之前值。如果一次重新渲染导致某些依赖项发生了改变，或你没有提供这个参数列表，你的函数 `createHandle` 将会被重新执行，而新生成的句柄则会被分配给 `ref`。

### 返回值 

`useImperativeHandle` 返回 `undefined`。

## 使用方法 

### 向父组件暴露一个自定义的 ref 句柄 

默认情况下，组件不会将它们的 DOM 节点暴露给父组件。举例来说，如果你想要 `MyInput` 的父组件 能访问到 `<input>` DOM 节点，你必须选择使用 `forwardRef`:。

```jsx
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  return <input {...props} ref={ref} />;
});
```

在上方的代码中，`MyInput` 的 `ref` 会接收到 `<input>` DOM 节点。然而，你可以选择暴露一个自定义的值。为了修改被暴露的句柄，在你的顶层组件调用 `useImperativeHandle`：

```jsx
import { forwardRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  useImperativeHandle(ref, () => {
    return {
      // ... 你的方法 ...
    };
  }, []);

  return <input {...props} />;
});
```

注意在上述代码中，该 `ref` 已不再被转发到 `<input>` 中。

举例来说，假设你不想暴露出整个 `<input>` DOM 节点，但你想要它其中两个方法：`focus` 和 `scrollIntoView`。为此，用单独额外的 `ref` 来指向真实的浏览器 DOM。然后使用 `useImperativeHandle` 来暴露一个句柄，它只返回你想要父组件去调用的方法：

```jsx
import { forwardRef, useRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => {
    return {
      focus() {
        inputRef.current.focus();
      },
      scrollIntoView() {
        inputRef.current.scrollIntoView();
      },
    };
  }, []);

  return <input {...props} ref={inputRef} />;
});
```