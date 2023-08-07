# useLayoutEffect

## 陷阱
`useLayoutEffect` 可能会影响性能。尽可能使用 `useEffect`。

`useLayoutEffect` 是 `useEffect` 的一个版本，在浏览器重新绘制屏幕之前触发。

```jsx
useLayoutEffect(setup, dependencies?)
```

## 参考 

```jsx
useLayoutEffect(setup, dependencies?) 
```

调用 `useLayoutEffect` 在浏览器重新绘制屏幕之前进行布局测量：

```jsx
import { useState, useRef, useLayoutEffect } from 'react';

function Tooltip() {
    const ref = useRef(null);
    const [tooltipHeight, setTooltipHeight] = useState(0);

    useLayoutEffect(() => {
        const {height} = ref.current.getBoundingClientRect();
        setTooltipHeight(height);
    }, []);
    // ...
}
```

### 参数 
+ `setup`：处理副作用的函数。`setup` 函数选择性返回一个清理（`cleanup`）函数。在将组件首次添加到 DOM 之前，React 将运行 `setup` 函数。在每次因为依赖项变更而重新渲染后，React 将首先使用旧值运行 `cleanup` 函数（如果你提供了该函数），然后使用新值运行 `setup` 函数。在组件从 DOM 中移除之前，React 将最后一次运行 `cleanup` 函数。
+ 可选 `dependencies`：`setup` 代码中引用的所有响应式值的列表。响应式值包括 `props`、`state` 以及所有直接在组件内部声明的变量和函数。如果你的代码检查工具 配置了 React，那么它将验证每个响应式值都被正确地指定为一个依赖项。依赖项列表必须具有固定数量的项，并且必须像 `[dep1, dep2, dep3]` 这样内联编写。React 将使用 `Object.is` 来比较每个依赖项和它先前的值。如果省略此参数，则在每次重新渲染组件之后，将重新运行副作用函数。

### 返回值
`useLayoutEffect` 返回 `undefined`。

### 注意事项 
+ `useLayoutEffect` 是一个 Hook，因此只能在 组件的顶层 或自己的 Hook 中调用它。不能在循环或者条件内部调用它。如果你需要的话，抽离出一个组件并将副作用处理移动到那里。
+ 当 `StrictMode` 启用时，React 将在真正的 `setup` 函数首次运行前，运行一个额外的开发专有的 `setup` + `cleanup` 周期。这是一个压力测试，确保 `cleanup` 逻辑“映照”到 `setup` 逻辑，并停止或撤消 `setup` 函数正在做的任何事情。如果这导致一个问题，请实现清理函数。
+ Effect 只在客户端上运行，在服务端渲染中不会运行。
+ `useLayoutEffect` 内部的代码和所有计划的状态更新阻塞了浏览器重新绘制屏幕。如果过度使用，这会使你的应用程序变慢。如果可能的话，尽量选择 `useEffect`。

## 用法 

### 在浏览器重新绘制屏幕前计算布局 

大多数组件不需要依靠它们在屏幕上的位置和大小来决定渲染什么。他们只返回一些 JSX，然后浏览器计算他们的 布局（位置和大小）并重新绘制屏幕。

有时候，这还不够。想象一下悬停时出现在某个元素旁边的 tooltip。如果有足够的空间，tooltip 应该出现在元素的上方，但是如果不合适，它应该出现在下面。为了让 tooltip 渲染在最终正确的位置，你需要知道它的高度（即它是否适合放在顶部）。

要做到这一点，你需要分两步渲染：
1. 将 tooltip 渲染到任何地方（即使位置不对）。
2. 测量它的高度并决定放置 tooltip 的位置。
3. 把 tooltip 渲染放在正确的位置。

所有这些都需要在浏览器重新绘制屏幕之前完成。你不希望用户看到 `tooltip` 在移动。调用 `useLayoutEffect` 在浏览器重新绘制屏幕之前执行布局测量：

```jsx
function Tooltip() {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0); // 你还不知道真正的高度

  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setTooltipHeight(height); // 现在重新渲染，你知道了真实的高度
  }, []);

  // ... 在下方的渲染逻辑中使用 tooltipHeight ...
}
```

