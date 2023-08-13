# useState

`useState` 是一个 React Hook，它允许你向组件添加一个 状态变量。

```jsx
const [state, setState] = useState(initialState);
```

## 参考 

```jsx
useState(initialState);
```

在组件的顶层调用 `useState` 来声明一个 状态变量。

```jsx
import { useState } from 'react';

function MyComponent() {
    const [age, setAge] = useState(28);
    const [name, setName] = useState('Taylor');
    
    // react 会调用，后方的回调函数
    const [todos, setTodos] = useState(() => createTodos());
    
    // 这里写成 createTodos 效果也是一样的
    const [_todos, _setTodos] = useState(createTodos);
    // ...
}
```

按照惯例使用 数组解构 来命名状态变量，例如 `[something, setSomething]`。

### 参数 
+ `initialState`：你希望 s`tate 初始化的值。它可以是任何类型的值，但对于函数有特殊的行为。在初始渲染后，此参数将被忽略。
+ 如果传递函数作为 `initialState`，则它将被视为 初始化函数。它应该是纯函数，不应该接受任何参数，并且应该返回一个任何类型的值。当初始化组件时，React 将调用你的初始化函数，并将其返回值存储为初始状态。请参见下面的示例。

### 返回
`useState` 返回一个由两个值组成的数组：
1. 当前的 state。在首次渲染时，它将与你传递的 `initialState` 相匹配。
2. `set` 函数，它可以让你将 `state` 更新为不同的值并触发重新渲染。

### 注意事项 
`useState` 是一个 Hook，因此你只能在 组件的顶层 或自己的 Hook 中调用它。你不能在循环或条件语句中调用它。如果你需要这样做，请提取一个新组件并将状态移入其中。
在严格模式中，React 将 两次调用初始化函数，以 帮你找到意外的不纯性。这只是开发时的行为，不影响生产。如果你的初始化函数是纯函数（本该是这样），就不应影响该行为。其中一个调用的结果将被忽略。

`set` 函数，例如 `setSomething(nextState) `
`useState` 返回的 `set` 函数允许你将 `state` 更新为不同的值并触发重新渲染。你可以直接传递新状态，也可以传递一个根据先前状态来计算新状态的函数：

```jsx
const [name, setName] = useState('Edward');

function handleClick() {
    setName('Taylor');
    setAge(a => a + 1);
    // ...
}
```

### 参数 
+ `nextState`：你想要 `state` 更新为的值。它可以是任何类型的值，但对于函数有特殊的行为。
+ 如果你将函数作为 `nextState` 传递，它将被视为 更新函数。它必须是纯函数，只接受待定的 `state` 作为其唯一参数，并应返回下一个状态。React 将把你的更新函数放入队列中并重新渲染组件。在下一次渲染期间，React 将通过把队列中所有更新函数应用于先前的状态来计算下一个状态。

### 返回值 
`set` 函数没有返回值。

### 注意事项 

+ `set` 函数 仅更新 下一次 渲染的状态变量。如果在调用 `set` 函数后读取状态变量，则 仍会得到在调用之前显示在屏幕上的旧值。

+ 如果你提供的新值与当前 `state` 相同（由 `Object.is` 比较确定），React 将 跳过重新渲染该组件及其子组件。这是一种优化。虽然在某些情况下 React 仍然需要在跳过子组件之前调用你的组件，但这不应影响你的代码。

+ React 会 批量处理状态更新。它会在所有 事件处理函数运行 并调用其 set 函数后更新屏幕。这可以防止在单个事件期间多次重新渲染。在某些罕见情况下，你需要强制 React 更早地更新屏幕，例如访问 DOM，你可以使用 `flushSync`。

+ 在渲染期间，只允许在当前渲染组件内部调用 `set` 函数。React 将丢弃其输出并立即尝试使用新状态重新渲染。这种方式很少需要，但你可以使用它来存储 先前渲染中的信息。请参见下面的示例。

+ 在严格模式中，React 将 两次调用你的更新函数，以帮助你找到 意外的不纯性。这只是开发时的行为，不影响生产。如果你的更新函数是纯函数（本该是这样），就不应影响该行为。其中一次调用的结果将被忽略。

## 用法

### 为组件添加状态 

在组件的顶层调用 `useState` 来声明一个或多个 状态变量。

```jsx
import { useState } from 'react';

