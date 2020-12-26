# React Hook丨用好这9个钩子，所向披靡
```Hook``` 出来后，相信很多小伙伴都自己跃跃欲试，对于喜欢用```React```的，又喜欢```Hook```的，本篇文章将会与你一起玩转```Hook```。

> 钩子函数：某个阶段触发的回调函数。 例：```vue```的生命周期函数就是钩子函数

工欲善其事，必先利其器

让我们先深入了解```React```内置的这几个钩子

这里我们简单给几个钩子贴上标签

<br/>

#### ```useState【维护状态】```
#### ```useEffect【完成副作用操作】```
#### ```useContext【使用共享状态】```
#### ```useReducer【类似redux】```
#### ```useCallback【缓存函数】```
#### ```useMemo【缓存值】```
#### ```useRef【访问DOM】```
#### ```useImperativeHandle【使用子组件暴露的值/方法】```
#### ```useLayoutEffect【完成副作用操作，会阻塞浏览器绘制】```

<br/>

接下来，我们来针对这9个钩子一一深入了解

## ```useState```

> 普通更新 / 函数式更新 ```state```

```JavaScript
const Index = () => {
  const [count, setCount] = useState(0);
  const [obj, setObj] = useState({ id: 1 });
  return (
    <>
      {/* 普通更新 */}
      <div>count：{count}</div>
      <button onClick={() => setCount(count + 1)}>add</button>

      {/* 函数式更新 */}
      <div>obj：{JSON.stringify(obj)}</div>
      <button
        onClick={() =>
          setObj((prevObj) => ({ ...prevObj, ...{ id: 2, name: "张三" } }))
        }
      >
        merge
      </button>
    </>
  );
};
```

## ```useEffect```
> ```useEffet``` 我们可以理解成它替换了```componentDidMount```, ```componentDidUpdate```, ```componentWillUnmount``` 这三个生命周期，但是它的功能还更强大。

这个钩子比较重要，我们花点时间来掌握它。

包含3个生命周期的代码结构
```JavaScript
useEffect(
  () => {
    // 这里的代码块 等价于 componentDidMount
    // do something...
    // return的写法 等价于 componentWillUnmount 
    return () => {
       // do something...
    };
  },
  // 依赖列表，当依赖的值有变更时候，执行副作用函数，等价于 componentDidUpdate
  [ xxx，obj.xxx ]
);
```

注意：依赖列表是灵活的，有三种写法

当数组为空 ```[]```，表示不会应为页面的状态改变而执行回调方法```【即仅在初始化时执行，componentDidMount】```，
当这个参数不传递，表示页面的任何状态一旦变更都会执行回调方法
当数组非空，数组里的值一旦有变化，就会执行回调方法
我们还会遇到一些场景，如：
场景1：我依赖了某些值，但是我不要在初始化就执行回调方法，我要让依赖改变再去执行回调方法
我们这里有用到了 ```useRef``` 这个钩子：

```JavaScript
const firstLoad = useRef(true);
useEffect(() => {
  if (firstLoad.current) {
    firstLoad.current = false;
    return;
  }
  // do something...
}, [ xxx ]);
```

> 场景2：我有一个getData的异步请求方法，我要让其在初始化调用且点击某个按钮也可以调用
我们先这样写
```JavaScript
// ...
  const getData = async () => {
    const data = await xxx({ id: 1 });
    setDetail(data);
  };

  useEffect(() => {
    getData();
  }, []);

  const handleClick = () => {
    getData();
  };
// ...
```
但是报了个```Warning```：
```
Line 77:6:  React Hook useEffect has a missing dependency: 'getData'. 
Either include it or remove the dependency array 
react-hooks/exhaustive-deps
```

报错的意思就是：我需要 ```useEffect``` 需要添加```getData```依赖

这是```Hook```的规则，于是我们这样改：

```JavaScript
// ...
  const getData = async () => {
    const data = await xxx({ id: 1 });
    setDetail(data);
  };

  useEffect(() => {
    getData();
  }, [getData]);

  const handleClick = () => {
    getData();
  };
// ...
```

但是又报了个```Warning```：

```
Line 39:9:  The 'getData' function makes the dependencies of useEffect Hook (at line 76) change on every render. 
Move it inside the useEffect callback. 
Alternatively, wrap the 'getData' definition into its own useCallback() Hook react-hooks/exhaustive-deps
```

报错的意思就是：这个组件只要一有更新触发了```render```， ```getData```的就会重新被定义，此时的引用不一样，会导致```useEffect```运行。

