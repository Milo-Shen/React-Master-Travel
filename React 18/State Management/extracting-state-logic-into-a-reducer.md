# 迁移状态逻辑至 Reducer 中
对于拥有许多状态更新逻辑的组件来说，过于分散的事件处理程序可能会令人不知所措。对于这种情况，你可以将组件的所有状态更新逻辑整合到一个外部函数中，这个函数叫作 `reducer`。

## 你将会学习到
+ 什么是 reducer 函数
+ 如何将 useState 重构成 useReducer
+ 什么时候使用 reducer
+ 如何编写一个好的 reducer

## 使用 reducer 整合状态逻辑 
随着组件复杂度的增加，你将很难一眼看清所有的组件状态更新逻辑。例如，下面的 `TaskApp` 组件有一个数组类型的状态 `tasks`，并通过三个不同的事件处理程序来实现任务的添加、删除和修改：

```jsx
import { useState } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, setTasks] = useState(initialTasks);

  function handleAddTask(text) {
    setTasks([
      ...tasks,
      {
        id: nextId++,
        text: text,
        done: false,
      },
    ]);
  }

  function handleChangeTask(task) {
    setTasks(
      tasks.map((t) => {
        if (t.id === task.id) {
          return task;
        } else {
          return t;
        }
      })
    );
  }

  function handleDeleteTask(taskId) {
    setTasks(tasks.filter((t) => t.id !== taskId));
  }

  return (
    <>
      <h1>布拉格的行程安排</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  {id: 0, text: '参观卡夫卡博物馆', done: true},
  {id: 1, text: '看木偶戏', done: false},
  {id: 2, text: '打卡列侬墙', done: false},
];
```

这个组件的每个事件处理程序都通过 `setTasks` 来更新状态。随着这个组件的不断迭代，其状态逻辑也会越来越多。为了降低这种复杂度，并让所有逻辑都可以存放在一个易于理解的地方，你可以将这些状态逻辑移到组件之外的一个称为 *reducer* 的函数中。

Reducer 是处理状态的另一种方式。你可以通过三个步骤将 `useState` 迁移到 `useReducer`：

1. 将设置状态的逻辑 *修改* 成 dispatch 的一个 action；
2. *编写* 一个 reducer 函数；
3. 在你的组件中 *使用* reducer。

### 第 1 步: 将设置状态的逻辑修改成 dispatch 的一个 action
你的事件处理程序目前是通过设置状态来 *实现逻辑的*：

```jsx
function handleAddTask(text) {
  setTasks([
    ...tasks,
    {
      id: nextId++,
      text: text,
      done: false,
    },
  ]);
}

function handleChangeTask(task) {
  setTasks(
    tasks.map((t) => {
      if (t.id === task.id) {
        return task;
      } else {
        return t;
      }
    })
  );
}

function handleDeleteTask(taskId) {
  setTasks(tasks.filter((t) => t.id !== taskId));
}
```

移除所有的状态设置逻辑。只留下三个事件处理函数：

+ `handleAddTask(text)` 在用户点击 “添加” 时被调用。
+ `handleChangeTask(task)` 在用户切换任务或点击 “保存” 时被调用。
+ `handleDeleteTask(taskId)` 在用户点击 “删除” 时被调用。

使用 reducers 管理状态与直接设置状态略有不同。它不是通过设置状态来告诉 React “要做什么”，而是通过事件处理程序 dispatch 一个 “action” 来指明 “用户刚刚做了什么”。（而状态更新逻辑则保存在其他地方！）因此，我们不再通过事件处理器直接 “设置 `task`”，而是 dispatch 一个 “添加/修改/删除任务” 的 action。这更加符合用户的思维。

```jsx
function handleAddTask(text) {
  dispatch({
    type: 'added',
    id: nextId++,
    text: text,
  });
}

function handleChangeTask(task) {
  dispatch({
    type: 'changed',
    task: task,
  });
}

function handleDeleteTask(taskId) {
  dispatch({
    type: 'deleted',
    id: taskId,
  });
}
```

你传递给 `dispatch` 的对象叫做 “action”：

```jsx
function handleDeleteTask(taskId) {
  dispatch(
    // "action" 对象：
    {
      type: 'deleted',
      id: taskId,
    }
  );
}
```

它是一个普通的 JavaScript 对象。它的结构是由你决定的，但通常来说，它应该至少包含可以表明 *发生了什么事情* 的信息。（在后面的步骤中，你将会学习如何添加一个 `dispatch` 函数。）