function MyComponent() {
    const [age, setAge] = useState(42);
    const [name, setName] = useState('Taylor');
    // ...
}
```

按照惯例使用 数组解构 来命名状态变量，例如 `[something, setSomething]`。

`useState` 返回一个只包含两个项的数组：

1. 该状态变量 当前的 `state`，最初设置为你提供的 初始化 `state`。
2. `set` 函数，它允许你在响应交互时将 `state` 更改为任何其他值。

要更新屏幕上的内容，请使用新状态调用 `set` 函数：

```jsx
function handleClick() {
  setName('Robin');
}
```

React 会存储新状态，使用新值重新渲染组件，并更新 UI。

#### 陷阱
调用 `set` 函数 不会 改变已经执行的代码中当前的 `state`：

```jsx
function handleClick() {
  setName('Robin');
  console.log(name); // Still "Taylor"!
}
```

它只影响 下一次 渲染中 `useState` 返回的内容。

#### 基本的 useState 示例

```jsx
import { useState } from 'react';

export default function Counter() {
  const [count, setCount] = useState(0);

  function handleClick() {
    setCount(count + 1);
  }

  return (
    <button onClick={handleClick}>
      You pressed me {count} times
    </button>
  );
}
```

### 根据先前的 state 更新 state 

假设 `age` 为 `42`，这个处理函数三次调用 `setAge(age + 1)`：

```jsx
function handleClick() {
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
}
```

为了解决这个问题，你可以向 `setAge` 传递一个 更新函数，而不是下一个状态：

```jsx
function handleClick() {
  setAge(a => a + 1); // setAge(42 => 43)
  setAge(a => a + 1); // setAge(43 => 44)
  setAge(a => a + 1); // setAge(44 => 45)
}
```

这里，`a => a + 1` 是更新函数。它获取 待定状态 并从中计算 下一个状态。

React 将更新函数放入 队列 中。然后，在下一次渲染期间，它将按照相同的顺序调用它们：

1. `a => a + 1` 将接收 `42` 作为待定状态，并返回 `43` 作为下一个状态。
2. `a => a + 1` 将接收 `43` 作为待定状态，并返回 `44` 作为下一个状态。
3. `a => a + 1` 将接收 `44` 作为待定状态，并返回 `45` 作为下一个状态。

现在没有其他排队的更新，因此 React 最终将存储 `45` 作为当前状态。

按照惯例，通常将待定状态参数命名为状态变量名称的第一个字母，如 `age` 为 `a`。然而，你也可以把它命名为 `prevAge` 或者其他你觉得更清楚的名称。

React 在开发环境中可能会 两次调用你的更新函数 来验证其是否为 纯函数。

#### 是否总是优先使用更新函数？ 

你可能会听到这样的建议，如果要设置的状态是根据先前的状态计算得出的，则应始终编写类似于 `setAge(a => a + 1)` 的代码。这样做没有害处，但也不总是必要的。

在大多数情况下，这两种方法没有区别。React 始终确保对于有意的用户操作，如单击，age 状态变量将在下一次单击之前被更新。这意味着单击事件处理函数在事件处理开始没有得到“过时” `age` 的风险。

但是，如果在同一事件中进行多个更新，则更新函数可能会有帮助。如果访问状态变量本身不方便（在优化重新渲染时可能会遇到这种情况），它们也很有用。

如果比起轻微的冗余你更喜欢语法的一致性，你正设置的状态又是根据先前的状态计算出来的，那么总是编写一个更新函数是合理的。如果它是从某个其他状态变量的先前状态计算出的，则你可能希望将它们结合成一个对象然后 使用 `reducer`。

### 传递更新函数和直接传递下一个状态之间的区别

#### 1. 传递更新函数
这个例子传递了更新函数，因此“+3”按钮可以正常工作。

```jsx
import { useState } from 'react';

