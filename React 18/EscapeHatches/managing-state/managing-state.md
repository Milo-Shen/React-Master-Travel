# 状态管理

随着你的应用不断变大，更有意识的去关注应用状态如何组织，以及数据如何在组件之间流动会对你很有帮助。冗余或重复的状态往往是缺陷的根源。在本节中，你将学习如何组织好状态，如何保持状态更新逻辑的可维护性，以及如何跨组件共享状态。

## 使用状态响应输入 
使用 React，你不用直接从代码层面修改 UI。例如，不用编写诸如“禁用按钮”、“启用按钮”、“显示成功消息”等命令。相反，你只需要描述组件在不同状态（“初始状态”、“输入状态”、“成功状态”）下希望展现的 UI，然后根据用户输入触发状态更改。这和设计师对 UI 的理解很相似。

下面是一个使用 React 编写的反馈表单。请注意看它是如何使用 status 这个状态变量来决定启用或禁用提交按钮，以及是否显示成功消息的。

```jsx
import { useState } from 'react';

export default function Form() {
  const [answer, setAnswer] = useState('');
  const [error, setError] = useState(null);
  const [status, setStatus] = useState('typing');

  if (status === 'success') {
    return <h1>答对了！</h1>
  }

  async function handleSubmit(e) {
    e.preventDefault();
    setStatus('submitting');
    try {
      await submitForm(answer);
      setStatus('success');
    } catch (err) {
      setStatus('typing');
      setError(err);
    }
  }

  function handleTextareaChange(e) {
    setAnswer(e.target.value);
  }

  return (
    <>
      <h2>城市测验</h2>
      <p>
        哪个城市有把空气变成饮用水的广告牌？
      </p>
      <form onSubmit={handleSubmit}>
        <textarea
          value={answer}
          onChange={handleTextareaChange}
          disabled={status === 'submitting'}
        />
        <br />
        <button disabled={
          answer.length === 0 ||
          status === 'submitting'
        }>
          提交
        </button>
        {error !== null &&
          <p className="Error">
            {error.message}
          </p>
        }
      </form>
    </>
  );
}

function submitForm(answer) {
  // 模拟接口请求
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      let shouldError = answer.toLowerCase() !== 'lima'
      if (shouldError) {
        reject(new Error('猜的不错，但答案不对。再试试看吧！'));
      } else {
        resolve();
      }
    }, 1500);
  });
}
```

## 选择状态结构 
良好的状态组织，可以区分开易于修改和调试的组件与频繁出问题的组件。最重要的原则是，状态不应包含冗余或重复的信息。如果包含一些多余的状态，我们会很容易忘记去更新它，从而导致问题产生！

例如，这个表单有一个多余的 `fullName` 状态变量：

```jsx
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');
  const [fullName, setFullName] = useState('');

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
    setFullName(e.target.value + ' ' + lastName);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
    setFullName(firstName + ' ' + e.target.value);
  }

  return (
    <>
      <h2>让我们帮你登记</h2>
      <label>
        名：{' '}
        <input
          value={firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        姓：{' '}
        <input
          value={lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <p>
        你的票据将签发给：<b>{fullName}</b>
      </p>
    </>
  );
}
```

你可以移除它并在组件渲染时通过计算 `fullName` 来简化代码：

```jsx
import { useState } from 'react';

export default function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  const fullName = firstName + ' ' + lastName;

  function handleFirstNameChange(e) {
    setFirstName(e.target.value);
  }

  function handleLastNameChange(e) {
    setLastName(e.target.value);
  }

  return (
    <>
      <h2>让我们帮你登记</h2>
      <label>
        名：{' '}
        <input
          value={firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        姓：{' '}
        <input
          value={lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <p>
        你的票将发给：<b>{fullName}</b>
      </p>
    </>
  );
}
```

这看起来似乎只是一个小改动，但却可以避免很多潜在的问题。

