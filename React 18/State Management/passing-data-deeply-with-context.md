# 使用 Context 深层传递参数
通常来说，你会通过 props 将信息从父组件传递到子组件。但是，如果你必须通过许多中间组件向下传递 props，或是在你应用中的许多组件需要相同的信息，传递 props 会变的十分冗长和不便。Context 允许父组件向其下层无论多深的任何组件提供信息，而无需通过 props 显式传递。

## 你将会学习到
+ 什么是 “prop 逐级透传”
+ 如何使用 context 代替重复的参数传递
+ Context 的常见用法
+ Context 的常见替代方案

## 传递 props 带来的问题 
传递 props 是将数据通过 UI 树显式传递到使用它的组件的好方法。

但是当你需要在组件树中深层传递参数以及需要在组件间复用相同的参数时，传递 props 就会变得很麻烦。最近的根节点父组件可能离需要数据的组件很远，状态提升 到太高的层级会导致 “逐层传递 props” 的情况。

要是有一种方法可以在组件树中不需要 props 将数据“直达”到所需的组件中，那可就太好了。React 的 context 功能可以满足我们的这个心愿。

## Context：传递 props 的另一种方法 
Context 让父组件可以为它下面的整个组件树提供数据。Context 有很多种用途。这里就有一个示例。思考一下这个 `Heading` 组件接收一个 `level` 参数来决定它标题尺寸的场景

### App.js
```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>主标题</Heading>
      <Heading level={2}>副标题</Heading>
      <Heading level={3}>子标题</Heading>
      <Heading level={4}>子子标题</Heading>
      <Heading level={5}>子子子标题</Heading>
      <Heading level={6}>子子子子标题</Heading>
    </Section>
  );
}
```

### Section.js
```jsx
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

### Heading.js
```jsx
export default function Heading({ level, children }) {
  switch (level) {
    case 1:
      return <h1>{children}</h1>;
    case 2:
      return <h2>{children}</h2>;
    case 3:
      return <h3>{children}</h3>;
    case 4:
      return <h4>{children}</h4>;
    case 5:
      return <h5>{children}</h5>;
    case 6:
      return <h6>{children}</h6>;
    default:
      throw Error('未知的 level：' + level);
  }
}
```

假设你想让相同 `Section` 中的多个 Heading 具有相同的尺寸：

```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading level={1}>主标题</Heading>
      <Section>
        <Heading level={2}>副标题</Heading>
        <Heading level={2}>副标题</Heading>
        <Heading level={2}>副标题</Heading>
        <Section>
          <Heading level={3}>子标题</Heading>
          <Heading level={3}>子标题</Heading>
          <Heading level={3}>子标题</Heading>
          <Section>
            <Heading level={4}>子子标题</Heading>
            <Heading level={4}>子子标题</Heading>
            <Heading level={4}>子子标题</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

目前，你将 `level` 参数分别传递给每个 `<Heading>`：

```jsx
<Section>
  <Heading level={3}>关于</Heading>
  <Heading level={3}>照片</Heading>
  <Heading level={3}>视频</Heading>
</Section>
```

将 `level` 参数传递给 `<Section>` 组件而不是传给 `<Heading>` 组件看起来更好一些。这样的话你可以强制使同一个 section 中的所有标题都有相同的尺寸：

```jsx
<Section level={3}>
  <Heading>关于</Heading>
  <Heading>照片</Heading>
  <Heading>视频</Heading>
</Section>
```

但是 `<Heading>` 组件是如何知道离它最近的 `<Section>` 的 `level` 的呢？*这需要子组件可以通过某种方式“访问”到组件树中某处在其上层的数据。*

你不能只通过 props 来实现它。这就是 context 大显身手的地方。你可以通过以下三个步骤来实现它：

1. 创建 一个 `context`。（你可以将其命名为 `LevelContext`, 因为它表示的是标题级别。)
2. 在需要数据的组件内 使用 刚刚创建的 `context`。（`Heading` 将会使用 `LevelContext`。）
3. 在指定数据的组件中 提供 这个 `context`。 （`Section` 将会提供 `LevelContext`。）

Context 可以让父节点，甚至是很远的父节点都可以为其内部的整个组件树提供数据。