# useMemo

`useMemo` 是一个 React Hook，它在每次重新渲染的时候能够缓存计算的结果。

```jsx
const cachedValue = useMemo(calculateValue, dependencies);
```

## 参考 

### useMemo(calculateValue, dependencies) 
在组件的顶层调用 useMemo 来缓存每次重新渲染都需要计算的结果。

```jsx
import { useMemo } from 'react';

function TodoList({ todos, tab }) {
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab]
  );
  // ...
}
```

### 参数
+ `calculateValue`：要缓存计算值的函数。它应该是一个没有任何参数的纯函数，并且可以返回任意类型。React 将会在首次渲染时调用该函数；在之后的渲染中，如果 `dependencies` 没有发生变化，React 将直接返回相同值。否则，将会再次调用 `calculateValue` 并返回最新结果，然后缓存该结果以便下次重复使用。
+ `dependencies`：所有在 `calculateValue` 函数中使用的响应式变量组成的数组。响应式变量包括 props、state 和所有你直接在组件中定义的变量和函数。如果你在代码检查工具中 配置了 React，它将会确保每一个响应式数据都被正确地定义为依赖项。依赖项数组的长度必须是固定的并且必须写成 `[dep1, dep2, dep3]` 这种形式。React 使用 `Object.is` 将每个依赖项与其之前的值进行比较。

### 返回值 
在初次渲染时，`useMemo` 返回不带参数调用 `calculateValue` 的结果。
在接下来的渲染中，如果依赖项没有发生改变，它将返回上次缓存的值；否则将再次调用 `calculateValue`，并返回最新结果。

### 注意 
+ `useMemo` 是一个 React Hook，所以你只能 在组件的顶层 或者自定义 Hook 中调用它。你不能在循环语句或条件语句中调用它。如有需要，将其提取为一个新组件并使用 `state`。
+ 在严格模式下，为了 帮你发现意外的错误，React 将会 *调用你的计算函数两次*。这只是一个开发环境下的行为，并不会影响到生产环境。如果计算函数是一个纯函数（它本来就应该是），这将不会影响到代码逻辑。其中一次的调用结果将被忽略。
+ 除非有特定原因，React 不会丢弃缓存值。例如，在开发过程中，React 会在你编辑组件文件时丢弃缓存。无论是在开发环境还是在生产环境，如果你的组件在初始挂载期间被终止，React 都会丢弃缓存。在未来，React 可能会添加更多利用丢弃缓存的特性——例如，如果 React 在未来增加了对虚拟化列表的内置支持，那么丢弃那些滚出虚拟化列表视口的缓存是有意义的。你可以仅仅依赖 `useMemo` 作为性能优化手段。否则，使用 `state` 变量 或者 `ref` 可能更加合适。

这种缓存返回值的方式也叫做 记忆化（memoization），这也是该 Hook 叫做 useMemo 的原因。

## 用法 
### 跳过代价昂贵的重新计算 

在组件顶层调用 `useMemo` 以在重新渲染之间缓存计算结果：

```jsx
import { useMemo } from 'react';

function TodoList({ todos, tab, theme }) {
  const visibleTodos = useMemo(() => filterTodos(todos, tab), [todos, tab]);
  // ...
}
```

你需要给 `useMemo` 传递两样东西：
1. 一个没有任何参数的 `calculation` 函数，像这样 `() =>`，并且返回任何你想要的计算结果。
2. 一个由包含在你的组件中并在 `calculation` 中使用的所有值组成的 依赖列表。

在初次渲染时，你从 `useMemo` 得到的 值 将会是你的 `calculation` 函数执行的结果。

在随后的每一次渲染中，React 将会比较前后两次渲染中的 所有依赖项 是否相同。如果通过 `Object.is` 比较所有依赖项都没有发生变化，那么 `useMemo` 将会返回之前已经计算过的那个值。否则，React 将会重新执行 `calculation` 函数并且返回一个新的值。

换言之，`useMemo` 在多次重新渲染中缓存了 `calculation` 函数计算的结果直到依赖项的值发生变化。

*让我们通过一个示例来看看这在什么情况下是有用的。*

默认情况下，React 会在每次重新渲染时重新运行整个组件。例如，如果 `TodoList` 更新了 `state` 或从父组件接收到新的 `props`，`filterTodos` 函数将会重新运行：

```jsx
function TodoList({ todos, tab, theme }) {
  const visibleTodos = filterTodos(todos, tab);
  // ...
}
```