## 在组件间共享状态 
有时候你希望两个组件的状态始终同步更改。要实现这一点，可以将相关状态从这两个组件上移除，并把这些状态移到最近的父级组件，然后通过 props 将状态传递给这两个组件。这被称为“状态提升”，这是编写 React 代码时常做的事。

在以下示例中，要求每次只能激活一个面板。要实现这一点，父组件将管理激活状态并为其子组件指定 prop，而不是将激活状态保留在各自的子组件中。

```jsx
import { useState } from 'react';

export default function Accordion() {
  const [activeIndex, setActiveIndex] = useState(0);
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel
        title="关于"
        isActive={activeIndex === 0}
        onShow={() => setActiveIndex(0)}
      >
        阿拉木图人口约200万，是哈萨克斯坦最大的城市。在1929年至1997年之间，它是该国首都。
      </Panel>
      <Panel
        title="词源"
        isActive={activeIndex === 1}
        onShow={() => setActiveIndex(1)}
      >
        这个名字源于哈萨克语 <span lang="kk-KZ">алма</span>，是“苹果”的意思，通常被翻译成“满是苹果”。事实上，阿拉木图周围的地区被认为是苹果的祖籍，<i lang="la">Malus sieversii</i> 被认为是目前本土苹果的祖先。
      </Panel>
    </>
  );
}

function Panel({
  title,
  children,
  isActive,
  onShow
}) {
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={onShow}>
          显示
        </button>
      )}
    </section>
  );
}
```

## 保留和重置状态
当你重新渲染一个组件时， React 需要决定组件树中的哪些部分要保留和更新，以及丢弃或重新创建。在大多数情况下， React 的自动处理机制已经做得足够好了。默认情况下，React 会保留树中与先前渲染的组件树“匹配”的部分。

然而，有时这并不是你想要的。例如，在下面这个程序中，输入内容后再切换收件人并不会清空输入框。这可能会导致用户不小心发错消息：

### App.js
```jsx
import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat contact={to} />
    </div>
  )
}

const contacts = [
  { name: 'Taylor', email: 'taylor@mail.com' },
  { name: 'Alice', email: 'alice@mail.com' },
  { name: 'Bob', email: 'bob@mail.com' }
];
```

### ContactList.js
```jsx
export default function ContactList({
  selectedContact,
  contacts,
  onSelect
}) {
  return (
    <section className="contact-list">
      <ul>
        {contacts.map(contact =>
          <li key={contact.email}>
            <button onClick={() => {
              onSelect(contact);
            }}>
              {contact.name}
            </button>
          </li>
        )}
      </ul>
    </section>
  );
}
```

### Chat.js
```jsx
import { useState } from 'react';

export default function Chat({ contact }) {
  const [text, setText] = useState('');
  return (
    <section className="chat">
      <textarea
        value={text}
        placeholder={'Chat to ' + contact.name}
        onChange={e => setText(e.target.value)}
      />
      <br />
      <button>发送给 {contact.email}</button>
    </section>
  );
}
```

React 允许你覆盖默认行为，可通过向组件传递一个唯一 `key`（如 `<Chat key={email}/>` 来 强制 重置其状态。这会告诉 React ，如果收件人不同，应将其作为一个 不同的 `Chat` 组件，需要使用新数据和 UI（比如输入框）来重新创建它。现在，在接收者之间切换时就会重置输入框——即使渲染的是同一个组件。

### App.js
```jsx
import { useState } from 'react';
import Chat from './Chat.js';
import ContactList from './ContactList.js';

export default function Messenger() {
  const [to, setTo] = useState(contacts[0]);
  return (
    <div>
      <ContactList
        contacts={contacts}
        selectedContact={to}
        onSelect={contact => setTo(contact)}
      />
      <Chat key={to.email} contact={to} />
    </div>
  )
}

const contacts = [
  { name: 'Taylor', email: 'taylor@mail.com' },
  { name: 'Alice', email: 'alice@mail.com' },
  { name: 'Bob', email: 'bob@mail.com' }
];
```