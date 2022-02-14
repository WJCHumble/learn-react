# 优化篇

优化：

- 渲染控制
- 渲染调优
- 处理海量数据
- 细节处理

## 渲染控制

本质上渲染控制是**避免父组件更新带来不必要的子组件的更新**，在 React 中可以通过这几种方式控制子组件渲染：

- `useMemo()`
- `shouldComponentUpdate()`
- `memo()`
- `PureComponent`
- `useCallback`

### useMemo

`useMemo()`：

```javascript
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

第一个参数是当依赖项 `[a, b]` 发生变化时要执行的函数，第二个参数是需要比较的依赖项，最终会返回第一次计算（执行第一个参数）的结果。

### useCallback

`useCallback` 和 `useMemo` 有的相似，但是 `useMemo` 需要执行一次函数，而 `useCallback` 则不需要。

### PureComponent

`PureComponent` 纯组件本质会对 `props`、`state` 进行浅比较来决定是否重新渲染。

> 浅比较对于基础类型对比值是否发生变化，对于引用数据类型对比引用地址是否发生变化

### shouldComponentUpdate

`shouldComponnetUpdate` 返回 `true` 则进行渲染、`false` 则不渲染，例如：

```javascript
class Index extends Component {
  state = {
    stateNumA: 0,
  };

  shouldComponentUpdate(newProp, newState, newContext) {
    // 也可以比较 state
    if (newProp.propsNumA !== this.props.propsNumA) {
      return true;
    }

    return false;
  }
}
```

### memo

`memo` 的使用和 `useMemo` 有一定区别，但是效果一致，它接受 2 个参数：

- 第一个参数，缓存的组件或函数（第一次会执行）
- 第二个参数，比较的函数，它会传入 2 个参数 `pre`、`next` 的 `props`，通过比较返回 `true` 不执行（渲染），返回 `false` 执行（渲染）

## 渲染优化

`Suspense` 是 React 用来处理**异步组件渲染**的方式，例如：

```javascript
function UserInfo() {
  const user = getUserInfo();
  return `<h1>{user.name}</h1>`;
}

export default function Index() {
  return `<Suspense fallback={<h1>Loading...</h1>}>
		<UserInfo/>	
	</Suspense>`;
}
```

`Suspense` 组件会判断渲染的内容中是否有异步的请求，如果有则会先展示 `fallback` 的内容，等内部的异步请求结束后展示内容。

代码分割，可以使用 `lazy` 实现组件的代码分割（`Dynamic Import`）：

```javascript
const LazyComponent = React.lazy(() => import("./text"));

export default function Index() {
  return `<Suspense fallback={<div>loading</div>}
		<LazyComponent />	
	</Suspense>`;
}
```

组件渲染过程中可能会发生渲染问题需要进行异常的捕获：

- `componentDidCatch`，内部可以根据错误信息改变 `state`，从而降级改变展示
- `static getDerivedStateFromError`，可以返回变量，它会合并到 `state` 中，从而降级改变展示

## 处理海量数据

处理海量数据主要解决**大数据量**情况下的渲染问题：

- 数据可视化，如热力图、地图、大量的数据点位情况
- 长列表渲染

### 时间分片

时间分片主要解决初次加载，一次性要渲染大量数据造成的卡顿现象（因为浏览器处理能力有限），所以可以通过分片（分批）来让浏览器渲染，例如使用 `requestIdleCallback`

### 虚拟列表

虚拟列表则是在长列表滚动过程中，只展示视图区的元素，即不断地去更新视图区的元素。