下面是这如何一步步工作的：
1. `Tooltip` 使用初始值 `tooltipHeight = 0` 进行渲染（因此 `tooltip` 可能被错误地放置）。
2. React 将它放在 DOM 中，然后运行 `useLayoutEffect` 中的代码。
3. `useLayoutEffect` 测量 了 `tooltip` 内容的高度，并立即触发重新渲染。
4. 使用实际的 `tooltipHeight` 再次渲染 `Tooltip`（这样 `tooltip` 的位置就正确了）。
5. React 在 DOM 中对它进行更新，浏览器最终显示出 `tooltip`。

将鼠标悬停在下面的按钮上，看看 `tooltip` 是如何根据它是否合适来调整它的位置：

##### App.js

```jsx
import ButtonWithTooltip from './ButtonWithTooltip.js';

export default function App() {
  return (
    <div>
      <ButtonWithTooltip
        tooltipContent={
          <div>
            This tooltip does not fit above the button.
            <br />
            This is why it's displayed below instead!
          </div>
        }
      >
        Hover over me (tooltip above)
      </ButtonWithTooltip>
      <div style={{ height: 50 }} />
      <ButtonWithTooltip
        tooltipContent={
          <div>This tooltip fits above the button</div>
        }
      >
        Hover over me (tooltip below)
      </ButtonWithTooltip>
      <div style={{ height: 50 }} />
      <ButtonWithTooltip
        tooltipContent={
          <div>This tooltip fits above the button</div>
        }
      >
        Hover over me (tooltip below)
      </ButtonWithTooltip>
    </div>
  );
}
```

##### ButtonWithTooltip.js

```jsx
import { useState, useRef } from 'react';
import Tooltip from './Tooltip.js';

export default function ButtonWithTooltip({ tooltipContent, ...rest }) {
  const [targetRect, setTargetRect] = useState(null);
  const buttonRef = useRef(null);
  return (
    <>
      <button
        {...rest}
        ref={buttonRef}
        onPointerEnter={() => {
          const rect = buttonRef.current.getBoundingClientRect();
          setTargetRect({
            left: rect.left,
            top: rect.top,
            right: rect.right,
            bottom: rect.bottom,
          });
        }}
        onPointerLeave={() => {
          setTargetRect(null);
        }}
      />
      {targetRect !== null && (
        <Tooltip targetRect={targetRect}>
          {tooltipContent}
        </Tooltip>
      )
    }
    </>
  );
}
```

##### Tooltip.js

```jsx
import { useRef, useLayoutEffect, useState } from 'react';
import { createPortal } from 'react-dom';
import TooltipContainer from './TooltipContainer.js';

export default function Tooltip({ children, targetRect }) {
  const ref = useRef(null);
  const [tooltipHeight, setTooltipHeight] = useState(0);

  useLayoutEffect(() => {
    const { height } = ref.current.getBoundingClientRect();
    setTooltipHeight(height);
    console.log('Measured tooltip height: ' + height);
  }, []);

  let tooltipX = 0;
  let tooltipY = 0;
  if (targetRect !== null) {
    tooltipX = targetRect.left;
    tooltipY = targetRect.top - tooltipHeight;
    if (tooltipY < 0) {
      // 它不适合上方，因此把它放在下面。
      tooltipY = targetRect.bottom;
    }
  }

  return createPortal(
    <TooltipContainer x={tooltipX} y={tooltipY} contentRef={ref}>
      {children}
    </TooltipContainer>,
    document.body
  );
}
```

##### TooltipContainer.js

```jsx
export default function TooltipContainer({ children, x, y, contentRef }) {
    return (
        <div
            style={{
                position: 'absolute',
                pointerEvents: 'none',
                left: 0,
                top: 0,
                transform: `translate3d(${x}px, ${y}px, 0)`
            }}
        >
            <div ref={contentRef} className="tooltip">
                {children}
            </div>
        </div>
    );
}
```

注意，即使 `Tooltip` 组件需要两次渲染（首先，使用初始值为 `0` 的 `tooltipHeight` 渲染，然后使用实际测量的高度渲染），你也只能看到最终结果。这就是为什么在这个例子中需要 `useLayoutEffect` 而不是 `useEffect` 的原因。让我们来看看下面的细节差别。