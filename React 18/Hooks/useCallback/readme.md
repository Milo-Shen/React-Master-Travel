# useCallback
`useCallback` 是一个允许你在多次渲染中缓存函数的 React Hook

```jsx
const cachedFn = useCallback(fn, dependencies);
```

## 参考 
### `useCallback(fn, dependencies)`
在组件顶层调用 useCallback 以便在多次渲染中缓存函数：

```jsx
import { useCallback } from 'react';

export default function ProductPage({ productId, referrer, theme }) {
    const handleSubmit = useCallback((orderDetails) => {
        post('/product/' + productId + '/buy', {
            referrer,
            orderDetails,
        });
    }, [productId, referrer]);
}
```

### 参数 
+ `fn`：想要缓存的函数。此函数可以接受任何参数并且返回任何值。React 将会在初次渲染而非调用时返回该函数。当进行下一次渲染时，如果 `dependencies` 相比于上一次渲染时没有改变，那么 React 将会返回相同的函数。否则，React 将返回在最新一次渲染中传入的函数，并且将其缓存以便之后使用。React 不会调用此函数，而是返回此函数。你可以自己决定何时调用以及是否调用。
+ `~~dependencies~~`：有关是否更新 `fn` 的所有响应式值的一个列表。响应式值包括 props、state，和所有在你组件内部直接声明的变量和函数。如果你的代码检查工具 配置了 React，那么它将校验每一个正确指定为依赖的响应式值。依赖列表必须具有确切数量的项，并且必须像 `[dep1, dep2, dep3]` 这样编写。React 使用 `Object.is` 比较每一个依赖和它的之前的值。

### 返回值 
在初次渲染时，`useCallback` 返回你已经传入的 fn 函数
在之后的渲染中, 如果依赖没有改变，`useCallback` 返回上一次渲染中缓存的 `fn` 函数；否则返回这一次渲染传入的 `fn`。

### 注意 
+ `useCallback` 是一个 Hook，所以应该在 组件的顶层 或自定义 Hook 中调用。你不应在循环或者条件语句中调用它。如果你需要这样做，请新建一个组件，并将 `state` 移入其中。
+ 除非有特定的理由，React 将不会丢弃已缓存的函数。例如，在开发中，当编辑组件文件时，React 会丢弃缓存。在生产和开发环境中，如果你的组件在初次挂载中暂停，React 将会丢弃缓存。在未来，React 可能会增加更多利用了丢弃缓存机制的特性。例如，如果 React 未来内置了对虚拟列表的支持，那么在滚动超出虚拟化表视口的项目时，抛弃缓存是有意义的。如果你依赖 `useCallback` 作为一个性能优化途径，那么这些对你会有帮助。否则请考虑使用 `state` 变量 或 `ref`。

## 用法 
### 跳过组件的重新渲染 
当你优化渲染性能的时候，有时需要缓存传递给子组件的函数。让我们先关注一下如何实现，稍后去理解在哪些场景中它是有用的。
为了缓存组件中多次渲染的函数，你需要将其定义在 `useCallback` Hook 中：

```jsx
import { useCallback } from 'react';

function ProductPage({ productId, referrer, theme }) {
    const handleSubmit = useCallback((orderDetails) => {
        post('/product/' + productId + '/buy', {
            referrer,
            orderDetails,
        });
    }, [productId, referrer]);
    // ...
}
```

你需要传递两个参数给 `useCallback`：
1. 在多次渲染中需要缓存的函数
2. 函数内部需要使用到的所有组件内部值的 依赖列表

初次渲染时，在 `useCallback` 处接收的 返回函数 将会是已经传入的函数。
在之后的渲染中，React 将会使用 `Object.is` 把 当前的依赖 和已传入之前的依赖进行比较。如果没有任何依赖改变，`useCallback` 将会返回与之前一样的函数。否则 `useCallback` 将返回 *此次* 渲染中传递的函数。
简而言之，`useCallback` 在多次渲染中缓存一个函数，直至这个函数的依赖发生改变。

#### 让我们通过一个示例看看它何时有用。
假设你正在从 `ProductPage` 传递一个 `handleSubmit` 函数到 `ShippingForm` 组件中：

```jsx
function ProductPage({ productId, referrer, theme }) {
    // ...
    return (
        <div className={theme}>
            <ShippingForm onSubmit={handleSubmit}/>
        </div>
    );
}
```

注意，切换 `theme` props 后会让应用停滞一小会，但如果将 `<ShippingForm />` 从 JSX 中移除，应用将反应迅速。这就提示尽力优化 `ShippingForm` 组件将会很有用。
默认情况下，当一个组件重新渲染时， React 将递归渲染它的所有子组件，因此每当因 `theme` 更改时而 `ProductPage` 组件重新渲染时，`ShippingForm` 组件也会重新渲染。这对于不需要大量计算去重新渲染的组件来说影响很小。但如果你发现某次重新渲染很慢，你可以将 `ShippingForm` 组件包裹在 `memo` 中。如果 props 和上一次渲染时相同，那么 `ShippingForm` 组件将跳过重新渲染。

