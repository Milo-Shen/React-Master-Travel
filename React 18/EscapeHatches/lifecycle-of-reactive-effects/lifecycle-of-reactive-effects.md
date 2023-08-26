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

### 为什么同步可能需要多次进行 
想象一下，这个 `ChatRoom` 组件接收 `roomId` 属性，用户可以在下拉菜单中选择。假设初始时，用户选择了 `"general"` 作为 `roomId`。应用程序会显示 `"general"` 聊天室：

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId /* "general" */ }) {
  // ...
  return <h1>欢迎来到 {roomId} 房间！</h1>;
}
```

在 UI 显示之后，React 将运行 Effect 来 开始同步。它连接到 `"general"` 聊天室：

```jsx
function ChatRoom({ roomId /* "general" */ }) {
    useEffect(() => {
        const connection = createConnection(serverUrl, roomId); // 连接到 "general" 聊天室
        connection.connect();
        return () => {
            connection.disconnect(); // 断开与 "general" 聊天室的连接
        };
    }, [roomId]);
    // ...
}
```

到目前为止，一切都很顺利。

之后，用户在下拉菜单中选择了不同的房间（例如 `"travel"` ）。首先，React 会更新 UI：

思考接下来应该发生什么。用户在界面中看到 `"travel"` 是当前选定的聊天室。然而，上次运行的 Effect 仍然连接到 `"general"` 聊天室。*`roomId` 属性已经发生了变化，所以之前 Effect 所做的事情（连接到 `"general"` 聊天室）不再与 UI 匹配。*

此时，你希望 React 执行两个操作：
1. 停止与旧的 roomId 同步（断开与 `"general"` 聊天室的连接）
2. 开始与新的 roomId 同步（连接到 `"travel"` 聊天室）

幸运的是，你已经教会了 React 如何执行这两个操作！Effect 的主体部分指定了如何开始同步，而清理函数指定了如何停止同步。现在，React 只需要按照正确的顺序和正确的 `props` 和 `state` 来调用它们。让我们看看具体是如何实现的。

### React 如何重新同步 Effect 
回想一下，`ChatRoom` 组件已经接收到了 `roomId` 属性的新值。之前它是 `"general"`，现在变成了 `"travel"`。React 需要重新同步 Effect，以重新连接到不同的聊天室。

为了 *停止同步*，React 将调用 Effect 返回的清理函数，该函数在连接到 `"general"` 聊天室后返回。由于 `roomId` 为 `"general"`，清理函数将断开与 `"general"` 聊天室的连接：

```jsx
function ChatRoom({ roomId /* "general" */ }) {
  useEffect(() => {
      const connection = createConnection(serverUrl, roomId); // 连接到 "general" 聊天室
      connection.connect();
      return () => {
          connection.disconnect(); // 断开与 "general" 聊天室的连接
      };
      // ...
  })
}
```

然后，React 将运行在此渲染期间提供的 Effect。这次，`roomId` 为 `"travel"`，因此它将 开始同步 到 `"travel"` 聊天室（直到最终也调用了清理函数）：

```jsx
function ChatRoom({ roomId /* "travel" */ }) {
    useEffect(() => {
        const connection = createConnection(serverUrl, roomId); // 连接到 "travel" 聊天室
        connection.connect();
        // ...

    })
}
```

多亏了这一点，现在已经连接到了用户在 UI 中选择的同一个聊天室。避免了灾难！

每当组件使用不同的 `roomId` 重新渲染后，Effect 将重新进行同步。例如，假设用户将 `roomId` 从 `"travel"` 更改为 `"music"`。React 将再次通过调用清理函数 停止同步 Effect（断开与 `"travel"` 聊天室的连接）。然后，它将通过使用新的 `roomId` 属性再次运行 Effect 的主体部分 开始同步（连接到 `"music"` 聊天室）。

