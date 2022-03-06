## learn react

参考：

- [一名 vueCoder 总结的 React 基础](https://juejin.cn/post/6960556335092269063#heading-40)
- [我在大厂写 React 学到了什么？性能优化篇](https://mp.weixin.qq.com/s/lkYoiPorc6TQ19zPK_W-1A)
- [useEffect 使用指南](https://mp.wexin.qq.com/s/XN82E1jt1VjJK2eLwPRm4A)
- [Dan Abramov 访谈实录](https://mp.weixin.qq.com/s/SBVE34dW9g4BsabmLJV9wg)
- [「React 进阶」 学好这些 React 设计模式，能让你的 React 项目飞起来](https://mp.weixin.qq.com/s/dfnajqS0NqTkp7fzK4-4Sw)
- [React Hook 高级用法](https://mp.weixin.qq.com/s/8gbPY2YooCeF-3XgKcG21w)

只用函数组件 + Hook

## useState 状态

```javascript
function App() {
  const [count, setCount] = useState(0);

  const handleBtnClick = () => {
    setCount((preCount) => preCount + 1);
  };

  return (
    <div>
      <button onClick={handleBtnClick}></button>
      <p>{count}</p>
    </div>
  );
}
```

注意，如果 `useState` 的 `dispatchEvent` 不是函数的时候，如果重复传入一样的 `state`（引用地址相同），则不会触发数据（视图更新），例如：

```javascript
function App() {
  const [user, setUser] = useState({ name: "wjc" });

  const handleBtnClick = () => {
    user.name = "zs";
    // 在 useState 的 dispatchAction 处理逻辑中，会浅比较两次 state，由于引用地址相同，不会触发更新
    setUser(user);
    // 正确写法，拷贝一份 user
    // setUser({...user})
  };

  return (
    <div>
      <button onClick={handleBtnClick}></button>
      <p>{count}</p>
    </div>
  );
}
```

### useEffect 监听 State 变化

```javascript
function App() {
  const [count, setCount] = useState(0);

  const handleBtnClick = () => {
    setCount((preCount) => preCount + 1);
  };

  useEffect(() => {
    console.log(count);
  }, [count]);

  return (
    <div>
      <button onClick={handleBtnClick}></button>
      <p>{count}</p>
    </div>
  );
}
```

使用 `ReactDOM.flushSync()` 可以优先执行更新，比默认的批量更新更快

注意，在更新事件中是拿不到数据改变后的值的，例如：

```javascript
function App() {
  const [count, setCount] = useState(0);

  const handleBtnClick = () => {
    ReactDom.flushSync(() => {
      setCount(2);
      console.log(number);
    });

    setCount(1);
    console.log(number);
    setTimeout(() => {
      setCount(3);
      console.log(number);
    });
  };

  return (
    <div>
      <button onClick={handleBtnClick}></button>
      <p>{count}</p>
    </div>
  );
}
// 最后输出 0 0 0
```

因为，在函数组件的数据更新就是函数的执行，这个时候所有变量都会重新声明，而更新又是在类似下一个 tick 中更新的，直接用同步方式都是初始化的值

## Props

基本使用：

```javascript
function child() {
  const { count } = props;

  return <div>{{ count }}</div>;
}

function parent() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setCount((preCount) => preCount + 1);
  };

  return (
    <div>
      <button onClick={handleBtnClick}></button>
      <child count={count} />
    </div>
  );
}
```

### 监听 props

通过 `useEffect` 监听 `props` 变化：

```javascript
function child() {
  const { count } = props;

  // 监听 props 变化
  React.useEffect(() => {
    console.log(`props 改变: ${props.count}`);
  }, [props.count]);

  return <div>{{ count }}</div>;
}

function parent() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    setCount((preCount) => preCount + 1);
  };

  return (
    <div>
      <button onClick={handleBtnClick}></button>
      <child count={count} />
    </div>
  );
}
```

### props children 模式

**`props` 插槽组件**

```javascript
function App {
  //...
  return (
    <Container>
      <Children>
    </Container>
  )
}
```

这里可以通过 `props.children` 属性访问到 `Children` 组件（`Reactelement` 对象）

作用：

- 根据需要控制 `Children` 组件渲染
- 可以使用 `React.cloneElement` 强化 `props`，混入新的 `props`，或者修改 `Children` 的子元素

**`render props` 模式**

```javascript
function App {
  //...
  return (
    <Container>{(ContainerProps) => <Children {...ContainerProps} />}</Container>
  )
}
```

这种情况下， `Children` 组件是以函数的方式创建的，所以不能直接通过 `props.children` 访问到它，需要通过：

```javascript
function App {
  //...
  const containerProps = {
    name: "wjc"
  }
  // 以函数的方式访问
  console.log(props.children(containerProps))
  return (
    <Container>{(ContainerProps) => <Children {...ContainerProps} />}</Container>
  )
}
```

### Props 技巧

**抽象 props**

抽象 `props` 通常用于抽象传递 `props`，本质是直接结构传递，不需要指定 `props` 的名称

```javascript
function Son(props) {
  console.log(props)
  return (<div> hello, world</div>)
}

function Father(props) {
  const fatherProps = {
    msg: "aaa"
  }

  return (<Son {...props} {...fatherProps}>)
}

function Index() {
  const indexProps = {
    name: "alien",
    age: 28
  }

  return (<Father {...indexProps}>)
}
```

## Lifecycle

在 `class component` 中的生命周期：

**useEffect**

`useEffect` 是异步执行，它会在主线程、DOM 更新、JavaScript 执行完、试图绘制完毕才执行。

**useLayoutEffect**

`useLayoutEffect` 是同步执行，它是在 DOM 挂载前执行

### 函数组件对于 Class 组件生命周期的替代方案

**componentDidMount 替代方案**

```javascript
React.useEffect(() => {
  // 请求数据、事件监听、操作 DOM
}, []);
```

由于 `dep` 为 `[]`，表示当前 `effect` 没有任何依赖项，即只会在初始化的时候执行一次

**componentWillUnmount 替代方案**

```javascript
React.useEffect(() => {
  // 请求数据、事件监听、操作 DOM
  return function commponentWillUnmount() {
    // 解除事件监听、清除定时器、延时器
  };
}, []);
```

在 `componentDidMount` 的前提下，`useEffect` 的第一个函数返的函数，可以作为 `componentWillUnmmount` 使用

**componentWillReceiveProps 代替方案**

不能完全代替 `componentWillReceiveProps`，区别：

- 一个是在 `render` 阶段，一个是在 `commit` 阶段
- `useEffect` 在初始化的时候会执行一次，而 `componentWillReceiveProps` 只有组件更新 `props` 的时候才会执行（没有初始化执行）

```javascript
React.useEffect(() => {
  console.log("props 变化");
}, [props]);
```

**componentDidUpdate 替代方案**

不能完全代替 `componentDidUpdate`，区别：

- `useEffect` 异步执行，`componentDidUpdate` 同步执行
- `useEffect` 会默认执行一次，`componentDidUpdate` 只会在组件更新完成后执行

```javascript
React.useEffect(() => {
  console.log("组件更新完成：componentDidUpdate");
});
```

> 注意没有 `deps`

## ref

函数使用 `useRef` hooks 实现：

```javascript
export default function Index() {
  const currentDom = React.useRef(null);
  React.useEffect(() => {
    console.log(currentDom.current);
  }, []);

  return <div ref={currentDom}>ref</div>;
}
```

> 本质上 `ref` 可以用来操作组件实例进行通信和方法的调用

## Context

> Context 本质上类似于 Provide Inject 实现跨组件传递数据

首先，创建 `Context`

```javascript
const ThemeContext = React.createContext(null);
```

然后，从 `Context` 出发分别提供 `Provider` 和 `Consumer`

```javascript
const ThemeProvider = ThemeContext.Provider; //提供者
const ThemeConsumer = ThemeContext.Consumer; // 订阅消费者
```

使用 `Provider`：

```javascript
const ThemeProvider = ThemeContext.Provider; //提供者
export default function ProviderDemo() {
  const [contextValue, setContextValue] = React.useState({
    color: "#ccc",
    background: "pink",
  });
  return (
    <div>
      <ThemeProvider value={contextValue}>
        <Son />
      </ThemeProvider>
    </div>
  );
}
```

> 当 `Provider` 的 `value` 改变会让消费者重新渲染

使用 `Consumer`：

1.使用 `useContext`

```javascript
const ThemeContext = React.createContext(null);
// 函数组件 - useContext方式
function ConsumerDemo() {
  const contextValue = React.useContext(ThemeContext); /*  */
  const { color, background } = contextValue;
  return <div style={{ color, background }}>消费者</div>;
}
const Son = () => <ConsumerDemo />;
```

2.使用 `Consumer`

```javascript
const ThemeConsumer = ThemeContext.Consumer; // 订阅消费者

function ConsumerDemo(props) {
  const { color, background } = props;
  return <div style={{ color, background }}>消费者</div>;
}
const Son = () => (
  <ThemeConsumer>
    {/* 将 context 内容转化成 props  */}
    {(contextValue) => <ConsumerDemo {...contextValue} />}
  </ThemeConsumer>
);
```

> 还是要注意一下渲染的消耗，尽量使用 `React.memo` 优化

## 高阶组件

高阶组件本质上是接收一个组件，然后最终返回一个组件。

> 本质上是对组件的重新封装、劫持

**属性代理**

**反向继承**
