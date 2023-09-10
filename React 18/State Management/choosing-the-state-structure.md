# 选择 State 结构
构建良好的 `state` 可以让组件变得易于修改和调试，而不会经常出错。以下是你在构建 `state` 时应该考虑的一些建议。

## 你将会学习到
+ 使用单个 state 变量还是多个 state 变量
+ 组织 state 时应避免的内容
+ 如何解决 state 结构中的常见问题

## 构建 state 的原则 
当你编写一个存有 state 的组件时，你需要选择使用多少个 state 变量以及它们都是怎样的数据格式。尽管选择次优的 state 结构下也可以编写正确的程序，但有几个原则可以指导您做出更好的决策：

1. *合并关联的 state*。如果你总是同时更新两个或更多的 state 变量，请考虑将它们合并为一个单独的 state 变量。
2. *避免互相矛盾的 state*。当 state 结构中存在多个相互矛盾或“不一致”的 state 时，你就可能为此会留下隐患。应尽量避免这种情况。
3. *避免冗余的 state*。如果你能在渲染期间从组件的 props 或其现有的 state 变量中计算出一些信息，则不应将这些信息放入该组件的 state 中。
4. *避免重复的 state*。当同一数据在多个 state 变量之间或在多个嵌套对象中重复时，这会很难保持它们同步。应尽可能减少重复。
5. *避免深度嵌套的 state*。深度分层的 state 更新起来不是很方便。如果可能的话，最好以扁平化方式构建 state。

这些原则背后的目标是 *使 state 易于更新而不引入错误*。从 state 中删除冗余和重复数据有助于确保所有部分保持同步。这类似于数据库工程师想要 “规范化”数据库结构，以减少出现错误的机会。用爱因斯坦的话说，*“让你的状态尽可能简单，但不要过于简单。”*

现在让我们来看看这些原则在实际中是如何应用的。

## 合并关联的 state
有时候你可能会不确定是使用单个 state 变量还是多个 state 变量。

你会像下面这样做吗？

```jsx
const [x, setX] = useState(0);
const [y, setY] = useState(0);
```

或这样？

```jsx
const [position, setPosition] = useState({ x: 0, y: 0 });
```

从技术上讲，你可以使用其中任何一种方法。但是，*如果某两个 state 变量总是一起变化，则将它们统一成一个 state 变量可能更好*。这样你就不会忘记让它们始终保持同步，就像下面这个例子中，移动光标会同时更新红点的两个坐标：

```jsx
import { useState } from 'react';

export default function MovingDot() {
  const [position, setPosition] = useState({
    x: 0,
    y: 0
  });
  return (
    <div
      onPointerMove={e => {
        setPosition({
          x: e.clientX,
          y: e.clientY
        });
      }}
      style={{
        position: 'relative',
        width: '100vw',
        height: '100vh',
      }}>
      <div style={{
        position: 'absolute',
        backgroundColor: 'red',
        borderRadius: '50%',
        transform: `translate(${position.x}px, ${position.y}px)`,
        left: -10,
        top: -10,
        width: 20,
        height: 20,
      }} />
    </div>
  )
}
```

另一种情况是，你将数据整合到一个对象或一个数组中时，你不知道需要多少个 state 片段。例如，当你有一个用户可以添加自定义字段的表单时，这将会很有帮助。

### 陷阱
如果你的 `state` 变量是一个对象时，请记住，你不能只更新其中的一个字段 而不显式复制其他字段。例如，在上面的例子中，你不能写成 `setPosition({ x: 100 })`，因为它根本就没有 y 属性! 相反，如果你想要仅设置 x，则可执行 `setPosition({ ...position, x: 100 })`，或将它们分成两个 state 变量，并执行 `setX(100)`。

## 避免矛盾的 state 
下面是带有 `isSending` 和 `isSent` 两个 state 变量的酒店反馈表单：

```jsx
import { useState } from 'react';

export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [isSending, setIsSending] = useState(false);
  const [isSent, setIsSent] = useState(false);

  async function handleSubmit(e) {
    e.preventDefault();
    setIsSending(true);
    await sendMessage(text);
    setIsSending(false);
    setIsSent(true);
  }

  if (isSent) {
    return <h1>Thanks for feedback!</h1>
  }

  return (
    <form onSubmit={handleSubmit}>
      <p>How was your stay at The Prancing Pony?</p>
      <textarea
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button
        disabled={isSending}
        type="submit"
      >
        Send
      </button>
      {isSending && <p>Sending...</p>}
    </form>
  );
}

// 假装发送一条消息。
function sendMessage(text) {
  return new Promise(resolve => {
    setTimeout(resolve, 2000);
  });
}
```

尽管这段代码是有效的，但也会让一些 `state` “极难处理”。例如，如果你忘记同时调用 `setIsSent` 和 `setIsSending`，则可能会出现 `isSending` 和 `isSent` 同时为 `true` 的情况。你的组件越复杂，你就越难理解发生了什么。

*因为 `isSending` 和 `isSent` 不应同时为 `true`，`所以最好用一个` status 变量来代替它们，这个 `state` 变量可以采取三种有效状态其中之一：* `'typing'` (初始), `'sending'`, 和 `'sent'`:

```jsx
import { useState } from 'react';

export default function FeedbackForm() {
  const [text, setText] = useState('');
  const [status, setStatus] = useState('typing');

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('sending');
    await sendMessage(text);
    setStatus('sent');
  }

  const isSending = status === 'sending';
  const isSent = status === 'sent';

  if (isSent) {
    return <h1>Thanks for feedback!</h1>
  }

  return (
    <form onSubmit={handleSubmit}>
      <p>How was your stay at The Prancing Pony?</p>
      <textarea
        disabled={isSending}
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button
        disabled={isSending}
        type="submit"
      >
        Send
      </button>
      {isSending && <p>Sending...</p>}
    </form>
  );
}

// 假装发送一条消息。
function sendMessage(text) {
  return new Promise(resolve => {
    setTimeout(resolve, 2000);
  });
}
```

你仍然可以声明一些常量，以提高可读性：

```jsx
const isSending = status === 'sending';
const isSent = status === 'sent';
```

但它们不是 state 变量，所以你不必担心它们彼此失去同步。