#### action 对象可以有多种结构。
按照惯例，我们通常会添加一个字符串类型的 `type` 字段来描述发生了什么，并通过其它字段传递额外的信息。`type` 是特定于组件的，在这个例子中 `added` 和 `addded_task` 都可以。选一个能描述清楚发生的事件的名字！
```jsx
dispatch({
  // 针对特定的组件
  type: 'what_happened',
  // 其它字段放这里
});
```

### 第 2 步: 编写一个 reducer 函数 
reducer 函数就是你放置状态逻辑的地方。它接受两个参数，分别为当前 state 和 action 对象，并且返回的是更新后的 state：

```jsx
function yourReducer(state, action) {
  // 给 React 返回更新后的状态
}
```

React 会将状态设置为你从 reducer 返回的状态。

在这个例子中，要将状态设置逻辑从事件处理程序移到 reducer 函数中，你需要：

1. 声明当前状态（`tasks`）作为第一个参数；
2. 声明 `action` 对象作为第二个参数；
3. 从 `reducer` 返回 下一个 状态（React 会将旧的状态设置为这个最新的状态）。

下面是所有迁移到 `reducer` 函数的状态设置逻辑：

```jsx
function tasksReducer(tasks, action) {
  if (action.type === 'added') {
    return [
      ...tasks,
      {
        id: action.id,
        text: action.text,
        done: false,
      },
    ];
  } else if (action.type === 'changed') {
    return tasks.map((t) => {
      if (t.id === action.task.id) {
        return action.task;
      } else {
        return t;
      }
    });
  } else if (action.type === 'deleted') {
    return tasks.filter((t) => t.id !== action.id);
  } else {
    throw Error('未知 action: ' + action.type);
  }
}
```

由于 `reducer` 函数接受 `state`（tasks）作为参数，因此你可以 *在组件之外声明它*。这减少了代码的缩进级别，提升了代码的可读性。

#### 注意
上面的代码使用了 `if/els`e 语句，但是在 reducers 中使用 switch 语句 是一种惯例。两种方式结果是相同的，但 switch 语句读起来一目了然。

在本文档的后续部分我们会像这样使用：

```jsx
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('未知 action: ' + action.type);
    }
  }
}
```

我们建议将每个 `case` 块包装到 `{` 和 `}` 花括号中，这样在不同 `case` 中声明的变量就不会互相冲突。此外，`case` 通常应该以 `return` 结尾。如果你忘了 `return`，代码就会 进入 到下一个 `case`，这就会导致错误！

如果你还不熟悉 `switch` 语句，使用 `if/else` 也是可以的。

#### 为什么称之为 reducer? 
尽管 `reducer` 可以 “减少” 组件内的代码量，但它实际上是以数组上的 `reduce()` 方法命名的。

`reduce()` 允许你将数组中的多个值 “累加” 成一个值：

```jsx
const arr = [1, 2, 3, 4, 5];
const sum = arr.reduce(
  (result, number) => result + number
); // 1 + 2 + 3 + 4 + 5
```

你传递给 `reduce` 的函数被称为 “reducer”。它接受 `目前的结果` 和` 当前的值`，然后返回 `下一个结果`。React 中的 `reducer` 和这个是一样的：它们都接受 `目前的状态` 和 `action` ，然后返回 `下一个状态`。这样，`action` 会随着时间推移累积到状态中。

你甚至可以使用 `reduce()` 方法以及 `initialState` 和 `actions` 数组，通过传递你的 `reducer` 函数来计算最终的状态：

##### Index.js
```jsx
import tasksReducer from './tasksReducer.js';

let initialState = [];
let actions = [
  {type: 'added', id: 1, text: '参观卡夫卡博物馆'},
  {type: 'added', id: 2, text: '看木偶戏'},
  {type: 'deleted', id: 1},
  {type: 'added', id: 3, text: '打卡列侬墙'},
];

let finalState = actions.reduce(tasksReducer, initialState);

const output = document.getElementById('output');
output.textContent = JSON.stringify(finalState, null, 2);
```

##### tasksReducer.js
```jsx
export default function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('未知 action: ' + action.type);
    }
  }
}
```

##### result
```json
[
  {
    "id": 2,
    "text": "看木偶戏",
    "done": false
  },
  {
    "id": 3,
    "text": "打卡列侬墙",
    "done": false
  }
]
```

你可能不需要自己做这些，但这与 React 所做的很相似！

### 第 3 步: 在组件中使用 reducer 
最后，你需要将 `tasksReducer` 导入到组件中。记得先从 React 中导入 `useReducer` Hook：

```jsx
import { useReducer } from 'react';
```

