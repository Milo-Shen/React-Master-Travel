# forwardRef

`forwardRef` 允许你的组件使用 `ref` 将一个 DOM 节点暴露给父组件。

## 参考 

```jsx
forwardRef(render) 
```

使用 `forwardRef()` 来让你的组件接收一个 `ref` 并将其传递给一个子组件：

```jsx
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  // ...
});
```

### 参数 
+ `render`：组件的渲染函数。React 会调用该函数并传入父组件传递来的参数和 `ref`。返回的 `JSX` 将成为组件的输出。

### 返回值 
`forwardRef` 返回一个可以在 JSX 中渲染的 React 组件。与作为纯函数定义的 React 组件不同，`forwardRef` 返回的组件还能够接收 `ref` 属性。

### 警告 
在严格模式中，为了 帮助你找到意外的副作用，React 将会 调用两次你的渲染函数。不过这仅限于开发环境，并不会影响生产环境。如果你的渲染函数是纯函数（也应该是），这应该不会影响你的组件逻辑。其中一个调用的结果将被忽略。

### `render` 函数 
`forwardRef` 接受一个渲染函数作为参数。React 将会使用 `props` 和 `ref` 调用此函数：

```jsx
const MyInput = forwardRef(function MyInput(props, ref) {
  return (
    <label>
      {props.label}
      <input ref={ref} />
    </label>
  );
});
```

#### 参数 
+ `props`：父组件传递过来的参数。
+ `ref`：父组件传递的 `ref` 属性。`ref` 可以是一个对象或函数。如果父组件没有传递一个 `ref`，那么它将会是 null。你应该将接收到的 `ref` 转发给另一个组件，或者将其传递给 `useImperativeHandle`。

#### 返回值 
`forwardRef` 返回一个可以在 JSX 中渲染的 React 组件。与作为纯函数定义的 React 组件不同，`forwardRef` 返回的组件还能够接收 `ref` 属性。

## 用法

### 将 DOM 节点暴露给父组件 

默认情况下，每个组件的 DOM 节点都是私有的。然而，有时候将 DOM 节点公开给父组件是很有用的，比如允许对它进行聚焦。如果你想将其公开，可以将组件定义包装在 `forwardRef()` 中：

```jsx
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const { label, ...otherProps } = props;
  return (
    <label>
      {label}
      <input {...otherProps} />
    </label>
  );
});
```

你将在 `props` 之后收到一个 `ref` 作为第二个参数。将其传递到要公开的 DOM 节点中：

```jsx
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const { label, ...otherProps } = props;
  return (
    <label>
      {label}
      <input {...otherProps} ref={ref} />
    </label>
  );
});
```

这样，父级的 `Form` 组件就能够访问 `MyInput` 暴露的 `<input>` DOM 节点：

```jsx
function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
  }

  return (
    <form>
      <MyInput label="Enter your name:" ref={ref} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

该 `Form` 组件 将 `ref` 传递至 `MyInput`。`MyInput` 组件将该 `ref` 转发 至 `<input>` 浏览器标签。因此，`Form` 组件可以访问该 `<input>` DOM 节点并对其调用 `focus()`。

请记住，将组件内部的 `ref` 暴露给 DOM 节点会使得在稍后更改组件内部更加困难。通常，你会将 DOM 节点从可重用的低级组件中暴露出来，例如按钮或文本输入框，但不会在应用程序级别的组件中这样做，例如头像或评论。

### Examples of forwarding a ref

#### 第 1 个示例 共 2 个挑战: 聚焦文本输入框 
点击该按钮将聚焦输入框。`Form` 组件定义了一个 `ref` 并将其传递到 `MyInput` 组件。`MyInput` 组件将该 `ref` 转发至浏览器的 `<input>` 标签，这使得 `Form` 组件可以聚焦该 `<input>`。

##### App.js
```jsx
import { useRef } from 'react';
import MyInput from './MyInput.js';

export default function Form() {
  const ref = useRef(null);

  function handleClick() {
    ref.current.focus();
  }

  return (
    <form>
      <MyInput label="Enter your name:" ref={ref} />
      <button type="button" onClick={handleClick}>
        Edit
      </button>
    </form>
  );
}
```

##### MyInput.js
```
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const { label, ...otherProps } = props;
  return (
    <label>
      {label}
      <input {...otherProps} ref={ref} />
    </label>
  );
});

export default MyInput;
```

#### 第 2 个示例 共 2 个挑战: 播放和暂停视频 
点击按钮将调用 `<video>` DOM 节点上的 `play()` 和 `pause()` 方法。App 组件定义了一个 `ref` 并将其传递到 `MyVideoPlayer` 组件。`MyVideoPlayer` 组件将该 `ref` 转发到浏览器的 `<video>` 标签。这使得 `App` 组件可以播放和暂停 `<video>`。

##### App.js
```jsx
import { useRef } from 'react';
import MyVideoPlayer from './MyVideoPlayer.js';

export default function App() {
  const ref = useRef(null);
  return (
    <>
      <button onClick={() => ref.current.play()}>
        Play
      </button>
      <button onClick={() => ref.current.pause()}>
        Pause
      </button>
      <br />
      <MyVideoPlayer
        ref={ref}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
        type="video/mp4"
        width="250"
      />
    </>
  );
}
```

##### MyVideoPlayer.js
```jsx
import { forwardRef } from 'react';

const VideoPlayer = forwardRef(function VideoPlayer({ src, type, width }, ref) {
  return (
    <video width={width} ref={ref}>
      <source
        src={src}
        type={type}
      />
    </video>
  );
});

export default VideoPlayer;
```