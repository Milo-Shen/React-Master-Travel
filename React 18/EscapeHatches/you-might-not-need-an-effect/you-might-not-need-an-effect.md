# 你可能不需要 Effect

Effect 是 React 范式中的一个逃脱方案。它们让你可以 “逃出” React 并使组件和一些外部系统同步，比如非 React 组件、网络和浏览器 DOM。如果没有涉及到外部系统（例如，你想根据 props 或 state 的变化来更新一个组件的 state），你就不应该使用 Effect。移除不必要的 Effect 可以让你的代码更容易理解，运行得更快，并且更少出错。

## 你将会学习到
+ 为什么以及如何从你的组件中移除 Effect
+ 如何在没有 Effect 的情况下缓存昂贵的计算
+ 如何在没有 Effect 的情况下重置和调整组件的 state
+ 如何在事件处理函数之间共享逻辑
+ 应该将哪些逻辑移动到事件处理函数中
+ 如何将发生的变动通知到父组件

## 如何移除不必要的 Effect 

有两种不必使用 Effect 的常见情况:
+ *你不必使用 Effect 来转换渲染所需的数据。* 例如，你想在展示一个列表前先做筛选。你的直觉可能是写一个当列表变化时更新 state 变量的 Effect。然而，这是低效的。当你更新这个 state 时，React 首先会调用你的组件函数来计算应该显示在屏幕上的内容。然后 React 会把这些变化“提交”到 DOM 中来更新屏幕。然后 React 会执行你的 Effect。如果你的 Effect 也立即更新了这个 state，就会重新执行整个流程。为了避免不必要的渲染流程，应在你的组件顶层转换数据。这些代码会在你的 props 或 state 变化时自动重新执行。
+ *你不必使用 Effect 来处理用户事件。* 例如，你想在用户购买一个产品时发送一个 /api/buy 的 POST 请求并展示一个提示。在这个购买按钮的点击事件处理函数中，你确切地知道会发生什么。但是你不知道用户做了什么导致 Effect 被执行（例如，点击了某个按钮）。这就是为什么你通常应该在相应的事件处理函数中处理用户事件。

你 的确 可以使用 Effect 来和外部系统 同步 。例如，你可以写一个 Effect 来保持一个 jQuery 的组件和 React state 之间的同步。你也可以使用 Effect 来获取数据：例如，你可以同步当前的查询搜索和查询结果。请记住，比起直接在你的组件中写 Effect，现代 框架 提供了更加高效的，内置的数据获取机制。

为了帮助你获得正确的直觉，让我们来看一些常见的实例吧！

### 根据 props 或 state 来更新 state 

假设你有一个包含了两个 `state` 变量的组件：`firstName` 和 `lastName`。你想通过把它们联结起来计算出 `fullName`。此外，每当 `firstName` 和 `lastName` 变化时，你希望 `fullName` 都能更新。你的第一直觉可能是添加一个 `state` 变量：`fullName`，并在一个 Effect 中更新它：

```jsx
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');

  // 🔴 避免：多余的 state 和不必要的 Effect
  const [fullName, setFullName] = useState('');
  useEffect(() => {
    setFullName(firstName + ' ' + lastName);
  }, [firstName, lastName]);
  // ...
}
```

大可不必这么复杂。而且这样效率也不高：它先是用 `fullName` 的旧值执行了整个渲染流程，然后立即使用更新后的值又重新渲染了一遍。让我们移除 `state` 变量和 `Effect`：

```jsx
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');
  // ✅ 非常好：在渲染期间进行计算
  const fullName = firstName + ' ' + lastName;
  // ...
}
```

*如果一个值可以基于现有的 props 或 state 计算得出，不要把它作为一个 state，而是在渲染期间直接计算这个值。*  这将使你的代码更快（避免了多余的 “级联” 更新）、更简洁（移除了一些代码）以及更少出错（避免了一些因为不同的 state 变量之间没有正确同步而导致的问题）。如果这个观点对你来说很新奇，React 哲学 中解释了什么值应该作为 state。

### 缓存昂贵的计算 
这个组件使用它接收到的 `props` 中的 `filter` 对另一个 `prop` todos 进行筛选，计算得出 `visibleTodos`。你的直觉可能是把结果存到一个 `state` 中，并在 Effect 中更新它：

```jsx
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');

  // 🔴 避免：多余的 state 和不必要的 Effect
  const [visibleTodos, setVisibleTodos] = useState([]);
  useEffect(() => {
    setVisibleTodos(getFilteredTodos(todos, filter));
  }, [todos, filter]);

  // ...
}
```

就像之前的例子一样，这既没有必要，也很低效。首先，移除 `state` 和 Effect：

```jsx
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // ✅ 如果 getFilteredTodos() 的耗时不长，这样写就可以了。
  const visibleTodos = getFilteredTodos(todos, filter);
  // ...
}
```

一般来说，这段代码没有问题！但是，`getFilteredTodos()` 的耗时可能会很长，或者你有很多 `todos`。这些情况下，当 `newTodo` 这样不相关的 `state` 变量变化时，你并不想重新执行 `getFilteredTodos()`。