这个是影响性能的行为，我们用 ```useCallback``` 钩子来缓存它来提高性能：

```JavaScript
// ...
  const getData = useCallback(async () => {
    const data = await xxx({ id: 1 });
    setDetail(data);
  }, []);

  useEffect(() => {
    getData();
  }, [getData]);

  const handleClick = () => {
    getData();
  };
// ...
```

这只是一个例子，主要是为了说明 根据错误提示，从而引起的思考。当然你也可以用一个注释来关闭```eslint```，或者直接关闭```eslint```规则，这主要看你的取舍。

使用 ```// eslint-disable-next-line react-hooks/exhaustive-deps```，如：

```JavaScript
// ...
  const [count, setCount] = useState(1);
  const xxx = () => {};
  useEffect(() => {
    // use count do something...
    console.log(count);    

    xxx();
    // eslint-disable-next-line react-hooks/exhaustive-deps
  }, []);
// ...
```

## ```useContext```
> Context 提供了一种在组件之间共享此类值的方式，而不必显式地通过组件树 的逐层传递 props

一个例子说明
```JavaScrpit
const obj = {
  value: 1
};
const obj2 = {
  value: 2
};

const ObjContext = React.createContext(obj);
const Obj2Context = React.createContext(obj2);

const App = () => {
  return (
    <ObjContext.Provider value={obj}>
      <Obj2Context.Provider value={obj2}>
        <ChildComp />
      </Obj2Context.Provider>
    </ObjContext.Provider>
  );
};
// 子级
const ChildComp = () => {
  return <ChildChildComp />;
};
// 孙级或更多级
const ChildChildComp = () => {
  const obj = useContext(ObjContext);
  const obj2 = useContext(Obj2Context);
  return (
    <>
      <div>{obj.value}</div>
      <div>{obj2.value}</div>
    </>
  );
};
```

## ```useReducer```
> 在某些场景下，```useReducer``` 会比 ```useState``` 更适用，当```state```逻辑较复杂。我们就可以用这个钩子来代替```useState```，它的工作方式犹如 ```Redux```，看一个例子：

```JavaScript
const initialState = [
  { id: 1, name: "张三" },
  { id: 2, name: "李四" }
];

const reducer = (state: any, { type, payload }: any) => {
  switch (type) {
    case "add":
      return [...state, payload];
    case "remove":
      return state.filter((item: any) => item.id !== payload.id);
    case "update":
      return state.map((item: any) =>
        item.id === payload.id ? { ...item, ...payload } : item
      );
    case "clear":
      return [];
    default:
      throw new Error();
  }
};

const List = () => {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      List: {JSON.stringify(state)}
      <button
        onClick={() =>
          dispatch({ type: "add", payload: { id: 3, name: "周五" } })
        }
      >
        add
      </button>

      <button onClick={() => dispatch({ type: "remove", payload: { id: 1 } })}>
        remove
      </button>

      <button
        onClick={() =>
          dispatch({ type: "update", payload: { id: 2, name: "李四-update" } })
        }
      >
        update
      </button>
      
      <button onClick={() => dispatch({ type: "clear" })}>clear</button>
    </>
  );
};
```

暴露出去的 ```type``` 可以让我们更加的了解，当下我们正在做什么事。

## ```useCallback```
```JavaScript
// 除非 `a` 或 `b` 改变，否则不会变
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
```

动手滑到上面，已经有提到了一个例子，说到了 ```useCallback``` ，算是一个场景， 我们都知道它可以用来缓存一个函数。

接下来我们讲讲另一个场景。

前面讲的话：```react```中只要父组件的```render```了，那么默认情况下就会触发子组的```render```，```react```提供了来避免这种重渲染的性能开销的一些方法： ```React.PureComponent```、```React.memo``` ，```shouldComponentUpdate()```

让我们来耐心看一个例子，当我们子组件接受属性是一个方法的时候，如：

```JavaScript
const Index = () => {
  const [count, setCount] = useState(0);

  const getList = (n) => {
    return Array.apply(Array, Array(n)).map((item, i) => ({
      id: i,
      name: "张三" + i
    }));
  };

  return (
    <>
      <Child getList={getList} />
      <button onClick={() => setCount(count + 1)}>count+1</button>
    </>
  );
};

const Child = ({ getList }) => {
  console.log("child-render");
  return (
    <>
      {getList(10).map((item) => (
        <div key={item.id}>
          id：{item.id}，name：{item.name}
        </div>
      ))}
    </>
  );
};
```

