---
title: React提高09 Hooks
top: false
date: 2019-03-27 18:20:38
updated: 2019-10-24 14:21:41
tags:
- Hooks
categories: React
---

React Hooks学习笔记。

这篇文章真的写了好久，从1月多，Hooks还是实验特性时就开始看，中间断断续续，再加上一开始看英文的文档，拖到了现在。

总结的太磨叽了，又弄了一个精简版，那这个做分享吧。

心里也没底。

[文中涉及到的代码在这里。](https://github.com/duola8789/react-learning)

做了一个[分享的PPT](/files/React-Hooks.pptx)，如果有人需要的话可以拿走。

<!-- more -->

React Hooks是V16.8的新特性，是一个向后兼容的新特性（不会引入破坏性的改变）。

Hook是一种能够“侵入”React的函数组件的状态和生命周期特性的函数。Hook不能用在`class`中，因为Hook的目的就是你能够抛弃`class`来使用React

React提供了一些内嵌的Hooks，例如`useState`。也可以创建自己的Hook，达到在不同的组件间复用有状态的行为。

## 引入的原因

实现比现有方案（HOC/Render Props）更优雅的代码复用，为纯组件引入状态，能够将组件划分为更细的粒度。

1. 现有的React的状态组件复用方式（高阶组件、Render Props）有各自的问题， 而Hooks可以优雅的（不改变组件层次）实现代码复用
2. Hooks可以将组件根据功能，将组件划分为更小的粒度，便于调试、测试和维护
3. Hooks可以不使用`Class`来编写组件，提高代码性能，降低React的使用难度

## Hooks的使用规定

（1）只在**最顶层调用Hooks，不要在内部循环、条件语句或嵌套函数中调用Hooks**（这是因为React是通过多个Hooks的调用顺序来确定多个`useState`中状态变量的对应关系），如果想要有条件的运行一个`useEffect`，可以将条件判断放在Hook内部

```JS
useEffect(function persistForm() {
  // 👍 We're not breaking the first rule anymore
  if (name !== '') {
    localStorage.setItem('formData', name);
  }
});
```

（2）只在React函数中调用Hooks，不在普通的JavaScript函数中调用

可以通过ESLint的[`eslint-plugin-react-hooks`插件](https://www.npmjs.com/package/eslint-plugin-react-hooks)来检查、规范Hooks的使用，避免不规范的使用而导致的bug。

安装：

```BASH
npm install eslint-plugin-react-hooks -D
```

ESLint的配置文件：

```JS
{
  "plugins": [
    // ...
    "react-hooks"
  ],
  "rules": {
    // ...
    "react-hooks/rules-of-hooks": "error"
  }
}
```
将来这个插件会默认集成在Create React App和类似工具中。


## 内置Hooks

React提供了一系列[内置Hooks](https://reactjs.org/docs/hooks-reference.html)，共分为两大类，基础Hook和附加Hook。

基础Hook包括：

- `useState`
- `useEffect`
- `useContext`

附加Hook包括：

- `useReducer`
- `useCallback`
- `useMemo`
- `useRef`
- `useImperativeHandle`
- `useLayoutEffect`
- `useDebugValue`

##  `useState`

### 1 `useState` - 基础用法

```JS
import { useState } from 'react';
const [count, setCount] = useState(0);
```

内置的`useState`用来为纯组件添加状态变量和更新方法，可以认为是`this.state`和`this.setState`的简化版，以数组的形式获取状态变量，返回数组中的第一个项是一个状态，第二个是更新方法，`useState`接受的参数是初始值。

当再次**渲染**时，React会在函数组件中获取`count`的最新值，如果想要更新`count`可以调用`setCount`

### 2 `useState` - 手动合并

注意：`useState`不会自动合并更新对象，所以需要手动进行合并，举个例子，在class组件中：

```
class Test extends React.Component {
  state = { a: 1, b: 2, };

  render() {
    console.log(this.state);
    return (
      <div>
        <button onClick={() => this.setState({ a: 100 })}>click</button>
      </div>
    );
  }
}
```

点击按钮，`setState`会自动将对象合并，打印结果是`{a: 100, b: 2}`

而在使用Hooks的组件中中：

```
function Test() {
  const [state, setState] = useState({ a: 1, b: 2 });
  console.log(state);
  return (
    <div>
      <button onClick={() => setState({ a: 100 })}>click</button>
    </div>
  );
}
```

`useState`在更新时不会将对象合并，所以打印的结果是`{a: 100}`，所以需要手动进行合并，采取函数式赋值的方式：

```JS
setState(prevState => ({ ...prevState, a: 100 }))
```

这样才能保证更新后的对象是我们想要的对象。

### 3 `useState` - 延迟初始化

`useState`的参数`initialState`是首次渲染期间使用的状态，在后续的更新渲染过程中，它会被忽略，因为`state`会采用上一次更新后最新的值，但是如果这个初始状态仍然会被计算一次。

```
// 使用useCallback
function compute() {
  console.log('computing...');
  return 0;
}

export default function () {
  const [count, setCount] = useState(compute() + 1);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Add Count {count}</button>
    </div>
  );
}
```

点击按钮，`compute`函数每次都被调用。

如果这个状态是一个高开销的计算结果，可以改为提供函数，这个函数仅在初始渲染时执行，可以避免性能浪费：

```JS
const [count, setCount] = useState(() => compute() + 1);
```

在后续渲染时，初始状态计算就会被跳过了。

## `useEffect`

### 1 `useEffect` - 基础用法

```
useEffect(didUpdate, dependencyArray);
```

接受一个函数`didUpdate`和依赖数组`dependencyArray`（可选），默认在在每次渲染（首次渲染及后续更新）后执行`didUpdate`方法。React会保证在DOM更新完成后才会调用effect。

通过使用这个Hook，React会保存传入的函数并且在每次DOM更新后进行调用。组件中的`useEffect`可以获取函数的内部的所有变量和Prop，因为它已经在函数的作用域中了。

```JS
useEffect(() => {
  document.title = `You clicked ${count} times`;
});
```

`useEffect`将Class组件的`componentDidMount`/的`componentDidUpdate`生命周期合并，逻辑更集中，而且可以减少因为没有在`componentDidUpdate`处理更新前的状态而导致的bug。

可以再组件中使用多个`useEffect`来分离关注点，**这让我们能够基于代码的行为来分割代码**，而不是基于生命周期。React会按照`useEffect`声明的顺序，运行组件中的每一个`useEffect`

### 2 `useEffect` - 销毁

如果在`useEffect`中的更新函数中创建的一些事件需要在组件卸载时清理（比如定时器或者事件订阅等），

可以为`didUpdate`更新函数返回一个新的函数，这个函数可以作为清理函数，用来执行销毁操作即可。和执行一样，销毁也是在每次渲染后都会执行，可以防止内存泄漏。

```JS
useEffect(() => {
  document.title = `You clicked ${count} times`;
  return () => {
    document.title = `ok`;  
  }
});
```

### 3 `useEffect` - 避免重复渲染

`useEffect`默认的表现是在每次渲染后触发，当组件的任何一个状态发生改变时，更新函数都会执行。某些情况下，每次渲染都销毁或者应用effect会造成性能问题。在Class组件中，我们可以通过在`componentDidUpdate`中对比`prevProps`和`prevState`来解决这个问题

如果在重复渲染时某些特定值未发生改变，你可以让React不再运行effect。具体做法是将一个数组作为可选的第二个参数传递给`useEffect`。这时只有当数组中的任一一项的值发生变化，`useEffect`的更新函数才会执行。

```JS
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // Only re-run the effect if count changes
```

这个数组并不会作为参数传递给更新函数内部，但是**更新函数中引用的每个值都应该出现在输入数组中**，这样才能避免更新函数依赖的某个值发生了变化，而函数没有重新执行（ESLint的插件会自动检测并插入这个数组，推荐使用）。

注意，如果进行这种优化，确保数组中包含了被更新函数使用的、会随时间变化的外部变量。否则你的代码从上次渲染中获取的参考值不会改变。

如果想只执行、销毁`useEffect`一次（组件创建和销毁时），可以传递一个空数组作为第二个参数。这告诉了React这个`useEffect`不依赖任何从`props`和`state`中任何变量，所以不需要重复执行。这种情况与只在`componentDidMount`和`componentWillUnmount`执行代码是类似的。建议谨慎使用，因为容易导致bug

### 4  `useEffect` - 执行时机

`useEffect`中的更新函数会延迟到`layout`和`paint`后触发，也就是说在浏览器更新屏幕之后才会触发，因为它所针对的事件是订阅等事件处理程序，不应该组织UI界面的更新。

但是有一些事件不能推迟，比如用户可见的DOM改变必须在下一次绘制之前同步触发，避免用户感觉到操作与视觉的不一致性。对于这个类型的事件需要在`useLayoutEffect`中触发，它与`useEffect`的不同就是在触发时机上的不同。

虽然`useEffect`延迟到浏览器绘制完成之后执行，但是它保证在任何新渲染之前触发。


### 5 `useEffect` - 在更新函数中获取本次渲染更新后的值

在`useEffect`的更新函数中，**拿到的`state`和`props`总是当次渲染的初始值**，即便在更新函数中执行了`setState`之后仍是这样。

看这样一个例子：

```
export default function () {
  const [count, setCount] = useState(0);

  // useEffect1
  useEffect(() => {
    console.log(count, 'useEffect1');
  }, [count]);

  // useEffect2
  useEffect(() => {
    console.log(count, 'useEffect2');
  }, [count]);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>{count}</button>
    </div>
  );
}
```

当我们点击按钮的时候，`count`值变为`1`，这个时候，`useEffect1`和`useEffect2`中都因为`count`值变化而重新执行，打印的结果都是`1`，UI界面也同步更新为`1`

对上面的例子稍加改造，在`useEffect1`中添加`setCount(100)`，再次点击按钮，看一下执行结果：

```
export default function () {
  const [count, setCount] = useState(0);

  // useEffect1
  useEffect(() => {
    setCount(100);
    console.log(count, 'useEffect1');
  }, [count]);

  // useEffect2
  useEffect(() => {
    console.log(count, 'useEffect2');
  }, [count]);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>{count}</button>
    </div>
  );
}
```

1. 页面初始化，此时`count`值为`0`，也就是说，本轮渲染的`count`初始值是`0`，
2. 执行`useEffect1`，对`count`复制`setCount(100)`，此时，在`useEffect1`中，**拿到的`state`仍然是当次渲染的初始值**，所以打印的结果是`0 useEffect1`
3. 执行`useEffect2`，此时，在`useEffect2`中，**拿到的`state`仍然是当次渲染的初始值**，所以打印的结果是`0 useEffect2`
4. 执行`return`部分，此时UI界面更新，获取到`setCount(100)`后的`count`值，所以此时界面展示`100`
5. 由于`count`值有初始的`0`变成了`100`，所以`useEffect1`和`useEffect2`会再次被分别调用，和上一轮调用的唯一区别就是本次渲染`count`的初始值变为了`100`
6. 所以会分别打印`100 useEffect1`和`100 useEffect2`
7. 由于`count`值稳定在了`100`，所以`useEffect`不会再被调用
8. 如果点击按钮，执行过程是类似的

实际上`useEffect`的执行时机与class组件的[`setState`的执行时机]不完全想恶童(https://duola8789.github.io/2019/07/24/01%20%E5%89%8D%E7%AB%AF%E7%AC%94%E8%AE%B0/03%20React/React02%20%E6%8F%90%E9%AB%98/React%E6%8F%90%E9%AB%9801%20SetState%E7%9A%84%E6%89%A7%E8%A1%8C%E6%97%B6%E6%9C%BA/)类似：
>
> `setState`会不会立刻更新`state`取决于调用`setState`时是不是已经处于批量更新事务中。在批量更新事务中调用`setState`不会立即执行，而是放到队列中等待批量更新事务结束后统一执行。
>
> 组件的生命周期函数和绑定的事件回调函数都是在批量更新事务中执行的。

可以认为每次渲染时通过`useState`声明的状态是不可变的（Immutable），每次渲染都会对它拍一个快照保存下来，当状态更新重新渲染时就会形成N个状态。不光是`state`和`props`通过快照的形式保存，组件的事件处理和`useEffect`都是同样的形式。

我犯过的一个错误就是，在`useEffect1`中通过`setCount1`更新了`count1`的值，而在`useEffect2`中要使用更新后的`count1`的值，这就会导致错误，因为在任何一个`useEffect`中拿到的`count1`的值是当次更新的`count1`的初始值，而不会是在`useEffect1`中更新后的值。

如何解决这个问题呢？我觉得有两个方法，一个是更好的组织`useEffect`，一个`useEffect`中不要完成过多的功能，更不要成为一个中间过程，为最终渲染的结果提供中间数据，而是让每个`useEffect`都提供渲染需要的最终数据。

如果确实要在`useEffect`的更新函数中使用更新后的`state`，那么就需要使用React提供了另外一种内置Hook了，`useRef`。

> 注意，在渲染结果中拿到的都是更新后的最新的`props`和`state`，如果在渲染结果中出现了旧的`props`和`state`，那么很可能是遗漏了一些依赖，导致对应的`useEffect`没有按照预期执行。还是推荐使用前面提到的ESLint的插件来帮助我们发现和解决问题。

## `useRef`

```JS
const refContainer = useRef(initialValue);
```

`useRef`返回一个可变的对象，其`current`属性被初始化为传递的参数，返回的这个对象就保留在组件的生命周期中。

`useRef`返回的`ref`对象在所有Render过程中保持着唯一引用，如果认为`state`是不可变的数据，那么`ref`对象就可以认为是可变对象，对`ref.current`的赋值和取值，拿到的都是同一个状态。

```
export default function () {
  const count = useRef(1);

  // useEffect1
  useEffect(() => {
    // 赋值
    count.current = 100;
    console.log(count, 'useEffect1');
  });

  // useEffect2
  useEffect(() => {
    // 赋值
    count.current = 200;
    console.log(count, 'useEffect2');
  });
  
  return (
    <div />
  );
}
```

使用`useRef`就可以在当次渲染获取到改变后的值，所以打印结果是：

```
100 "useEffect1"
200 "useEffect2"
```

要注意，避免在渲染结果中（`return`中）直接引用`ref`对象，可能会导致预料之外的结果。相反，应该只在事件处理程序和`useEffect`中修改、使用`ref`对象。

## `useContext`

```JS
const context = useContext(MyContext);
```

用来创建`context`对象，参数接受一个`React.createContext`的结果，返回改`context`的当前值。当前的`context`值由上层组件中距离当前组件最近的`<MyContext.Provider>`的名为`value`的Prop决定。

`useContext(MyContext)`只是简化了子组件使用`MyContext.Consumen`的方式，仍然需要在上层组件树中使用`<MyContext.Provider> `来为下层组件提供`context`。

不使用`useContext`：

```
// 创建一个 context 对象
const MyContext = React.createContext();

// Provider
const Provider = ({ children }) => {
  const [msg, setMsg] = useState('Hello Child');

  return (
    <MyContext.Provider value={{ msg, setMsg }}>
      <h2>Parent -- {msg}</h2>
      {children}
    </MyContext.Provider>
  );
};

// 不使用 useContext
const Consumer  = MyContext.Consumer;
const Child = () => {
  return (
    <Consumer>
      {({ msg, setMsg }) => (
        <button onClick={() => setMsg('Hello Parent')}>Child -- {msg}</button>
      )}
    </Consumer>
  );
};

export default function () {
  return (
    <Provider>
      <Child />
    </Provider>
  );
}
```

使用`useContext`进行简化：

```
// 创建一个 context 对象
const MyContext = React.createContext();

// Provider
const Provider = ({ children }) => {
  const [msg, setMsg] = useState('Hello Child');

  return (
    <MyContext.Provider value={{ msg, setMsg }}>
      <h2>Parent -- {msg}</h2>
      {children}
    </MyContext.Provider>
  );
};

// 使用 useContext
const Child = () => {
  const { msg, setMsg } = useContext(MyContext);

  return (
    <button onClick={() => setMsg('Hello Parent')}>Child -- {msg}</button>
  );
};

export default function () {
  return (
    <Provider>
      <Child />
    </Provider>
  );
}
```

## `useReducer`

```JS
const [state, dispatch] = useReducer(reducer, initialState，initialAction);
```

`useState`的替代方案，当组件使用Flux架构组织管理数据时有用。

接受类型为`(state, action) => newState`的Reducer，返回与`dispatch`方法匹配的当前状态。`initialAction`是可选的，提供初始的`action`。


```
const initialState = { count: 0 };

function reducer(state, action) {
  const { type } = action;
  switch (type) {
    case 'reset': {
      return initialState;
    }
    case 'increment': {
      return { count: state.count + 1 };
    }
    case 'decrement': {
      return { count: state.count - 1 };
    }
  }
}

export default function () {
  const [state, dispatch] = useReducer(reducer, initialState);

  return (
    <div>
      <p>{state.count}</p>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
    </div>
  );
}
```

这个时候的`state`由Reducer得来，更新方法`dispatch`是匹配reducer的`dispatch({type: 'type'})`更新方法。

用它配合`useContext`可以避免在多层组件中深度传递回调的需要。

## `useCallback`

```JS
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

主要是用来处理在`useEffect`之外的定义函数无法管理依赖，也无法成为`useEffect`的依赖，每次渲染都会生成新的快照的情况，使用之后只有在函数的依赖发生变化时才会生成新的函数，有利于提高性能，依赖也更清晰。

如果一个函数依赖了组件的`state`，并且由于复用的原因，不能放在`useEffect`中，就将这个函数用`useCallback`包装，返回的变量可以作为对应的`useEffect`的依赖，当其依赖发生变化时，返回新的函数引用，同时触发对应的`useEffect`重新执行。

我理解使用的原因主要出于性能优化和便于维护，例如，如果在组件中定义了一个函数：

```JS
function fetch() {
  return state + 1000;
}
```

其中的`useEffect`无法添加`fetch`作为依赖，因为它是一个普通的函数，而且每次渲染`fetch`都会生成一个快照，如果使用了`useCallback`：


```JS
const fetch = useCallback(() => {
  return state + 1000;
}, [state]);
```
使用了`useCallback`之后，依赖更清晰，并且在`state`未发生变化时不会生成新的快照，有助于性能的提高。


## `useMemo`

```JS
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

与`useCallback`类似，返回的是一个不生成快照的对象，而非函数。

`useMemo`只会在其中一个输入发生更改时重新计算，此优化有助于避免在每个渲染上进行高开销的计算。


## `useLayoutEffect`

前面介绍过，与`useEffect`的不同点仅仅在于执行时机不同，`useLayoutEffect`在绘制前同步触发，`useEffect`会推迟到绘制后触发


## 参考

- [Hooks API Reference@React](https://reactjs.org/docs/hooks-reference.html)
- [Hooks API 参考@React中文文档](http://react.html.cn/docs/hooks-reference.html)
- [精读《useEffect 完全指南》@掘进](https://juejin.im/post/5c9827745188250ff85afe50#heading-8)
- [useEffect 完整指南@overreacted](https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/)
- [快速上手三大基础 React Hooks@掘金](https://juejin.im/post/5c8918ca6fb9a049f572023e)
