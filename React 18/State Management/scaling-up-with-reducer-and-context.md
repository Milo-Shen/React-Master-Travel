# 使用 Reducer 和 Context 拓展你的应用

Reducer 可以整合组件的状态更新逻辑。Context 可以将信息深入传递给其他组件。你可以组合使用它们来共同管理一个复杂页面的状态。

## 你将会学习到
+ 如何结合使用 reducer 和 context
+ 如何去避免通过 props 传递 state 和 dispatch
+ 如何将 context 和状态逻辑保存在一个单独的文件中

## 结合使用 reducer 和 context 
在 reducer 介绍 的例子里面，状态被 reducer 所管理。reducer 函数包含了所有的状态更新逻辑并在此文件的底部声明：

### App.js
```jsx
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

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
      task: task
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId
    });
  }

  return (
    <>
      <h1>Day off in Kyoto</h1>
      <AddTask
        onAddTask={handleAddTask}
      />
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
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: 'Philosopher’s Path', done: true },
  { id: 1, text: 'Visit the temple', done: false },
  { id: 2, text: 'Drink matcha', done: false }
];
```

### AddTask.js
```jsx
import { useState } from 'react';

export default function AddTask({ onAddTask }) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="Add task"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        onAddTask(text);
      }}>Add</button>
    </>
  )
}
```

### TaskList.js
```jsx
import { useState } from 'react';

export default function TaskList({
  tasks,
  onChangeTask,
  onDeleteTask
}) {
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task
            task={task}
            onChange={onChangeTask}
            onDelete={onDeleteTask}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ task, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={e => {
            onChange({
              ...task,
              text: e.target.value
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Save
        </button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>
          Edit
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={e => {
          onChange({
            ...task,
            done: e.target.checked
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(task.id)}>
        Delete
      </button>
    </label>
  );
}
```

Reducer 有助于保持事件处理程序的简短明了。但随着应用规模越来越庞大，你就可能会遇到别的困难。目前，`tasks` 状态和 `dispatch` 函数仅在顶级 `TaskApp` 组件中可用。要让其他组件读取任务列表或更改它，你必须显式 传递 当前状态和事件处理程序，将其作为 props。

例如，`TaskApp` 将 一系列 task 和事件处理程序传递给 `TaskList`：

```jsx
<TaskList
  tasks={tasks}
  onChangeTask={handleChangeTask}
  onDeleteTask={handleDeleteTask}
/>
```

`TaskList` 将事件处理程序传递给 `Task`：

```jsx
<Task
  task={task}
  onChange={onChangeTask}
  onDelete={onDeleteTask}
/>
```

在像这样的小示例里这样做没什么问题，但是如果你有成千上百个组件，传递所有状态和函数可能会非常麻烦！

这就是为什么，比起通过 `props` 传递它们，你可能想把 `tasks` 状态和 `dispatch` 函数都 放入 context。*这样，所有的在 `TaskApp` 组件树之下的组件都不必一直往下传 `props` 而可以直接读取 `tasks` 和 `dispatch` 函数。*

下面将介绍如何结合使用 reducer 和 context：

1. 创建 context。
2. 将 state 和 dispatch 放入 context。
3. 在组件树的任何地方 使用 context。

### 第一步: 创建 context 
`useReducer` 返回当前的 `tasks` 和 `dispatch` 函数来让你更新它们：

```jsx
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

为了将它们从组件树往下传，你将 创建 两个不同的 context：

+ TasksContext 提供当前的 tasks 列表。
+ TasksDispatchContext 提供了一个函数可以让组件分发动作。

将它们从单独的文件导出，以便以后可以从其他文件导入它们：

#### TasksContext.js
```jsx
import { createContext } from 'react';

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```

在这里，你把 `null` 作为默认值传递给两个 context。实际值是由 `TaskApp` 组件提供的。