export default function Counter() {
  const [age, setAge] = useState(42);

  function increment() {
    setAge(a => a + 1);
  }

  return (
    <>
      <h1>Your age: {age}</h1>
      <button onClick={() => {
        increment();
        increment();
        increment();
      }}>+3</button>
      <button onClick={() => {
        increment();
      }}>+1</button>
    </>
  );
}
```

#### 2. 直接传递下一个状态
这个示例 没有 传递更新函数，所以“+3”按钮 不能按预期的方式工作。

```jsx
import { useState } from 'react';

export default function Counter() {
  const [age, setAge] = useState(42);

  function increment() {
    setAge(age + 1);
  }

  return (
    <>
      <h1>Your age: {age}</h1>
      <button onClick={() => {
        increment();
        increment();
        increment();
      }}>+3</button>
      <button onClick={() => {
        increment();
      }}>+1</button>
    </>
  );
}
```

### 更新状态中的对象和数组
你可以将对象和数组放入状态中。在 React 中，状态被认为是只读的，因此 你应该替换它而不是改变现有对象。例如，如果你在状态中保存了一个 `form` 对象，请不要改变它：

```jsx
// 🚩 不要像下面这样改变一个对象：
form.firstName = 'Taylor';
```

相反，可以通过创建一个新对象来替换整个对象：

```jsx
// ✅ 使用新对象替换 state
setForm({
  ...form,
  firstName: 'Taylor'
});
```

阅读有关 更新状态中的对象 和 更新状态中的数组 来了解更多。

### 状态中的对象和数组的示例

#### 第 1 个示例 共 4 个挑战: 表单（对象）
在此示例中，`form` 状态变量保存一个对象。每个输入框都有一个变更处理函数，用整个表单的下一个状态调用 `setForm。{ ...form }` 展开语法确保替换状态对象而不是改变它。

##### App.js

```jsx
import { useState } from 'react';

export default function Form() {
  const [form, setForm] = useState({
    firstName: 'Barbara',
    lastName: 'Hepworth',
    email: 'bhepworth@sculpture.com',
  });

  return (
    <>
      <label>
        First name:
        <input
          value={form.firstName}
          onChange={e => {
            setForm({
              ...form,
              firstName: e.target.value
            });
          }}
        />
      </label>
      <label>
        Last name:
        <input
          value={form.lastName}
          onChange={e => {
            setForm({
              ...form,
              lastName: e.target.value
            });
          }}
        />
      </label>
      <label>
        Email:
        <input
          value={form.email}
          onChange={e => {
            setForm({
              ...form,
              email: e.target.value
            });
          }}
        />
      </label>
      <p>
        {form.firstName}{' '}
        {form.lastName}{' '}
        ({form.email})
      </p>
    </>
  );
}
```

#### 第 2 个示例 共 4 个挑战: 表单（嵌套对象）

在此示例中，状态更为嵌套。当你更新嵌套状态时，你需要复制一份正在更新的对象，以及向上“包含”它的所有对象。阅读 更新嵌套对象 以了解更多。

##### App.js

```jsx
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    name: 'Niki de Saint Phalle',
    artwork: {
      title: 'Blue Nana',
      city: 'Hamburg',
      image: 'https://i.imgur.com/Sd1AgUOm.jpg',
    }
  });

  function handleNameChange(e) {
    setPerson({
      ...person,
      name: e.target.value
    });
  }

  function handleTitleChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        title: e.target.value
      }
    });
  }

  function handleCityChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        city: e.target.value
      }
    });
  }

  function handleImageChange(e) {
    setPerson({
      ...person,
      artwork: {
        ...person.artwork,
        image: e.target.value
      }
    });
  }

  return (
    <>
      <label>
        Name:
        <input
          value={person.name}
          onChange={handleNameChange}
        />
      </label>
      <label>
        Title:
        <input
          value={person.artwork.title}
          onChange={handleTitleChange}
        />
      </label>
      <label>
        City:
        <input
          value={person.artwork.city}
          onChange={handleCityChange}
        />
      </label>
      <label>
        Image:
        <input
          value={person.artwork.image}
          onChange={handleImageChange}
        />
      </label>
      <p>
        <i>{person.artwork.title}</i>
        {' by '}
        {person.name}
        <br />
        (located in {person.artwork.city})
      </p>
      <img 
        src={person.artwork.image} 
        alt={person.artwork.title}
      />
    </>
  );
}
```

#### 第 3 个示例 共 4 个挑战: 列表（数组）
在本例中，`todos` 状态变量保存一个数组。每个按钮的处理函数使用该数组的下一个版本调用 `setTodos。[...todos]` 展开语法，`todos.map()` 和 `todos.filter()` 确保状态数组被替换而不是改变。

##### App.js

```jsx
import { useState } from 'react';
import AddTodo from './AddTodo.js';
import TaskList from './TaskList.js';

