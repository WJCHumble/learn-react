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
// 或者 import 入的
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