```jsx
import { memo } from 'react';

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  // ...
});
```

当代码像上面一样改变后，如果 props 与上一次渲染时相同，`ShippingForm` 将跳过重新渲染。这时缓存函数就变得很重要。假设定义了 `handleSubmit` 而没有定义 `useCallback`：

```jsx
function ProductPage({ productId, referrer, theme }) {
  // 每当 theme 改变时，都会生成一个不同的函数
  function handleSubmit(orderDetails) {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }
  
  return (
    <div className={theme}>
      {/* 这将导致 ShippingForm props 永远都不会是相同的，并且每次它都会重新渲染 */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

与字面量对象 `{}` 总是会创建新对象类似，在 JavaScript 中，`function () {}` 或者 `() => {}` 总是会生成不同的函数。正常情况下，这不会有问题，但是这意味着 `ShippingForm` props 将永远不会是相同的，并且 `memo` 对性能的优化永远不会生效。而这就是 `useCallback` 起作用的地方

```jsx
function ProductPage({ productId, referrer, theme }) {
  // 在多次渲染中缓存函数
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]); // 只要这些依赖没有改变

  return (
    <div className={theme}>
      {/* ShippingForm 就会收到同样的 props 并且跳过重新渲染 */}
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}
```

将 `handleSubmit` 传递给 `useCallback` 就可以确保它在多次重新渲染之间是相同的函数，直到依赖发生改变。注意，除非出于某种特定原因，否则不必将一个函数包裹在 `useCallback` 中。在本例中，你将它传递到了包裹在 `memo` 中的组件，这允许它跳过重新渲染。不过还有其他场景可能需要用到 `useCallback`，本章将对此进行进一步描述。
`useCallback` 只应作用于性能优化。如果代码在没有它的情况下无法运行，请找到根本问题并首先修复它，然后再使用 `useCallback`。

### useCallback 与 useMemo 有何关系？ 
`useMemo` 经常与 `useCallback` 一同出现。当尝试优化子组件时，它们都很有用。他们会 记住（或者说，缓存）正在传递的东西：

```jsx
import { useMemo, useCallback } from 'react';