你可以使用 `useMemo` Hook 缓存（或者说 记忆（`memoize`））一个昂贵的计算。

```jsx
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  const visibleTodos = useMemo(() => {
    // ✅ 除非 todos 或 filter 发生变化，否则不会重新执行
    return getFilteredTodos(todos, filter);
  }, [todos, filter]);
  // ...
}
```

或者写成一行：

```jsx
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // ✅ 除非 todos 或 filter 发生变化，否则不会重新执行 getFilteredTodos()
  const visibleTodos = useMemo(() => getFilteredTodos(todos, filter), [todos, filter]);
  // ...
}
```

*这会告诉 React，除非 `todos` 或 `filter` 发生变化，否则不要重新执行传入的函数。* React 会在初次渲染的时候记住 `getFilteredTodos()` 的返回值。在下一次渲染中，它会检查 `todos` 或 `filter` 是否发生了变化。如果它们跟上次渲染时一样，`useMemo` 会直接返回它最后保存的结果。如果不一样，React 将再次调用传入的函数（并保存它的结果）。

你传入 useMemo 的函数会在渲染期间执行，所以它仅适用于 纯函数 场景。

### 如何判断计算是昂贵的？ 

一般来说只有你创建或循环遍历了成千上万个对象时才会很耗费时间。如果你想确认一下，可以添加控制台输出来测量某一段代码的执行时间：

```jsx
console.time('筛选数组');
const visibleTodos = getFilteredTodos(todos, filter);
console.timeEnd('筛选数组');
```

触发要测量的交互（例如，在输入框中输入）。你会在控制台中看到类似 筛选数组：0.15ms 这样的输出日志。如果总耗时达到了一定量级（比方说 1ms 或更多），那么把计算结果记忆（memoize）起来可能是有意义的。做一个实验，你可以把计算传入 useMemo 中来验证该交互导致的总耗时是减少了还是没什么变化：

```jsx
console.time('筛选数组');
const visibleTodos = useMemo(() => {
  return getFilteredTodos(todos, filter); // 如果 todos 或 filter 没有发生变化将跳过执行
}, [todos, filter]);
console.timeEnd('筛选数组');
```

`useMemo` 不会让 第一次 渲染变快。它只是帮助你跳过不必要的更新。

请注意，你的设备性能可能比用户的更好，因此最好通过人工限制工具来测试性能。例如，Chrome 提供了 CPU 节流 工具。

同时需要注意的是，在开发环境测试性能并不能得到最准确的结果（例如，当开启 严格模式 时，你会看到每个组件渲染了两次，而不是一次）。所以为了得到最准确的时间，需要将你的应用构建为生产模式，同时使用与你的用户性能相当的设备进行测试。

### 当 props 变化时重置所有 state
`ProfilePage` 组件接收一个 prop：`userId`。页面上有一个评论输入框，你用了一个 state：`comment` 来保存它的值。有一天，你发现了一个问题：当你从一个人的个人资料导航到另一个时，`comment` 没有被重置。这导致很容易不小心把评论发送到不正确的个人资料。为了解决这个问题，你想在 `userId` 变化时，清除 `comment` 变量：

```jsx
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  // 🔴 避免：当 prop 变化时，在 Effect 中重置 state
  useEffect(() => {
    setComment('');
  }, [userId]);
  // ...
}
```

但这是低效的，因为 `ProfilePage` 和它的子组件首先会用旧值渲染，然后再用新值重新渲染。并且这样做也很复杂，因为你需要在 `ProfilePage` 里面 所有 具有 `state` 的组件中都写这样的代码。例如，如果评论区的 UI 是嵌套的，你可能也想清除嵌套的 comment `state`。

取而代之的是，你可以通过为每个用户的个人资料组件提供一个明确的 `key` 来告诉 React 它们原则上是 不同 的个人资料组件。将你的组件拆分为两个组件，并从外部的组件传递一个 key 属性给内部的组件：

```jsx
export default function ProfilePage({ userId }) {
  return (
    <Profile
      userId={userId}
      key={userId}
    />
  );
}

function Profile({ userId }) {
  // ✅ 当 key 变化时，该组件内的 comment 或其他 state 会自动被重置
  const [comment, setComment] = useState('');
  // ...
}
```

通常，当在相同的位置渲染相同的组件时，React 会保留状态。*通过将 `userId` 作为 `key` 传递给 `Profile` 组件，使  React 将具有不同 `userId` 的两个 `Profile` 组件视为两个不应共享任何状态的不同组件。* 每当 `key`（这里是 `userId`）变化时，React 将重新创建 DOM，并 重置 `Profile` 组件和它的所有子组件的 `state`。现在，当在不同的个人资料之间导航时，`comment` 区域将自动被清空。

请注意，在这个例子中，只有外部的 `ProfilePage` 组件被导出并在项目中对其他文件可见。渲染 `ProfilePage` 的那些组件不用传递 `key` 给它：它们只需把 `userId` 作为常规 `prop` 传入即可。而 `ProfilePage` 将其作为 `key` 传递给内部的 `Profile` 组件是它的实现细节而已。

