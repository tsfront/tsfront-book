# Hooks

> 自从React v16.8版本引入Hooks，加速了原先class类型组件的淘汰速度，也为React 争取回了不少用户。网友一片真香！

[Toc]

## [1.动机](https://reactjs.org/docs/hooks-intro.html#motivation)

### 1.1 当前组件间状态逻辑难以重复利用。「It’s hard to reuse stateful logic between components」「 Cross-Cutting 问题」

- Hooks allow you to reuse stateful logic without changing your component hierarchy

- 在没有hook的年代，替代方案有：
  - [render props](https://reactjs.org/docs/render-props.html)
    - 通过向子组件传递函数型属性，达到共享代码。[a technique for sharing code between React components using a prop whose value is a function](https://reactjs.org/docs/render-props.html#use-render-props-for-cross-cutting-concerns)
    - 只要通过属性传递函数，都算`render props`。比如向子组件属性传递一个handler：[*any* prop that is a function that a component uses to know what to render is technically a “render prop"](https://cdb.reacttraining.com/use-a-render-prop-50de598f11ce)
    - 对React.PureComponent 使用，需要特别注意。[Be careful when using Render Props with React.PureComponent](https://reactjs.org/docs/render-props.html#be-careful-when-using-render-props-with-reactpurecomponent)
  - [higher-order components(HOC)](https://reactjs.org/docs/higher-order-components.html)
    - 输入component/输出component的函数型组件。a higher-order component is a function that takes a component and returns a new component
    - 典型的，如react-redux `connect`函数；react-router-dom `withRouter`函数；各种 `Provider`函数
    - 注意不要对被包裹的组件进行mutate（又来这个词了）。[Don’t Mutate the Original Component. Use Composition.](https://reactjs.org/docs/higher-order-components.html#dont-mutate-the-original-component-use-composition)

  

### 1.2 class组件生命周期过于复杂、以至于看不明白。Complex components become hard to understand

- Hooks let you split one component into smaller functions based on what pieces are related (such as setting up a subscription or fetching data)
- `useEffect` hook

### 1.3 class组件痼疾缠身、与react的宗旨（官网用了`spirit of react`）背驰。

- have to understand how `this` works
- have to remember to bind the event handlers
- Hooks let you use more of React’s features without classes
- Hooks embrace functions, but without sacrificing the practical spirit of React（Hooks拥抱/适合函数类型的组件、还不牺牲React实用性的精神（潜台词，Class类型牺牲了...）

## 2.概览

### 2.1 `useState()`

- 必须在function组件里使用。call it inside a function component to add some local state to it
- 在组件渲染期间，一直存在/可用。React will preserve this state between re-renders
- 用法：`const [name, setName] = useState('zhangsan')`，前者是状态变量，后者是状态的操作函数。约定俗成，状态变量`abc`的操作函数，命名成`setAbc`
- `setAbc`，类同class组件里的`this.setState`。但是！！！**it doesn’t merge the old and new state together**

```react
// 使用this.setState
<button onClick={() => this.setState({ count: this.state.count + 1 })}>
    Click me
</button>

// 使用setCount
<button onClick={() => setCount(count + 1)}>
    Click me
</button>
```

- `useState(xxx)`接受一个初始化参数，且仅在首次渲染才有效。The initial state argument is only used during the first render.

- `useState`定义的状态，可以不是（也最好不是）object类型。unlike `this.state`, the state here doesn’t have to be an object。一个组件里，可以使用很多次`useState()`。

- ```react
  const ExampleWithManyStates = () => {
    // Declare multiple state variables!
    const [age, setAge] = useState(42);
    const [fruit, setFruit] = useState('banana');
    const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
    // ...
  }
  ```

- 灵魂拷问：But what is a Hook?

- hook就是埋在react function组件里面的“钩子”。就钩的内容不同，衍生了很多种类的hook。比如，状态的钩子就是`useState`；生命周期的钩子就是`useEffect`（在特定阶段执行）。Hooks are functions that let you “hook into” React state and lifecycle features from function components

- hook自打“出生”，就与class组件无缘了。Hooks don’t work inside classes
- 详细使用见：https://reactjs.org/docs/hooks-state.html

### 2.2 `useEffect`

- 在class组件时代，你会有：请求数据、订阅、修改dom等操作。这些操作被称作`side effects`。这么命名，是因为这些操作1）会影响到组件之外的其他组件、2）不能在组件渲染期间操作。**class组件所以引入了生命周期概念，以便插入这些操作。**

- `useEffect`的引入，使得function组件（也叫无状态组件、也就没有生命周期）内也能操作上述side effect。It serves the same purpose as componentDidMount, componentDidUpdate, and componentWillUnmount in React classes, but unified into a single API。

- 以下示例，在react 更新完dom后，用`useEffect`设置网页的标题

- ```react
  import React, { useState, useEffect } from 'react';
  
  const Example = () => {
    const [count, setCount] = useState(0);
  
    // 类似 class组件中 componentDidMount 和 componentDidUpdate:
    useEffect(() => {
      document.title = `You clicked ${count} times`;
    });
  
    return (
      <div>
        <p>You clicked {count} times</p>
        <button onClick={() => setCount(count + 1)}>
          Click me
        </button>
      </div>
    );
  }
  ```

- `useEffect`定义在组件中，可以触达组件内的所有属性（`props`）和状态（`state`）变量。 Effects are declared inside the component so they have access to its props and state

- `useEffect(执行函数，依赖数组)`。**默认情况下**，每一次组件渲染完都会执行一次 `useEffect`的内容，包括组件首次渲染。如果要控制仅首次渲染执行、后续再次渲染组件不执行`useEffect`，需要设置第二个依赖数组参数为`[]`。

- 如果执行函数可以有return，也可以没有。如果return一个函数，可以起到“clean up”作用。这个“clean up”作用是在 组件unmount时候、或者下一次执行useEffect之前 执行。

- ```react
  const FriendStatusWithCounter = (props) => {
    const [isOnline, setIsOnline] = useState(null);
    
    useEffect(() => {
      // 当朋友上线时候，聊天室跟进朋友状态
      ChatAPI.subscribeToFriendStatus(props.friend.id, handleStatusChange);
      return () => {
        // 当朋友下线时候，聊天室取消朋友状态
        ChatAPI.unsubscribeFromFriendStatus(props.friend.id, handleStatusChange);
      };
    });
  
    const handleStatusChange = (status) => {
      setIsOnline(status.isOnline);
    }
  ```

- 类同`useState`，一个组件中可以有多个`useEffect`。可以根据effect内容进行划分，而不用像class组件时代根据生命周期划分、同一个生命周期内杂糅很多不同操作的effect。

- 详细使用见：https://reactjs.org/docs/hooks-effect.html

### 2.3 其他hook，参考：https://reactjs.org/docs/hooks-reference.html

## 3.规范

### 3.1 hook是js函数，但区别于一般js函数

- hook仅能在最高level使用，不能在循环、条件判断或内嵌函数内调用。Only call Hooks at the top level，Don’t call Hooks inside loops, conditions, or nested functions.

- hook仅在react 函数组件内使用。普通js函数内，是不能使用react内置的这些hooks，除非是你手动自建的hook。

## 4.官网QA

### 4.1 使用策略

- 哪个版本开始使用 hook？v16.8.0
- 是否立刻马上改写所有class组件？保持旧代码；在新代码中尝试hook
- 哪些是hook特有功能而class组件没有的？参考：[making-sense-of-react-hook](https://medium.com/@dan_abramov/making-sense-of-react-hooks-fdbde8803889)
- hook会改变我对react 的认知？状态、生命周期、上下文、ref 概念一直存在，hook换了一种直接实现方式而已
- hook是否可以涵盖所有class组件应用领域？否，getSnapshotBeforeUpdate, getDerivedStateFromError 和 componentDidCatch 生命周期暂时还没在hook中有同等实现。
- hook替代了render props和higher-order components?部分，但不能全部替代。
- hook对流行的redux `connect`和react-router影响？ 你还可以继续使用第三方组件。react-redux 自v7.1.0支持了hook，提供了`useDispatch`、`useSelector`等hook接口。react-router 自v5.1支持了hook
- hook如果做静态类型检查？

### 4.2 从class迁移到hooks

- **class组件生命周期方法如何对应hooks**？
  - constructor：function组件不需要constructor。在`useState(xxx)`初始化状态
  - ~~getDerivedStateFromProps：[How do I implement `getDerivedStateFromProps`?](https://reactjs.org/docs/hooks-faq.html#how-do-i-implement-getderivedstatefromprops)~~
  - ~~shouldComponentUpdate：[How do I implement `shouldComponentUpdate`?](https://reactjs.org/docs/hooks-faq.html#how-do-i-implement-shouldcomponentupdate)~~
  - render：函数体本身就是对等于class组件的render()内容
  - componentDidMount，componentDidUpdate，componentWillUnmount：通过`useEffect`不同使用方法可以替代，[Can I skip an effect on updates?](https://reactjs.org/docs/hooks-faq.html#can-i-skip-an-effect-on-updates)、[Can I run an effect only on updates?](https://reactjs.org/docs/hooks-faq.html#can-i-run-an-effect-only-on-updates)
  - ~~getSnapshotBeforeUpdate、componentDidCatch、getDerivedStateFromError：暂时没有hook版，敬请期待~~

- 如何在hook中请求数据？以下简单例子，详细参考：[react-hooks-fetch-data](https://www.robinwieruch.de/react-hooks-fetch-data/)

  ```react
  import React, { useState, useEffect } from "react";
  import axios from 'axios';
  
  const SearchResults = () => {
    const [data, setData] = useState({ hits: [] });
    const [query, setQuery] = useState('react');
  
    useEffect(() => {
      let ignore = false;
      // 注意：这里定义了异步函数    
      async const fetchData = () => {
        const result = await axios('https://hn.algolia.com/api/v1/search?query=' + query);
        if (!ignore) setData(result.data);
      }
      // 执行异步函数
      fetchData();
      // 定义了回收函数
      return () => { ignore = true; }
    }, [query]);
    
    return (<div></div>)
  }
  
  ```

- ~~有没有定义常量的hook？使用`useRef`（我也还没搞明白）~~
- 我应该使用一个（object）状态还是多个状态？
  - class组件使用者习惯用oject包裹所有状态变量，然后在useState中定义。可以，但不建议
  - 修改其中一个属性，更新的code将会冗余
  - 官方建议，将可能一起变化的多个变量，写在一个状态变量里面。we recommend to split state into multiple state variables based on which values tend to change together

- 如何取到上一个属性和状态值？通过`useRef` + `useEffect`

  ```react
  // 【1】记录状态变量的前值
  const Counter() => {
    const [count, setCount] = useState(0);
    // 使用useRef 记住历史状态
    const prevCountRef = useRef();
    useEffect(() => {
      prevCountRef.current = count;
      ...
      setCount(count + 1)
    });
    
    const prevCount = prevCountRef.current;
  
    return <h1>Now: {count}, before: {prevCount}</h1>;
  }
  
  // 【2】记录普通变量的前值
  const Counter() => {
    const [count, setCount] = useState(0);
    // 使用useRef记住任何变量
    const calculation = count + 100;
    const prevCalculationRef = useRef();
    useEffect(() => {
      prevCalculationRef.current = calculation;
      ...
      setCount(count + 1)
    });
    
    const prevCalculation = prevCalculationRef.current;
    
    return <h1>Now: {calculation}, before: {prevCalculation}</h1>;
  }  
  ```

- 为什么我的组件内状态、属性是陈旧的、没有及时更新？
  - 一般不会，组件内任何函数、包括事件处理函数、effect函数，都能及时“看到”最新一次渲染后的属性、状态值。
  - 除非错误设置了 useEffect 第二个依赖数组参数

### 4.3 性能优化

- 如何制止后续更新后再次执行 effect？

  - useEffect默认是组件每次渲染后都会执行，以确保状态得到更新。
  - 个别场合下，这会引起一些不必要的性能消耗。可以在`useEffect`第二参数设定依赖变量，来减少重复执行effect次数
  - 更极端的，设置`useEffect`第二参数为`[]`，则永远仅在首次渲染执行一次

  ```react
  useEffect(
    () => {
      const subscription = props.source.subscribe();
      return () => {
        subscription.unsubscribe();
      };
    },
    // 如果不指定，相当于props任何变化 导致组件重新渲染，接着导致一次effect执行
    // 如果设定props.source，那么只有props.source有变动，effect才会再次执行
    // 注意！！！ 一旦设置了第二参数，就一定要设置全、各种状态变量、属性变量，否则你会有「我的effect总不执行」的困惑
    [props.source],
  );
  ```

- 在useEffect中删除/忽略依赖的函数是否安全？ 不！

  - 如果真的需要定义函数，建议在useEffect中定义

  ```react
  const Example = ({ someProp }) => {
    const doSomething() => {
      // 这个外部函数 使用了属性、就有可能会有变动
      console.log(someProp);
    }
  
    useEffect(() => {
      doSomething();
    }, []); // 🔴 This is not safe (it calls `doSomething` which uses `someProp`)
  }
  
  const Example({ someProp }) => {
    useEffect(() => {
      const doSomething() => {
        console.log(someProp);
      }
      
      doSomething();
    }, [someProp]); // ✅ OK (our effect only uses `someProp`)
  }
  ```

  - 如果确确实实不能定义在useEffect中，比如有些函数是通用的，还会被其他处调用。1）把函数移出你的组件之外、这样确保该函数不会依赖任何组件内的状态变量、属性变量。2）如果该函数纯粹计算用、且对渲染没有任何影响，你可以把函数放在组件内、但放在useEffect之外，当普通函数执行。3）用`useCallback`包裹函数，并定义好它的依赖，这样就可以直接在本组件useEffect中使用，见下面示例

    ```react
    const ProductPage({ productId }) => {
      // 用useEffect包裹，确保不会在每次渲染期间发生变动
      const fetchProduct = useCallback(() => {
        // ... 一些涉及productId的计算  ...
        // 同时在依赖中写明哪些依赖
      }, [productId]);
      
      useEffect(() => {
        fetchProduct();
      }, [fetchProduct]); 
      
      // ...
    }
    ```

- 如果useEffect的依赖变动太频繁，我该怎么办？

  - 很直觉强制设置 第二参数为`[]`。但！！！这会有bug。

  ```react
  const Counter() => {
    const [count, setCount] = useState(0);
    useEffect(() => {
      const id = setInterval(() => {
        // 依赖状态变量`count`
        setCount(count + 1); 
      }, 1000);
      return () => clearInterval(id);
    }, []); // 🔴 Bug: `count` is not specified as a dependency
    // 设置 `[count]` 也不合适，会重复创建很多setInterval...理解下？？？？？？？？？？/
    return <h1>{count}</h1>;
  }
  
  // 优化版
  const Counter() => {
    const [count, setCount] = useState(0);
    useEffect(() => {
      const id = setInterval(() => {
        setCount(c => c + 1); // ✅ This doesn't depend on `count` variable outside
      }, 1000);
      return () => clearInterval(id);
    }, []); // ✅ Our effect doesn't use any variables in the component scope
    return <h1>{count}</h1>;
  }
  ```
  - 如何手动写一个`shouldComponentUpdate`? 使用`React.memo`对属性进行比较。只有在props发生改变时候才update。注意不能对state进行比较

  ```react
  const Button = React.memo((props) => {
    // your component
  });
  ```

  - 如何内存（比较）计算？使用`useMemo`
    - 以下代码执行会执行`computeExpensiveValue(a, b)`。但如果依赖变量`[a, b]`在内存中没有变化，`useMemo`会跳过本次执行`computeExpensiveValue(a, b)`函数，而直接使用上次执行后的结果。
      - 但注意！！！ 传递给useMemo的函数`() => computeExpensiveValue(a, b)，还是随组件每次重新渲染都会执行的。仅仅只是执行的时候，跳不跳过`computeExpensiveValue(a, b)`。

  ```react
  const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
  ```

  