接下来，你就可以替换掉之前的 `useState`:

```jsx
const [tasks, setTasks] = useState(initialTasks);
```

只需要像下面这样使用 `useReducer`:

```jsx
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

`useReducer` 和 `useState` 很相似——你必须给它传递一个初始状态，它会返回一个有状态的值和一个设置该状态的函数（在这个例子中就是 dispatch 函数）。但是，它们两个之间还是有点差异的。

`useReducer` 钩子接受 2 个参数：

1. 一个 reducer 函数
2. 一个初始的 state

它返回如下内容：

1. 一个有状态的值
2. 一个 dispatch 函数（用来 “派发” 用户操作给 reducer）

现在一切都准备就绪了！我们在这里把 reducer 定义在了组件的末尾：

#### App.js
```jsx
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>布拉格的行程安排</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('未知 action: ' + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  {id: 0, text: '参观卡夫卡博物馆', done: true},
  {id: 1, text: '看木偶戏', done: false},
  {id: 2, text: '打卡列侬墙', done: false}
];
```

如果有需要，你甚至可以把 reducer 移到一个单独的文件中：

#### App.js
```jsx
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';
import tasksReducer from './tasksReducer.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>布拉格的行程安排</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  {id: 0, text: '参观卡夫卡博物馆', done: true},
  {id: 1, text: '看木偶戏', done: false},
  {id: 2, text: '打卡列侬墙', done: false},
];
```

#### taskReducer.js
```jsx
export default function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('未知 action：' + action.type);
    }
  }
}
```

当像这样分离关注点时，我们可以更容易地理解组件逻辑。现在，事件处理程序只通过派发 `action` 来指定 *发生了什么*，而 `reducer` 函数通过响应 `actions` 来决定 *状态如何更新*。

## 对比 `useState` 和 `useReducer` 
Reducers 并非没有缺点！以下是比较它们的几种方法：

+ *代码体积*： 通常，在使用 `useState` 时，一开始只需要编写少量代码。而 `useReducer` 必须提前编写 `reducer` 函数和需要调度的 `actions`。但是，当多个事件处理程序以相似的方式修改 `state` 时，`useReducer` 可以减少代码量。
+ *可读性*： 当状态更新逻辑足够简单时，`useState` 的可读性还行。但是，一旦逻辑变得复杂起来，它们会使组件变得臃肿且难以阅读。在这种情况下，`useReducer` 允许你将状态更新逻辑与事件处理程序分离开来。
+ *可调试性*： 当使用 `useState` 出现问题时, 你很难发现具体原因以及为什么。 而使用 `useReducer` 时， 你可以在 `reducer` 函数中通过打印日志的方式来观察每个状态的更新，以及为什么要更新（来自哪个 `action`）。 如果所有 `action` 都没问题，你就知道问题出在了 `reducer` 本身的逻辑中。 然而，与使用 `useState` 相比，你必须单步执行更多的代码。
+ *可测试性*： `reducer` 是一个不依赖于组件的纯函数。这就意味着你可以单独对它进行测试。一般来说，我们最好是在真实环境中测试组件，但对于复杂的状态更新逻辑，针对特定的初始状态和 `action`，断言 `reducer` 返回的特定状态会很有帮助。
+ *个人偏好*： 并不是所有人都喜欢用 `reducer`，没关系，这是个人偏好问题。你可以随时在 `useState` 和 `useReducer` 之间切换，它们能做的事情是一样的！

如果你在修改某些组件状态时经常出现问题或者想给组件添加更多逻辑时，我们建议你还是使用 `reducer`。当然，你也不必整个项目都用 `reducer`，这是可以自由搭配的。你甚至可以在一个组件中同时使用 `useState` 和 `useReducer`。

## 编写一个好的 reducers 
编写 `reducers` 时最好牢记以下两点：

+ *reducers 必须是纯粹的*。 这一点和 状态更新函数 是相似的，`reducers` 在是在渲染时运行的！（`actions` 会排队直到下一次渲染)。 这就意味着 `reducers` 必须纯净，即当输入相同时，输出也是相同的。它们 *不应该包含异步请求、定时器或者任何副作用（对组件外部有影响的操作）* 。它们 *应该以不可变值的方式去更新 对象 和 数组*。
+ *每个 action 都描述了一个单一的用户交互，即使它会引发数据的多个变化*。 举个例子，如果用户在一个由 `reducer` 管理的表单（包含五个表单项）中点击了 重置按钮，那么 `dispatch` 一个 `reset_form` 的 `action` 比 `dispatch` 五个单独的 `set_field` 的 `action` 更加合理。如果你在一个 `reducer` 中打印了所有的 `action` 日志，那么这个日志应该是很清晰的，它能让你以某种步骤复现已发生的交互或响应。这对代码调试很有帮助！

## 使用 Immer 简化 reducers 
与在平常的 `state` 中 修改对象 和 数组 一样，你可以使用 `Immer` 这个库来简化 `reducer`。在这里，`useImmerReducer` 让你可以通过 `push` 或 `arr[i] =` 来修改 state ：

### App.js
```jsx
import { useImmerReducer } from 'use-immer';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

