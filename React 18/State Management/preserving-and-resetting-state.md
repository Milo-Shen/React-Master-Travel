# 对 state 进行保留和重置
各个组件的 state 是各自独立的。根据组件在 UI 树中的位置，React 可以跟踪哪些 state 属于哪个组件。你可以控制在重新渲染过程中何时对 state 进行保留和重置。

## 你将会学习到
+ React 如何“处理”组件结构
+ React 何时选择保留或重置 state
+ 如何强制 React 重置组件的 state
+ key 和组件类型如何影响 state 是否被保留

## UI 树 
浏览器使用许多树形结构来为 UI 建立模型。DOM 用于表示 HTML 元素，CSSOM 则表示 CSS 元素。甚至还有 Accessibility tree！

React 也使用树形结构来对你创造的 UI 进行管理和建模。React 根据你的 JSX 生成 UI 树。React DOM 根据 UI 树去更新浏览器的 DOM 元素。（React Native 则将这些 UI 树转译成移动平台上特有的元素。）

