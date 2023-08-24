# 响应式 Effect 的生命周期

Effect 与组件有不同的生命周期。组件可以挂载、更新或卸载。Effect 只能做两件事：开始同步某些东西，然后停止同步它。如果 Effect 依赖于随时间变化的 `props` 和 `state`，这个循环可能会发生多次。React 提供了代码检查规则来检查是否正确地指定了 Effect 的依赖项，这能够使 Effect 与最新的 `props` 和 `state` 保持同步。

## 你将会学习到
+ Effect 的生命周期与组件的生命周期有何不同
+ 如何独立地考虑每个 Effect
+ 什么时候以及为什么 Effect 需要重新同步
+ 如何确定 Effect 的依赖项
+ 值是响应式的含义是什么
+ 空依赖数组意味着什么
+ React 如何使用检查工具验证依赖关系是否正确
+ 与代码检查工具产生分歧时，该如何处理

## Effect 的生命周期 
每个 React 组件都经历相同的生命周期：
+ 当组件被添加到屏幕上时，它会进行组件的 *挂载*。
+ 当组件接收到新的 `props` 或 `state` 时，通常是作为对交互的响应，它会进行组件的 *更新*。
+ 当组件从屏幕上移除时，它会进行组件的 *卸载*。

*这是一种很好的思考组件的方式，但并不适用于 Effect*。相反，尝试从组件生命周期中跳脱出来，独立思考 Effect。Effect 描述了如何将外部系统与当前的 `props` 和 `state` 同步。随着代码的变化，同步的频率可能会增加或减少。

为了说明这一点，考虑下面这个示例。Effect 将组件连接到聊天服务器：

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, [roomId]);
  // ...
}
```

Effect 的主体部分指定了如何 *开始同步*：

```jsx
 // ...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
    // ...
```

Effect 返回的清理函数指定了如何 *停止同步*：

```jsx
    // ...
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => {
      connection.disconnect();
    };
    // ...
```

你可能会直观地认为当组件挂载时 React 会 *开始同步*，而当组件卸载时会 *停止同步*。然而，事情并没有这么简单！有时，在组件保持挂载状态的同时，可能还需要 *多次开始和停止同步*。

让我们来看看 *为什么* 这是必要的、*何时* 会发生以及 *如何* 控制这种行为。

### 注意
有些 Effect 根本不返回清理函数。在大多数情况下，可能希望返回一个清理函数，但如果没有返回，React 将表现得好像返回了一个空的清理函数。