我们来尝试解读一下，当点击```“count+1”```按钮，发生了这样子的事：

```父组件render``` > ```子组件render``` > ```子组件输出"child-render"```

我们为了避免子组件做没必要的渲染，这里用了```React.memo```，如：

```JavaScript
// ...
const Child = React.memo(({ getList }) => {
  console.log("child-render");
  return (
    <>
      {getList(10).map((item) => (
        <div key={item.id}>
          id：{item.id}，name：{item.name}
        </div>
      ))}
    </>
  );
});
// ...
```

> 我们不假思索的认为，当我们点击```“count+1”```时，子组件不会再重渲染了。但现实是，还是依然会渲染，这是为什么呢？ 
>> 答：Reace.memo只会对props做浅比较，也就是父组件重新render之后会传入 不同引用的方法 getList，浅比较之后不相等，导致子组件还是依然会渲染。

这时候，```useCallback``` 就可以上场了，它可以缓存一个函数，当依赖没有改变的时候，会一直返回同一个引用。如：

```JavaScript
// ...
const getList = useCallback((n) => {
  return Array.apply(Array, Array(n)).map((item, i) => ({
    id: i,
    name: "张三" + i
  }));
}, []);
// ...
```

总结：如果子组件接受了一个方法作为属性，我们在使用 ```React.memo``` 这种避免子组件做没必要的渲染时候，就需要用 ```useCallback``` 进行配合，否则 ```React.memo``` 将无意义。

## ```useMemo```
> 与 ```vue``` 的 ```computed``` 类似，主要是用来避免在每次渲染时都进行一些高开销的计算，举个简单的例子。

不管页面 ```render``` 几次，时间戳都不会被改变，因为已经被被缓存了，除非依赖改变。

```JavaScript
// ...
const getNumUseMemo = useMemo(() => {
  return `${+new Date()}`;
}, []);
// ...
```

## ```useRef```
> 我们用它来访问```DOM```，从而操作```DOM```，如点击按钮聚焦文本框：

```JavaScript
const Index = () => {
  const inputEl = useRef(null);
  const handleFocus = () => {
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={handleFocus}>Focus</button>
    </>
  );
};
```

注意：返回的 ```ref``` 对象在组件的整个生命周期内保持不变。 它类似于一个 ```class``` 的实例属性，我们利用了它这一点。 动手滑到上面再看上面看那个有 ```useRef``` 的例子。

刚刚举例的是访问```DOM```，那如果我们要访问的是一个组件，操作组件里的具体DOM呢？我们就需要用到 ```React.forwardRef``` 这个高阶组件，来转发```ref```，如：

```JavaScript
const Index = () => {
  const inputEl = useRef(null);
  const handleFocus = () => {
    inputEl.current.focus();
  };
  return (
    <>
      <Child ref={inputEl} />
      <button onClick={handleFocus}>Focus</button>
    </>
  );
};

const Child = forwardRef((props, ref) => {
  return <input ref={ref} />;
});
```

## ```useImperativeHandle```
> ```useImperativeHandle``` 可以让我们在父组件调用到子组件暴露出来的属性/方法。如：

```JavaScript
const Index = () => {
  const inputEl = useRef();
  useEffect(() => {
    console.log(inputEl.current.someValue);
    // test
  }, []);

  return (
    <>
      <Child ref={inputEl} />
      <button onClick={() => inputEl.current.setValues((val) => val + 1)}>
        累加子组件的value
      </button>
    </>
  );
};

const Child = forwardRef((props, ref) => {
  const inputRef = useRef();
  const [value, setValue] = useState(0);
  useImperativeHandle(ref, () => ({
    setValue,
    someValue: "test"
  }));
  return (
    <>
      <div>child-value:{value}</div>
      <input ref={inputRef} />
    </>
  );
});
```

类似于```vue```在组件上用 ```ref``` 标志，然后 ```this.$refs.xxx``` 来操作dom或者调用子组件值/方法，只是```react```把它用两个钩子来表示”。

## ```useLayoutEffect```
在所有的 ```DOM``` 变更之后同步调用```effect```。可以使用它来读取 ```DOM``` 布局并同步 触发重渲染。在浏览器执行绘制之前，```useLayoutEffect``` 内部的更新计划将被同步刷新，也就是说它会阻塞浏览器绘制。所以尽可能使用 ```useEffect``` 以避免阻塞视觉更新。

[点我，看个例子，恍然大悟](https://codesandbox.io/s/useeffect-vs-uselayouteffect-ju1uj?file=/src/App.tsx)