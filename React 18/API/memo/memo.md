# memo
`memo` 允许你的组件在 `props` 没有改变的情况下跳过重新渲染。

## 参考 

### memo(Component, arePropsEqual?) 
使用 `memo` 将组件包装起来，以获得该组件的一个 *记忆化* 版本。通常情况下，只要该组件的 props 没有改变，这个记忆化版本就不会在其父组件重新渲染时重新渲染。但 React 仍可能会重新渲染它：记忆化是一种性能优化，而非保证。

```jsx
import { memo } from 'react';

const SomeComponent = memo(function SomeComponent(props) {
  // ...
});
```

### 参数 
+ `Component`：要进行记忆化的组件。memo 不会修改该组件，而是返回一个新的、记忆化的组件。它接受任何有效的 React 组件，包括函数组件和 `forwardRef` 组件。
+ 可选参数 `arePropsEqual`：一个函数，接受两个参数：组件的前一个 `props` 和新的 `props`。如果旧的和新的 `props` 相等，即组件使用新的 `props` 渲染的输出和表现与旧的 `props` 完全相同，则它应该返回 `true`。否则返回 `false`。通常情况下，你不需要指定此函数。默认情况下，React 将使用 `Object.is` 比较每个 `prop`。

### 返回值 
`memo` 返回一个新的 React 组件。它的行为与提供给 `memo` 的组件相同，只是当它的父组件重新渲染时 React 不会总是重新渲染它，除非它的 `props` 发生了变化。

## 用法 

### 当 `props` 没有改变时跳过重新渲染 
React 通常在其父组件重新渲染时重新渲染一个组件。你可以使用 `memo` 创建一个组件，当它的父组件重新渲染时，只要它的新 `props` 与旧 `props` 相同时，React 就不会重新渲染它。这样的组件被称为 记忆化的（`memoized`）组件。

要记忆化一个组件，请将它包装在 `memo` 中，使用它返回的值替换原来的组件：

```jsx
const Greeting = memo(function Greeting({ name }) {
  return <h1>Hello, {name}!</h1>;
});

export default Greeting;
```


React 组件应该始终具有 纯粹的渲染逻辑。这意味着如果其 `props`、`state` 和 `context` 没有改变，则必须返回相同的输出。通过使用 `memo`，你告诉 React 你的组件符合此要求，因此只要其 `props` 没有改变，React 就不需要重新渲染。即使使用 `memo`，如果它自己的 `state` 或正在使用的 `context` 发生更改，组件也会重新渲染。

在此示例中，请注意 `Greeting` 组件在 `name` 更改时重新渲染（因为那是它的 `props` 之一），但是在 `address` 更改时不会重新渲染（因为它不作为 `props` 传递给 `Greeting`）：

```jsx
import { memo, useState } from 'react';

export default function MyApp() {
  const [name, setName] = useState('');
  const [address, setAddress] = useState('');
  return (
    <>
      <label>
        Name{': '}
        <input value={name} onChange={e => setName(e.target.value)} />
      </label>
      <label>
        Address{': '}
        <input value={address} onChange={e => setAddress(e.target.value)} />
      </label>
      <Greeting name={name} />
    </>
  );
}

const Greeting = memo(function Greeting({ name }) {
  console.log("Greeting was rendered at", new Date().toLocaleTimeString());
  return <h3>Hello{name && ', '}{name}!</h3>;
});
```

### 注意
*你应该只将 `memo` 用作为性能优化*。如果你的代码没有 memo 就无法运行，首先找出潜在问题并进行修复。然后，你可以通过添加 `memo` 来提高性能。

### 在每个地方都应该添加 `memo` 吗？ 
如果你的应用像此站点一样，大多数交互是粗略的（例如直接替换页面或整个部分），那么通常不需要记忆化。另一方面，如果你的应用更像是绘图编辑器，大多数交互是细粒度的（例如移动图形），那么你可能会发现记忆化非常有用。

只有当你的组件经常使用完全相同的 `props` 重新渲染时，并且其重新渲染逻辑是非常昂贵的，使用 `memo` 优化才有价值。如果你的组件重新渲染时没有明显的延迟，那么 `memo` 就不必要了。请记住，如果传递给组件的 `props` 始终不同，例如在渲染期间传递对象或普通函数，则 `memo` 是完全无用的。这就是为什么你通常需要在 `memo` 中同时使用 `useMemo` 和 `useCallback`。

在其他情况下将组件包装在 `memo` 中是没有任何好处的。这种做法也没有什么明显的危害，因此一些团队会选择不考虑个别情况，并尽可能使用 `memo`。这种方法的缺点是代码变得不易读。此外，并不是所有的记忆化都是有效的：一个“总是新的”值足以破坏整个组件的记忆化。

*实践中，你可以通过遵循一些原则来使许多 memoization 变得不必要：*
1. 当一个组件在视觉上包裹其他组件时，让它 接受 JSX 作为子组件。这样，当包装组件更新其自身状态时，React 知道其子组件不需要重新渲染。
2. 优先使用局部状态，并且不要将 状态提升 到不必要的层级。例如，不要将短暂状态（如表单数据和项元素是否 `hover` 状态）保留在树的顶部或全局状态库中。
3. 保持你的 渲染逻辑纯粹。如果重新渲染组件会导致问题或产生一些明显的视觉瑕疵，则这是你组件中的 bug！修复 bug 而不是添加 memoization。
4. 避免 不必要的 Effect 来更新状态。React 应用中的大多数性能问题都是由于 Effect 引起的更新链，这些 Effect 会使你的组件一次又一次地重新渲染。
5. 尝试 从你的 Effect 中删除不必要的依赖项。例如，与其使用 memoization，不如将某些对象或函数移动到 Effect 内部或组件外部，这通常更简单。

如果特定交互仍然感觉不流畅，请 使用 React 开发者工具 profiler 来查看哪些组件最需要 memoization，并在需要时添加 memoization。这些原则使你的组件更易于调试和理解，因此建议在任何情况下都遵循它们。从长远来看，我们正在研究 自动进行细粒度 memoization，以解决这个问题。