let nextId = 3;
const initialTodos = [
  { id: 0, title: 'Buy milk', done: true },
  { id: 1, title: 'Eat tacos', done: false },
  { id: 2, title: 'Brew tea', done: false },
];

export default function TaskApp() {
  const [todos, setTodos] = useState(initialTodos);

  function handleAddTodo(title) {
    setTodos([
      ...todos,
      {
        id: nextId++,
        title: title,
        done: false
      }
    ]);
  }

  function handleChangeTodo(nextTodo) {
    setTodos(todos.map(t => {
      if (t.id === nextTodo.id) {
        return nextTodo;
      } else {
        return t;
      }
    }));
  }

  function handleDeleteTodo(todoId) {
    setTodos(
      todos.filter(t => t.id !== todoId)
    );
  }

  return (
    <>
      <AddTodo
        onAddTodo={handleAddTodo}
      />
      <TaskList
        todos={todos}
        onChangeTodo={handleChangeTodo}
        onDeleteTodo={handleDeleteTodo}
      />
    </>
  );
}
```

##### AddTodo.js

```jsx
import { useState } from 'react';

export default function AddTodo({ onAddTodo }) {
    const [title, setTitle] = useState('');
    return (
        <>
            <input
                placeholder="Add todo"
                value={title}
                onChange={e => setTitle(e.target.value)}
            />
            <button onClick={() => {
                setTitle('');
                onAddTodo(title);
            }}>Add</button>
        </>
    )
}
```

##### TaskList.js

```jsx
import { useState } from 'react';

