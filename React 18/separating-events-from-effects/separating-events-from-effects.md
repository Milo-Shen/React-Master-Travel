# 将事件从 Effect 中分开
事件处理函数只有在你再次执行同样的交互时才会重新运行。Effect 和事件处理函数不一样，它只有在读取的 props 或 state 值和上一次渲染不一样时才会重新同步。有时你需要这两种行为的混合体：即一个 Effect 只在响应某些值时重新运行，但是在其他值变化时不重新运行。本章将会教你怎么实现这一点。

## 你将会学习到
+ 怎么在事件处理函数和 Effect 之间做选择
+ 为什么 Effect 是响应式的，而事件处理函数不是
+ 当你想要 Effect 的部分代码变成非响应式时要做些什么
+ Effect Event 是什么，以及怎么从 Effect 中提取
+ 怎么使用 Effect Event 读取最新的 props 和 state

### 在事件处理函数和 Effect 中做选择 
首先让我们回顾一下事件处理函数和 Effect 的区别。

假设你正在实现一个聊天室组件，需求如下：

1. 组件应该自动连接选中的聊天室。
2. 每当你点击 “Send” 按钮，组件应该在当前聊天界面发送一条消息。

假设你已经实现了这部分代码，但是还没有确定应该放在哪里。你是应该用事件处理函数还是 Effect 呢？每当你需要回答这个问题时，请考虑一下 为什么代码需要运行。

### 事件处理函数只在响应特定的交互操作时运行 
从用户角度出发，发送消息是 *因为* 他点击了特定的“Send”按钮。如果在任意时间或者因为其他原因发送消息，用户会觉得非常混乱。这就是为什么发送消息应该使用事件处理函数。事件处理函数是让你处理特定的交互操作的：

```jsx
function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');
  // ...
  function handleSendClick() {
    sendMessage(message);
  }
  // ...
  return (
    <>
      <input value={message} onChange={e => setMessage(e.target.value)} />
      <button onClick={handleSendClick}>Send</button>;
    </>
  );
}
```

借助事件处理函数，你可以确保 `sendMessage(message)` *只* 在用户点击按钮的时候运行。