### 当 prop 变化时调整部分 state 

有时候，当 `prop` 变化时，你可能只想重置或调整部分 `state` ，而不是所有 `state`。

`List` 组件接收一个 `items` 列表作为 `prop`，然后用 `state` 变量 `selection` 来保持已选中的项。当 `items` 接收到一个不同的数组时，你想将 `selection` 重置为 `null`：

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 🔴 避免：当 prop 变化时，在 Effect 中调整 state
  useEffect(() => {
    setSelection(null);
  }, [items]);
  // ...
}
```

这不太理想。每当 `items` 变化时，`List` 及其子组件会先使用旧的 `selection` 值渲染。然后 React 会更新 DOM 并执行 Effect。最后，调用 `setSelection(null)` 将导致 `List` 及其子组件重新渲染，重新启动整个流程。

让我们从删除 Effect 开始。取而代之的是在渲染期间直接调整 `state`：

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 好一些：在渲染期间调整 state
  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}
```

像这样 *存储前序渲染的信息* 可能很难理解，但它比在 Effect 中更新这个 `state` 要好。上面的例子中，在渲染过程中直接调用了 `setSelection`。当它执行到 `return` 语句退出后，React 将 *立即* 重新渲染 `List`。此时 React 还没有渲染 `List` 的子组件或更新 DOM，这使得 `List` 的子组件可以跳过渲染旧的 `selection` 值。

在渲染期间更新组件时，React 会丢弃已经返回的 JSX 并立即尝试重新渲染。为了避免非常缓慢的级联重试，React 只允许在渲染期间更新 同一 组件的状态。如果你在渲染期间更新另一个组件的状态，你会看到一条报错信息。条件判断 `items !== prevItems` 是必要的，它可以避免无限循环。你可以像这样调整 `state`，但任何其他副作用（比如变化 DOM 或设置的延时）应该留在事件处理函数或 Effect 中，以 保持组件纯粹。

虽然这种方式比 Effect 更高效，但大多数组件也不需要它。无论你怎么做，根据 `props` 或其他 `state` 来调整 `state` 都会使数据流更难理解和调试。总是检查是否可以通过添加 `key` 来重置所有 `state`，或者 在渲染期间计算所需内容。例如，你可以存储已选中的 `item` `ID` 而不是存储（并重置）已选中的 `item`：

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // ✅ 非常好：在渲染期间计算所需内容
  const selection = items.find(item => item.id === selectedId) ?? null;
  // ...
}
```

现在完全不需要 “调整” `state` 了。如果包含已选中 `ID` 的项出现在列表中，则仍然保持选中状态。如果没有找到匹配的项，则在渲染期间计算的 `selection` 将会是 `null`。行为不同了，但可以说更好了，因为大多数对 `items` 的更改仍可以保持选中状态。

### 在事件处理函数中共享逻辑 
假设你有一个产品页面，上面有两个按钮（购买和付款），都可以让你购买该产品。当用户将产品添加进购物车时，你想显示一个通知。在两个按钮的 `click` 事件处理函数中都调用 `showNotification()` 感觉有点重复，所以你可能想把这个逻辑放在一个 Effect 中：

```jsx
function ProductPage({ product, addToCart }) {
  // 🔴 避免：在 Effect 中处理属于事件特定的逻辑
  useEffect(() => {
    if (product.isInCart) {
      showNotification(`已添加 ${product.name} 进购物车！`);
    }
  }, [product]);

  function handleBuyClick() {
    addToCart(product);
  }

  function handleCheckoutClick() {
    addToCart(product);
    navigateTo('/checkout');
  }
  // ...
}
```

这个 Effect 是多余的。而且很可能会导致问题。例如，假设你的应用在页面重新加载之前 “记住” 了购物车中的产品。如果你把一个产品添加到购物车中并刷新页面，通知将再次出现。每次刷新该产品页面时，它都会出现。这是因为 `product.isInCart` 在页面加载时已经是 `true` 了，所以上面的 Effect 每次都会调用 `showNotification()`。

当你不确定某些代码应该放在 Effect 中还是事件处理函数中时，先自问 为什么 要执行这些代码。Effect 只用来执行那些显示给用户时组件 需要执行 的代码。在这个例子中，通知应该在用户 按下按钮 后出现，而不是因为页面显示出来时！删除 Effect 并将共享的逻辑放入一个被两个事件处理程序调用的函数中：

```jsx
function ProductPage({ product, addToCart }) {
  // ✅ 非常好：事件特定的逻辑在事件处理函数中处理
  function buyProduct() {
    addToCart(product);
    showNotification(`已添加 ${product.name} 进购物车！`);
  }

  function handleBuyClick() {
    buyProduct();
  }

  function handleCheckoutClick() {
    buyProduct();
    navigateTo('/checkout');
  }
  // ...
}
```

这既移除了不必要的 Effect，又修复了问题。