如果计算速度很快，这将不会产生问题。但是，当正在过滤转换一个大型数组，或者进行一些昂贵的计算，而数据没有改变，那么可能希望跳过这些重复计算。如果 `todos` 与 `tab` 都与上次渲染时相同，那么像之前那样将计算函数包装在 `useMemo` 中，便可以重用已经计算过的 `visibleTodos`。

这种缓存行为叫做 记忆化。

### 注意
你应该仅仅把 useMemo 作为性能优化的手段。如果没有它，你的代码就不能正常工作，那么请先找到潜在的问题并修复它。然后再添加 useMemo 以提高性能。

### 如何衡量计算过程的开销是否昂贵？
一般来说，除非要创建或循环遍历数千个对象，否则开销可能并不大。如果你想获得更详细的信息，可以在控制台来测量花费这上面的时间：

```jsx
console.time('filter array');
const visibleTodos = filterTodos(todos, tab);
console.timeEnd('filter array');
```

然后执行你正在监测的交互（例如，在输入框中输入文字）。你将会在控制台看到如下的日志 `filter array: 0.15ms`。如果全部记录的时间加起来很长（1ms 或者更多），那么记忆此计算结果是有意义的。作为对比，你可以将计算过程包裹在 `useMemo` 中，以验证该交互的总日志时间是否减少了：

```jsx
console.time('filter array');
const visibleTodos = useMemo(() => {
  return filterTodos(todos, tab); // 如果 todos 和 tab 都没有变化，那么将会跳过渲染。
}, [todos, tab]);
console.timeEnd('filter array');
```

`useMemo` 不会让首次渲染更快，它只会帮助你跳过不必要的更新工作。

请记住，你的开发设备可能比用户的设备性能更强大，因此最好人为降低当前浏览器性能来测试。例如，Chrome 提供了 [CPU Throttling](https://developer.chrome.com/blog/new-in-devtools-61/#throttling) 选项来降低浏览器性能。
另外，请注意，在开发环境中测量性能无法为你提供最准确的结果（例如，当开启 严格模式 时，你会看到每个组件渲染两次而不是一次）。要获得最准确的时间，请构建用于生产的应用程序并在用户使用的设备上对其进行测试。

### 你应该在所有地方添加 useMemo 吗？ 
如果你的应用程序类似于此站点，并且大多数交互都很粗糙（例如替换页面或整个章节），则通常不需要使用记忆化。反之，如果你的应用程序更像是绘图编辑器，并且大多数交互都是颗粒状的（如移动形状），那么你可能会发现记忆化非常有用。

使用 `useMemo` 进行优化仅在少数情况下有价值：

+ 你在 `useMemo` 中进行的计算明显很慢，而且它的依赖关系很少改变。
+ 将计算结果作为 `props` 传递给包裹在 `memo` 中的组件。当计算结果没有改变时，你会想跳过重新渲染。记忆化让组件仅在依赖项不同时才重新渲染。
+ 你传递的值稍后用作某些 Hook 的依赖项。例如，也许另一个 `useMemo` 计算值依赖它，或者 `useEffect` 依赖这个值

在其他情况下，将计算过程包装在 `useMemo` 中没有任何好处。不过这样做也没有重大危害，所以一些团队选择不考虑具体情况，尽可能多地使用 `useMemo`。不过这种做法会降低代码可读性。此外，并不是所有 `useMemo` 的使用都是有效的：一个“永远是新的”的单一值就足以破坏整个组件的记忆化效果。

#### 在实践中，你可以通过遵循一些原则来避免 `useMemo` 的滥用：
1. 当一个组件在视觉上包装其他组件时，让它 接受 JSX 作为子元素。随后，如果包装组件更新自己的 state，React 知道它的子组件不需要重新渲染。
2. 建议使用 state 并且不要 *提升状态* 超过必要的程度。不要将表单和项是否悬停等短暂状态保存在树的顶部或全局状态库中。
3. 保持 *渲染逻辑纯粹*。如果重新渲染组件会导致问题或产生一些明显的视觉瑕疵，那么这是组件自身的问题！请修复这个错误，而不是添加记忆化。
4. 避免 *不必要地更新 Effect。* React 应用程序中的大多数性能问题都是由 Effect 的更新链引起的，这些更新链不断导致组件重新渲染。
5. 尝试 *从 Effect 中删除不必要的依赖关系。* 例如，将某些对象或函数移动到副作用内部或组件外部通常更简单，而不是使用记忆化。