export default function TaskList({
  todos,
  onChangeTodo,
  onDeleteTodo
}) {
  return (
    <ul>
      {todos.map(todo => (
        <li key={todo.id}>
          <Task
            todo={todo}
            onChange={onChangeTodo}
            onDelete={onDeleteTodo}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ todo, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let todoContent;
  if (isEditing) {
    todoContent = (
      <>
        <input
          value={todo.title}
          onChange={e => {
            onChange({
              ...todo,
              title: e.target.value
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Save
        </button>
      </>
    );
  } else {
    todoContent = (
      <>
        {todo.title}
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
        checked={todo.done}
        onChange={e => {
          onChange({
            ...todo,
            done: e.target.checked
          });
        }}
      />
      {todoContent}
      <button onClick={() => onDelete(todo.id)}>
        Delete
      </button>
    </label>
  );
}
```

#### 用 Immer 编写简洁的更新逻辑

如果不能直接改变数组和对象来进行更新感觉很烦琐，你可以使用像 `Immer` 这样的库来减少重复的代码。`Immer` 可以让你编写简洁的代码，就像你可以直接改变对象一样，但在底层它执行的是不改变的更新：

##### App.js

```jsx
import { useState } from 'react';
import { useImmer } from 'use-immer';

let nextId = 3;
const initialList = [
  { id: 0, title: 'Big Bellies', seen: false },
  { id: 1, title: 'Lunar Landscape', seen: false },
  { id: 2, title: 'Terracotta Army', seen: true },
];

export default function BucketList() {
  const [list, updateList] = useImmer(initialList);

  function handleToggle(artworkId, nextSeen) {
    updateList(draft => {
      const artwork = draft.find(a =>
        a.id === artworkId
      );
      artwork.seen = nextSeen;
    });
  }

  return (
    <>
      <h1>Art Bucket List</h1>
      <h2>My list of art to see:</h2>
      <ItemList
        artworks={list}
        onToggle={handleToggle} />
    </>
  );
}

function ItemList({ artworks, onToggle }) {
  return (
    <ul>
      {artworks.map(artwork => (
        <li key={artwork.id}>
          <label>
            <input
              type="checkbox"
              checked={artwork.seen}
              onChange={e => {
                onToggle(
                  artwork.id,
                  e.target.checked
                );
              }}
            />
            {artwork.title}
          </label>
        </li>
      ))}
    </ul>
  );
}
```

### 避免重复创建初始状态 

React 只在初次渲染时保存初始状态，后续渲染时将其忽略。

```jsx
function TodoList() {
    const [todos, setTodos] = useState(createInitialTodos());
    // ...
}
```

尽管 `createInitialTodos()` 的结果仅用于初始渲染，但你仍然在每次渲染时调用此函数。如果它创建大数组或执行昂贵的计算，这可能会浪费资源。

为了解决这个问题，你可以将它 作为初始化函数传递给 `useState`：

```jsx
function TodoList() {
    const [todos, setTodos] = useState(createInitialTodos);
    // ...
}
```

请注意，你传递的是 `createInitialTodos` 函数本身，而不是 `createInitialTodos()` 调用该函数的结果。如果将函数传递给 `useState`，React 仅在初始化期间调用它。

React 在开发模式下可能会调用你的 初始化函数 两次，以验证它们是否是 纯函数。

### 传递初始化函数和直接传递初始状态之间的区别

#### 传递初始化函数 

这个例子传递了初始化函数，因此 `createInitialTodos` 函数仅在初始化期间运行。当组件重新渲染，例如你在输入框中键入内容时，它不会再次运行。

```jsx
import { useState } from 'react';

function createInitialTodos() {
  const initialTodos = [];
  for (let i = 0; i < 50; i++) {
    initialTodos.push({
      id: i,
      text: 'Item ' + (i + 1)
    });
  }
  return initialTodos;
}

export default function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos);
  const [text, setText] = useState('');

  return (
    <>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        setTodos([{
          id: todos.length,
          text: text
        }, ...todos]);
      }}>Add</button>
      <ul>
        {todos.map(item => (
          <li key={item.id}>
            {item.text}
          </li>
        ))}
      </ul>
    </>
  );
}
```

#### 直接传递初始状态

这个例子 没有 传递初始化函数，因此 `createInitialTodos` 函数会在每次渲染时运行，比如当你在输入框中输入时。这种行为没有什么明显的差异，但这种代码是不那么高效的。

```jsx
import { useState } from 'react';

function createInitialTodos() {
  const initialTodos = [];
  for (let i = 0; i < 50; i++) {
    initialTodos.push({
      id: i,
      text: 'Item ' + (i + 1)
    });
  }
  return initialTodos;
}

export default function TodoList() {
  const [todos, setTodos] = useState(createInitialTodos());
  const [text, setText] = useState('');

  return (
    <>
      <input
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        setTodos([{
          id: todos.length,
          text: text
        }, ...todos]);
      }}>Add</button>
      <ul>
        {todos.map(item => (
          <li key={item.id}>
            {item.text}
          </li>
        ))}
      </ul>
    </>
  );
}
```

### 使用 key 重置状态 

在 渲染列表 时，你经常会遇到 `key` 属性。然而，它还有另外一个用途。

你可以 通过向组件传递不同的 `key` 来重置组件的状态。在这个例子中，重置按钮改变 `version` 状态变量，我们将它作为一个 `key` 传递给 `Form` 组件。当 `key` 改变时，React 会从头开始重新创建 `Form` 组件（以及它的所有子组件），所以它的状态被重置了。

```jsx
import { useState } from 'react';

export default function App() {
  const [version, setVersion] = useState(0);

  function handleReset() {
    setVersion(version + 1);
  }

  return (
    <>
      <button onClick={handleReset}>Reset</button>
      <Form key={version} />
    </>
  );
}

function Form() {
  const [name, setName] = useState('Taylor');

  return (
    <>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <p>Hello, {name}.</p>
    </>
  );
}
```

### 存储前一次渲染的信息 

通常情况下，你会在事件处理函数中更新状态。然而，在极少数情况下，你可能希望在响应渲染时调整状态——例如，当 `props` 改变时，你可能希望改变状态变量。

在大多数情况下，你不需要这样做：

1. 如果你需要的值可以完全从当前 `props` 或其他 `state` 中计算出来，那么 完全可以移除那些多余的状态。如果你担心重新计算的频率过高，可以使用 `useMemo` Hook 来帮助优化。
2. 如果你想重置整个组件树的状态，可以向组件传递一个不同的 `key`。
3. 如果可以的话，在事件处理函数中更新所有相关状态。

在极为罕见的情况下，如果上述方法都不适用，你还可以使用一种方式，在组件渲染时调用 `set` 函数来基于已经渲染的值更新状态。

这里是一个例子。这个 `CountLabel` 组件显示传递给它的 `count` props：

```jsx
export default function CountLabel({ count }) {
  return <h1>{count}</h1>
}
```

假设你想显示计数器是否自上次更改以来 增加或减少。`count` props 无法告诉你这一点——你需要跟踪它的先前值。添加 `prevCount` 状态变量来跟踪它，再添加另一个状态变量 `trend` 来保存计数是否增加或减少。比较 `prevCount` 和 `count`，如果它们不相等，则更新 `prevCount` 和 `trend`。现在你既可以显示当前的 `count` props，也可以显示 自上次渲染以来它如何改变。

##### App.js

```jsx
import { useState } from 'react';
import CountLabel from './CountLabel.js';

export default function App() {
  const [count, setCount] = useState(0);
  return (
    <>
      <button onClick={() => setCount(count + 1)}>
        Increment
      </button>
      <button onClick={() => setCount(count - 1)}>
        Decrement
      </button>
      <CountLabel count={count} />
    </>
  );
}
```

##### CountLabel.js

```jsx
import { useState } from 'react';

export default function CountLabel({ count }) {
  const [prevCount, setPrevCount] = useState(count);
  const [trend, setTrend] = useState(null);
  if (prevCount !== count) {
    setPrevCount(count);
    setTrend(count > prevCount ? 'increasing' : 'decreasing');
  }
  return (
    <>
      <h1>{count}</h1>
      {trend && <p>The count is {trend}</p>}
    </>
  );
}
```

请注意，在渲染时调用 `set` 函数时，它必须位于条件语句中，例如 `prevCount !== count`，并且必须在该条件语句中调用 `setPrevCount(count)`。否则，你的组件将在循环中重新渲染，直到崩溃。此外，你只能像这样更新 当前渲染 组件的状态。在渲染过程中调用 另一个 组件的 `set` 函数是错误的。最后，你的 `set` 调用仍应 不直接改变 `state` 来更新 状态——这并不意味着你可以违反其他 纯函数 的规则。

这种方式可能很难理解，通常最好避免使用。但是，它比在 `effect` 中更新状态要好。当你在渲染期间调用 `set` 函数时，React 将在你的组件使用 `return` 语句退出后立即重新渲染该组件，并在渲染子组件前进行。这样，子组件就不需要进行两次渲染。你的组件函数的其余部分仍会执行（然后结果将被丢弃）。如果你的条件判断在所有 Hook 调用的下方，可以提前添加一个 `return`; 以便更早地重新开始渲染。

## 疑难解答

### 我已经更新了状态，但日志仍显示旧值 

调用 `set` 函数 不能改变运行中代码的状态：

```jsx
function handleClick() {
  console.log(count);  // 0

  setCount(count + 1); // 请求使用 1 重新渲染
  console.log(count);  // 仍然是 0!

  setTimeout(() => {
    console.log(count); // 还是 0!
  }, 5000);
}
```

这是因为 状态表现为就像一个快照。更新状态会使用新的状态值请求另一个渲染，但并不影响在你已经运行的事件处理函数中的 `count` JavaScript 变量。

如果你需要使用下一个状态，你可以在将其传递给 `set` 函数之前将其保存在一个变量中：

```jsx
const nextCount = count + 1;
setCount(nextCount);

console.log(count);     // 0
console.log(nextCount); // 1
```

### 我已经更新了状态，但是屏幕没有更新 
如果下一个状态等于先前的状态，React 将忽略你的更新，这是由 `Object.is` 比较确定的。这通常发生在你直接更改状态中的对象或数组时：

```jsx
obj.x = 10;  // 🚩 错误：直接修改现有的对象
setObj(obj); // 🚩 不会发生任何事情
```

你修改了一个现有的 `obj` 对象并将其传递回 `setObj`，因此 React 忽略了更新。为了解决这个问题，你需要确保始终在状态中 替换 对象和数组，而不是对它们进行 更改：

```jsx
// ✅ 正确：创建一个新对象
setObj({
  ...obj,
  x: 10
});
```

### 出现错误：“Too many re-renders” 

有时可能会出现错误：“Too many re-renders”。React 会限制渲染次数，以防止进入无限循环。通常，这意味着 在渲染期间 无条件地设置状态，因此组件进入循环：渲染、设置状态（导致重新渲染）、渲染、设置状态（导致重新渲染）等等。通常，这是由错误地指定事件处理函数时引起的：

```jsx
// 🚩 错误：在渲染过程中调用事件处理函数
return <button onClick={handleClick()}>Click me</button>

// ✅ 正确：将事件处理函数传递下去
return <button onClick={handleClick}>Click me</button>

// ✅ 正确：传递一个内联函数
return <button onClick={(e) => handleClick(e)}>Click me</button>
```

如果找不到这个错误的原因，请单击控制台中错误旁边的箭头，查看 JavaScript 堆栈以找到导致错误的具体 `set` 函数调用。

### 初始化函数或更新函数运行了两次 

```jsx
function TodoList() {
    // 该函数组件会在每次渲染运行两次。

    const [todos, setTodos] = useState(() => {
        // 该初始化函数在初始化期间会运行两次。
        return createTodos();
    });

    function handleClick() {
        setTodos(prevTodos => {
            // 该更新函数在每次点击中都会运行两次
            return [...prevTodos, createTodo()];
        });
    }

    // ...
}
```

这是所期望的，且不应该破坏你的代码。

这种 仅在开发环境下生效 的行为有助于 保持组件的纯粹性。React 使用其中一个调用的结果，而忽略另一个调用的结果。只要你的组件、初始化函数和更新函数是纯粹的，就不会影响你的逻辑。但是，如果它们意外地不纯粹，这将帮助你注意到错误。

例如，这个不纯的更新函数改变了 `state` 中的一个数组：

```jsx
setTodos(prevTodos => {
  // 🚩 错误：改变 state
  prevTodos.push(createTodo());
});
```

因为 React 调用了两次更新函数，所以你将看到 todo 被添加了两次，所以你将知道出现了错误。在这个例子中，你可以通过 替换数组而不是更改数组 来修复这个错误：

```jsx
setTodos(prevTodos => {
  // ✅ 正确：使用新状态替换
  return [...prevTodos, createTodo()];
});
```

现在，这个更新函数是纯粹的，所以多调用一次不会对行为产生影响。这就是为什么 React 调用它两次可以帮助你找到错误的原因。只有组件、初始化函数和更新函数需要是纯粹的。事件处理函数不需要是纯粹的，所以 React 不会两次调用你的事件处理函数。

### 我尝试将 state 设置为一个函数，但它却被调用了 

你不能像这样把函数放入状态：

```jsx
const [fn, setFn] = useState(someFunction);

function handleClick() {
  setFn(someOtherFunction);
}
```

因为你传递了一个函数，React 认为 `someFunction` 是一个 初始化函数，而 `someOtherFunction` 是一个 更新函数，于是它尝试调用它们并存储结果。要实际 存储 一个函数，你必须在两种情况下在它们之前加上 `() =>`。然后 React 将存储你传递的函数。

```jsx
const [fn, setFn] = useState(() => someFunction);

function handleClick() {
  setFn(() => someOtherFunction);
}
```