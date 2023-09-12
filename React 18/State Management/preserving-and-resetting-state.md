# 对 state 进行保留和重置
各个组件的 state 是各自独立的。根据组件在 UI 树中的位置，React 可以跟踪哪些 state 属于哪个组件。你可以控制在重新渲染过程中何时对 state 进行保留和重置。

## 你将会学习到
+ React 如何“处理”组件结构
+ React 何时选择保留或重置 state
+ 如何强制 React 重置组件的 state
+ key 和组件类型如何影响 state 是否被保留