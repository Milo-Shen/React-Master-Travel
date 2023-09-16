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