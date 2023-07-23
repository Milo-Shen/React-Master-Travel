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

### 使用 useMemo 和直接计算之间的区别
#### 第 1 个示例 共 2 个挑战: 使用 useMemo 跳过重复计算 
在这个例子中，`filterTodos` 的执行被 人为减速了，这样就可以看到渲染期间调用的某些函数确实很慢时会发生什么。尝试切换选项卡并切换主题。

切换选项卡会感觉很慢，因为它迫使减速的 `filterTodos` 重新执行。这是预料之中的效果，因为“选项卡”已更改，因此整个计算 需要 重新运行。如果你好奇为什么它会运行两次，此处 对此进行了解释。

然后试试切换主题。在 `useMemo` 的帮助下，尽管已经被人为减速，但是它还是很快！缓慢的 `filterTodos` 调用被跳过，因为 `todos` 和 `tab`（你将其作为依赖项传递给 `useMemo`）自上次渲染以来都没有改变。

##### App.js

```jsx
import { useState } from 'react';
import { createTodos } from './utils.js';
import TodoList from './TodoList.js';

const todos = createTodos();

export default function App() {
  const [tab, setTab] = useState('all');
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <button onClick={() => setTab('all')}>
        All
      </button>
      <button onClick={() => setTab('active')}>
        Active
      </button>
      <button onClick={() => setTab('completed')}>
        Completed
      </button>
      <br />
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Dark mode
      </label>
      <hr />
      <TodoList
        todos={todos}
        tab={tab}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

##### TodoList.js

```jsx
import { useMemo } from 'react';
import { filterTodos } from './utils.js'

export default function TodoList({ todos, theme, tab }) {
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab]
  );
  return (
    <div className={theme}>
      <p><b>Note: <code>filterTodos</code> is artificially slowed down!</b></p>
      <ul>
        {visibleTodos.map(todo => (
          <li key={todo.id}>
            {todo.completed ?
              <s>{todo.text}</s> :
              todo.text
            }
          </li>
        ))}
      </ul>
    </div>
  );
}
```

##### utils.js

```jsx
export function createTodos() {
  const todos = [];
  for (let i = 0; i < 50; i++) {
    todos.push({
      id: i,
      text: "Todo " + (i + 1),
      completed: Math.random() > 0.5
    });
  }
  return todos;
}

export function filterTodos(todos, tab) {
  console.log('[ARTIFICIALLY SLOW] Filtering ' + todos.length + ' todos for "' + tab + '" tab.');
  let startTime = performance.now();
  while (performance.now() - startTime < 500) {
    // 在 500 毫秒内不执行任何操作以模拟极慢的代码
  }

  return todos.filter(todo => {
    if (tab === 'all') {
      return true;
    } else if (tab === 'active') {
      return !todo.completed;
    } else if (tab === 'completed') {
      return todo.completed;
    }
  });
}
```

### 跳过组件的重新渲染 
在某些情况下，`useMemo` 还可以帮助你优化重新渲染子组件的性能。为了说明这一点，假设这个 `TodoList` 组件将 `visibleTodos` 作为 `props` 传递给子 `List` 组件：

```jsx
export default function TodoList({ todos, tab, theme }) {
  // ...
  return (
    <div className={theme}>
      <List items={visibleTodos} />
    </div>
  );
}
```

你已经注意到切换 `theme` 属性会使应用程序冻结片刻，但是如果你从 JSX 中删除 `<List />`，感觉会很快。这说明尝试优化 `List` 组件是值得的。

默认情况下，当一个组件重新渲染时，React 会递归地重新渲染它的所有子组件。这就是为什么当 `TodoList` 使用不同的 `theme` 重新渲染时，`List` 组件 也会 重新渲染。这对于不需要太多计算来重新渲染的组件来说很好。但是如果你已经确认重新渲染很慢，你可以通过将它包装在 `memo` 中，这样当它的 `props` 跟上一次渲染相同的时候它就会跳过本次渲染：

```jsx
import { memo } from 'react';

const List = memo(function List({ items }) {
  // ...
});
```

通过此更改，如果 `List` 的所有 `props` 都与上次渲染时相同，则 `List` 将跳过重新渲染。这就是缓存计算变得重要的地方！想象一下，你在没有 `useMemo` 的情况下计算了 `visibleTodos`：

```jsx
export default function TodoList({ todos, tab, theme }) {
  // 每当主题发生变化时，这将是一个不同的数组……
  const visibleTodos = filterTodos(todos, tab);
  return (
    <div className={theme}>
      {/* ... 所以List的props永远不会一样，每次都会重新渲染 */}
      <List items={visibleTodos} />
    </div>
  );
}
```

在上面的示例中，`filterTodos` 函数总是创建一个不同数组，类似于 `{}` 总是创建一个新对象的方式。通常，这不是问题，但这意味着 `List` 属性永远不会相同，并且你的 `memo` 优化将不起作用。这就是 `useMemo` 派上用场的地方：

```jsx
export default function TodoList({ todos, tab, theme }) {
  // 告诉 React 在重新渲染之间缓存你的计算结果...
  const visibleTodos = useMemo(
    () => filterTodos(todos, tab),
    [todos, tab] // ...所以只要这些依赖项不变...
  );
  return (
    <div className={theme}>
      {/* ... List 也就会接受到相同的 props 并且会跳过重新渲染 */}
      <List items={visibleTodos} />
    </div>
  );
}
```

通过将 `visibleTodos` 的计算函数包裹在 `useMemo` 中，你可以确保它在重新渲染之间具有相同值，直到依赖项发生变化。你 不必 将计算函数包裹在 `useMemo` 中，除非你出于某些特定原因这样做。在此示例中，这样做的原因是你将它传递给包裹在 `memo` 中的组件，这使得它可以跳过重新渲染。添加 `useMemo` 的其他一些原因将在本页进一步描述。