function ProductPage({ productId, referrer }) {
  const product = useData('/product/' + productId);

  // 调用函数并缓存结果
  const requirements = useMemo(() => { 
    return computeRequirements(product);
  }, [product]);

  // 缓存函数本身
  const handleSubmit = useCallback((orderDetails) => { 
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);

  return (
    <div className={theme}>
      <ShippingForm requirements={requirements} onSubmit={handleSubmit} />
    </div>
  );
}
```

#### 区别在于你需要缓存什么:
+ *`useMemo` 缓存函数调用的结果。* 在这里，它缓存了调用 `computeRequirements(product)` 的结果。除非 `product` 发生改变，否则它将不会发生变化。这让你向下传递 `requirements` 时而无需不必要地重新渲染 `ShippingForm`。必要时，React 将会调用传入的函数重新计算结果。
+ *`useCallback` 缓存函数本身。* 不像 useMemo，它不会调用你传入的函数。相反，它缓存此函数。从而除非 productId 或 referrer 发生改变，handleSubmit 自己将不会发生改变。这让你向下传递 handleSubmit 函数而无需不必要地重新渲染 ShippingForm。直至用户提交表单，你的代码都将不会运行。

如果你已经熟悉了 `useMemo`，你可能发现将 `useCallback` 视为以下内容会很有帮助：

```jsx
// 在 React 内部的简化实现
function useCallback(fn, dependencies) {
  return useMemo(() => fn, dependencies);
}
```

### 是否应该在任何地方添加 useCallback？
如果你的应用程序与本网站类似，并且大多数交互都很粗糙（例如替换页面或整个部分），则通常不需要缓存。另一方面，如果你的应用更像是一个绘图编辑器，并且大多数交互都是精细的（如移动形状），那么你可能会发现缓存非常有用
使用 `useCallback` 缓存函数仅在少数情况下有意义：

+ 将其作为 props 传递给包装在 `[memo]` 中的组件。如果 props 未更改，则希望跳过重新渲染。缓存允许组件仅在依赖项更改时重新渲染。
+ 传递的函数可能作为某些 Hook 的依赖。比如，另一个包裹在 `useCallback` 中的函数依赖于它，或者依赖于 `useEffect` 中的函数。

在其他情况下，将函数包装在 `useCallback` 中没有任何意义。不过即使这样做了，也没有很大的坏处。所以有些团队选择不考虑个案，从而尽可能缓存。不好的地方可能是降低了代码可读性。而且，并不是所有的缓存都是有效的：一个始终是新的值足以破坏整个组件的缓存。

请注意，`useCallback` 不会阻止创建函数。你总是在创建一个函数（这很好！），但是如果没有任何东西改变，React 会忽略它并返回缓存的函数。

#### 在实践中, 你可以通过遵循一些原则来减少许多不必要的记忆化：
1. 当一个组件在视觉上包装其他组件时，让它 接受 JSX 作为子元素。随后，如果包装组件更新自己的 state，React 知道它的子组件不需要重新渲染。
2. 建议使用 state 并且不要 *提升状态* 超过必要的程度。不要将表单和项是否悬停等短暂状态保存在树的顶部或全局状态库中。
3. 保持 *渲染逻辑纯粹*。如果重新渲染组件会导致问题或产生一些明显的视觉瑕疵，那么这是组件自身的问题！请修复这个错误，而不是添加记忆化。
4. 避免 *不必要地更新 Effect。* React 应用程序中的大多数性能问题都是由 Effect 的更新链引起的，这些更新链不断导致组件重新渲染。
5. 尝试 *从 Effect 中删除不必要的依赖关系。* 例如，将某些对象或函数移动到副作用内部或组件外部通常更简单，而不是使用记忆化。

### 使用 useCallback 与直接声明函数的区别
#### 第 1 个示例 共 2 个挑战: 使用 useCallback 和 memo 跳过函数的重新渲染
在这个例子中，`ShippingForm` 组件被人为地减慢了速度，以便你可以看到渲染的 React 组件真正变慢时会发生什么。尝试递增计数器并切换主题。

递增计数器感觉很慢，因为它会强制变慢 `ShippingForm` 的重新渲染。这是意料之中的，因为计数器已更改，因此你需要在屏幕上反映用户的新选择。  

接下来，尝试更改主题。将 `useCallback` 和 `memo` 结合使用后，尽管人为减缓了速度，但它还是很快。由于 `useCallback` 依赖 `productId` 与 `referrer` 自上次渲染后始终没有发生改变，因此 `handleSubmit` 也没有改变。由于 `handleSubmit` 没有发生改变，`ShippingForm` 就跳过了重新渲染。

##### App.js
```jsx
import { useState } from 'react';
import ProductPage from './ProductPage.js';

export default function App() {
  const [isDark, setIsDark] = useState(false);
  return (
    <>
      <label>
        <input
          type="checkbox"
          checked={isDark}
          onChange={e => setIsDark(e.target.checked)}
        />
        Dark mode
      </label>
      <hr />
      <ProductPage
        referrerId="wizard_of_oz"
        productId={123}
        theme={isDark ? 'dark' : 'light'}
      />
    </>
  );
}
```

##### ProductPage.js
```jsx
import { useCallback } from 'react';
import ShippingForm from './ShippingForm.js';

export default function ProductPage({ productId, referrer, theme }) {
  const handleSubmit = useCallback((orderDetails) => {
    post('/product/' + productId + '/buy', {
      referrer,
      orderDetails,
    });
  }, [productId, referrer]);

  return (
    <div className={theme}>
      <ShippingForm onSubmit={handleSubmit} />
    </div>
  );
}

function post(url, data) {
  // 想象这发送了一个请求
  console.log('POST /' + url);
  console.log(data);
}
```

##### ShippingForm.js
```jsx
import { memo, useState } from 'react';

const ShippingForm = memo(function ShippingForm({ onSubmit }) {
  const [count, setCount] = useState(1);

  console.log('[ARTIFICIALLY SLOW] Rendering <ShippingForm />');
  
  let startTime = performance.now();
  while (performance.now() - startTime < 500) {
    // 500 毫秒内不执行任何操作来模拟极慢的代码
  }

  function handleSubmit(e) {
    e.preventDefault();
    const formData = new FormData(e.target);
    
    const orderDetails = {
      ...Object.fromEntries(formData),
      count
    };
    
    onSubmit(orderDetails);
  }

  return (
    <form onSubmit={handleSubmit}>
      <p><b>Note: <code>ShippingForm</code> is artificially slowed down!</b></p>
      <label>
        Number of items:
        <button type="button" onClick={() => setCount(count - 1)}>–</button>
        {count}
        <button type="button" onClick={() => setCount(count + 1)}>+</button>
      </label>
      <label>
        Street:
        <input name="street" />
      </label>
      <label>
        City:
        <input name="city" />
      </label>
      <label>
        Postal code:
        <input name="zipCode" />
      </label>
      <button type="submit">Submit</button>
    </form>
  );
});

export default ShippingForm;
```