function tasksReducer(draft, action) {
  switch (action.type) {
    case 'added': {
      draft.push({
        id: action.id,
        text: action.text,
        done: false,
      });
      break;
    }
    case 'changed': {
      const index = draft.findIndex((t) => t.id === action.task.id);
      draft[index] = action.task;
      break;
    }
    case 'deleted': {
      return draft.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('未知 action：' + action.type);
    }
  }
}

export default function TaskApp() {
  const [tasks, dispatch] = useImmerReducer(tasksReducer, initialTasks);

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task,
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId,
    });
  }

  return (
    <>
      <h1>布拉格的行程安排</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  {id: 0, text: '参观卡夫卡博物馆', done: true},
  {id: 1, text: '看木偶戏', done: false},
  {id: 2, text: '打卡列侬墙', done: false},
];
```

Reducers 应该是纯净的，所以它们不应该去修改 state。而 Immer 为你提供了一种特殊的 `draft` 对象，你可以通过它安全的修改 state。在底层，Immer 会基于当前 state 创建一个副本。这就是为什么通过 `useImmerReducer` 来管理 reducers 时，可以修改第一个参数，且不需要返回一个新的 state 的原因。

## 摘要
+ 把 `useState` 转化为 `useReducer`：
     通过事件处理函数 dispatch actions；
     编写一个 reducer 函数，它接受传入的 state 和一个 action，并返回一个新的 state；
     使用 `useReducer` 替换 `useState`；
+ Reducers 可能需要你写更多的代码，但是这有利于代码的调试和测试。
+ Reducers 必须是纯净的。
+ 每个 action 都描述了一个单一的用户交互。
+ 使用 Immer 来帮助你在 reducer 里直接修改状态。

## 从零开始实现 `useReducer` 
### App.js
```jsx
import { useReducer } from './MyReact.js';
import Chat from './Chat.js';
import ContactList from './ContactList.js';
import { initialState, messengerReducer } from './messengerReducer';

export default function Messenger() {
  const [state, dispatch] = useReducer(messengerReducer, initialState);
  const message = state.messages[state.selectedId];
  const contact = contacts.find((c) => c.id === state.selectedId);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedId={state.selectedId}
        dispatch={dispatch}
      />
      <Chat
        key={contact.id}
        message={message}
        contact={contact}
        dispatch={dispatch}
      />
    </div>
  );
}

const contacts = [
  {id: 0, name: 'Taylor', email: 'taylor@mail.com'},
  {id: 1, name: 'Alice', email: 'alice@mail.com'},
  {id: 2, name: 'Bob', email: 'bob@mail.com'},
];
```

### messengerReducer.js
```jsx
export const initialState = {
     selectedId: 0,
     messages: {
          0: 'Hello, Taylor',
          1: 'Hello, Alice',
          2: 'Hello, Bob',
     },
};

export function messengerReducer(state, action) {
     switch (action.type) {
          case 'changed_selection': {
               return {
                    ...state,
                    selectedId: action.contactId,
               };
          }
          case 'edited_message': {
               return {
                    ...state,
                    messages: {
                         ...state.messages,
                         [state.selectedId]: action.message,
                    },
               };
          }
          case 'sent_message': {
               return {
                    ...state,
                    messages: {
                         ...state.messages,
                         [state.selectedId]: '',
                    },
               };
          }
          default: {
               throw Error('未知 action：' + action.type);
          }
     }
}
```

### MyReact.js
```jsx
import { useState } from 'react';

export function useReducer(reducer, initialState) {
  const [state, setState] = useState(initialState);

  function dispatch(action) {
    const nextState = reducer(state, action);
    setState(nextState);
  }

  return [state, dispatch];
}
```

虽然在大多数情况下这并不重要，但更准确的实现是这样的：

```jsx
function dispatch(action) {
  setState((s) => reducer(s, action));
}
```

这是因为被派发的 actions 在下一次渲染之前都是处于排队状态的，这和 *状态更新函数* 类似。