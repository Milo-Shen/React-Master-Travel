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