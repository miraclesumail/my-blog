---
title: 'react-deep-interview'
description: 'react interview'
pubDate: 'Jan 11 2026'
heroImage: '../../assets/blog-placeholder-3.jpg'
---

<style>
    .small-code pre {
    font-size: 13px !important;  /* 强制覆盖默认大小 */
    line-height: 1.5 !important; /* 调整行高，让排版更紧凑 */
    padding: 1rem !important;    /* 缩小边距（可选） */
  }

  p {
    margin: 0 !important;
  }

  .bold {
    font-weight: bolder;
  }

  li {
     font-size: 16px !important;  /* 强制覆盖默认大小 */
    line-height: 1.4 !important; /* 调整行高，让排版更紧凑 */
    margin-bottom: 8px;
  }
</style>

1. 🤖 [react合成事件](#sync)
2. ⚙️ [useActionsState的表单提交](#form)
3. 🔋 [startTransition的使用场景](#start)
4. 🤸 [context和use的新用法](#context)
5. 🕸️ [state妙用](#state)

#### <a name="sync">🤖 react合成事件</a>

1. <p class="bold">性能优化：事件委托</p>
   在原生开发中如果有100个li列表，每个都绑定onClick会消耗巨大内存

- <span class="bold">react的做法：</span>它不会把时间处理器挂载到真实的dom上，在17之前所有的事件都委托在document上， 17+之后事件被委托到根节点(root)上
- <span class="bold">好处：</span>无论有多少个按钮，react只会在根节点绑定一个监听器，当事件冒泡到根节点时根据事件源执行回调，减少了内存的消耗
- <span class="bold">标准化和一致性：</span>react将原生事件封装成syntheticEvent对象，不同浏览器都是写一样的逻辑代码

2. <p class="bold">原生事件与合成事件的顺序</p>

- 原生事件先执行，因为它直接挂载在dom上
- 合成事件后执行，因为它需要等冒泡到root节点

如果在原生事件中调用了stopPropagation,那么合成事件永远不会触发

3. <p class="bold">常见的合成事件用法：</p>

<div class="small-code">

```js
import {ChangeEventHandler, MouseEventHandler, MouseEvent, ChangeEvent} from 'react';

const onChange: ChangeEventHandler<HTMLInputElement> = (e) => {
    console.log(e.preventDefault());
  };

const onChange1 = (e: ChangeEvent<HTMLInputElement>) => {};

const onClick1: MouseEventHandler<HTMLDivElement> = (e) => {
    console.log(e.preventDefault());
  };

const onClick1 = (e: MouseEvent<HTMLButtonElement>) => {
   console.log(e.target)
  };


<input type='text' onChange={onChange} />
<div onClick={onClick1}></div>
```

</div>

#### <a name="form">⚙️ useActionsState的表单提交</a>

useActionsState常用于现代react的表单提交

<div class="small-code">

```js
import { useActionState } from 'react';
import { delay } from '@/app/utils';

type State = {
  age: number;
  name: string;
  err?: boolean;
};

export default function Page() {
  const [state, action, isPending] = useActionState<State, FormData>(updateForm, {age: 18, name: 'tom'});

  async function updateForm(prev: State, formData: FormData) {
     await delay(3);

     if (Math.random() < 0.5) {
      return {
        ...prev,
        err: true
      }
     }

     return {
      name: 'jay',
      age: prev.age + 1
     }
  }

  return <form action={action}>
    <input defaultValue={state.name}/>

    {
      state.err && <div>err</div>
    }
  </form>
}
```

</div>

#### <a name="start">🔋 startTransition的使用场景</a>

1. <p class="bold">它实现了什么？（用户体验层面）</p>
   在传统的 React 中，所有的 setState 都是平等的。如果你在一个搜索框里输入文字，同时触发了大量的列表过滤渲染：

- <span class="bold">以前：</span>浏览器会为了渲染巨大的列表而卡死，导致你输入的文字延迟显示（Input Lag）。
- <span class="bold">现在：</span>使用 startTransition 后，React 会优先保证输入框文字立刻显示，而列表的更新会在后台慢慢渲染，甚至如果用户输入太快，React 会抛弃中间过时的列表渲染，直接渲染最新的。

2. <p class="bold">可中断的渲染 (Interruptible Rendering)</p>

- <span class="bold">标记：</span>一旦标记为低优先级，React 就会启动“并发模式”进行渲染。
- <span class="bold">切片执行：</span>React 开始处理 Fiber 树。每处理几个节点，它都会调用我们之前聊过的 shouldYield() 检查主线程是否有高优先级的任务（比如用户点击、键盘输入）。
- <span class="bold">让路：</span>如果发现用户正在输入，React 会立即中断当前的 Transition 渲染。
- <span class="bold">恢复/重做：</span>等高优先级的输入响应完成后，React 再回来继续处理之前的低优先级任务，或者如果此时状态已经变了（用户输入了新词），React 会干脆丢弃之前做了一半的工作，直接开始新的一轮渲染

<div class="small-code">

```js
import { useState, useTransition } from 'react';

function FilterList({ name }) {
  const [query, setQuery] = useState('');
  const [highlight, setHighlight] = useState('');

  // isPending 表示转换任务是否正在后台进行
  const [startTransition, isPending] = useTransition();

  function handleChange(e) {
    const value = e.target.value;

    // 1. 紧急更新：确保输入框不卡顿
    setQuery(value);

    // 2. 非紧急更新：包裹在 transition 中
    startTransition(() => {
      // 假设这个过滤操作涉及几千个 DOM 节点的重排，非常耗时
      setHighlight(value);
    });
  }

  return (
    <div>
      <input type='text' value={query} onChange={handleChange} />

      {/* 3. 使用 isPending 优化用户体验 */}
      {isPending && <p>正在为您更新列表...</p>}

      <div style={{ opacity: isPending ? 0.5 : 1 }}>
        {names.map((name, i) => (
          <ListItem key={i} name={name} highlight={highlight} />
        ))}
      </div>
    </div>
  );
}
```

</div>

使用 useDeferredValue 实现

<div class="small-code">

```js
import { useState, useDeferredValue, useMemo } from 'react';

function SearchPage({ bigData }) {
  const [query, setQuery] = useState(''); // 紧急：输入框文字变化

  // 产生一个“延迟版”的 query
  const deferredQuery = useDeferredValue(query);

  // 只有当 deferredQuery 变化时，才重新计算繁重的过滤逻辑
  // 即使 query 变了，只要 deferredQuery 没变（还在后台排队），这里就不会执行
  const filteredList = useMemo(() => {
    return bigData.filter((item) => item.includes(deferredQuery));
  }, [deferredQuery, bigData]);

  return (
    <div>
      <input value={query} onChange={(e) => setQuery(e.target.value)} />

      {/* 视觉反馈：通过比较新旧值来判断是否正在“延迟中” */}
      <div style={{ opacity: query !== deferredQuery ? 0.5 : 1 }}>
        {filteredList.map((item) => (
          <div key={item}>{item}</div>
        ))}
      </div>
    </div>
  );
}
```

</div>

#### <a name="context">🤸 context和use的新用法</a>

现在可以直接使用Context作为根组件

<div class="small-code">

```js
// React 19 以前
<ThemeContext.Provider value={value}>...</ThemeContext.Provider>

// React 19 及以后
<ThemeContext value={value}>...</ThemeContext>

// 子组件用法还是一样
const context = useContext(ThemeContext);
```

</div>

1. <p class="bold">use 的特别之处：打破 Hooks 限制</p>

- <span class="bold">可以在条件语句中调用：</span>传统的 Hooks（如 useContext, useState）必须写在组件顶层，不能放在 if 或 for 里。但 use 可以。
- <span class="bold">可以在循环中调用：</span>可以在循环逻辑里动态决定读取哪个 Context。

<div class="small-code">

```js
function Settings({ showTheme }) {
  if (showTheme) {
    // ✅ 在 React 19 中这是合法的！
    // 如果 showTheme 为 false，这行代码就不会运行
    const theme = use(ThemeContext);
    return <div>当前主题：{theme}</div>;
  }
  return <div>设置已隐藏</div>;
}
```

</div>

2. <p class="bold">它不仅仅用于 Context，还能处理 Promise</p>

<div class="small-code">

```js
import { use, Suspense } from 'react';

// 在父组件里传promise
function Message({ messagePromise }) {
  // ✅ 直接在渲染逻辑里“等待”Promise 结果
  const content = use(messagePromise);
  return <p>消息内容: {content}</p>;
}

export default function App() {
  const promise = fetchMessage(); // 这是一个异步请求
  return (
    <Suspense fallback={<p>正在加载...</p>}>
      <Message messagePromise={promise} />
    </Suspense>
  );
}
```

</div>

#### <a name="state">🕸️ state妙用</a>

1. <p class="bold">观察一下代码的输出</p>
<div class="small-code">

```js
export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber(number + 1);
          setNumber(number + 1);
          setNumber(number + 1);
        }}
      >
        +3
      </button>
    </>
  );
}
```

</div>

为什么打印的是1，是闭包还是batchUpdate。。。

2. <p class="bold">这样修改之后的输出</p>
<div class="small-code">

```js
export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber((n) => n + 1);
          setNumber((n) => n + 1);
          setNumber((n) => n + 1);
        }}
      >
        +3
      </button>
    </>
  );
}
```

</div>

当你传递一个函数给 setNumber 时，React 会把它放入一个队列。在渲染前，React 会遍历这个队列，把上一个计算出的结果作为参数传给下一个函数，从而确保累加。

3. <p class="bold">这样修改之后的输出</p>

<div class="small-code">

```js
export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber(number + 5);
          setNumber((n) => n + 1);
          setNumber(3);
        }}
      >
        Increase the number
      </button>
    </>
  );
}
```

</div>
思考上述代码输出几, 最后一行是 setNumber(3);他会进行覆盖最后是3

4. <p class="bold">思考下面这个输出</p>

<div class="small-code">

```js
export default function RequestTracker() {
  const [pending, setPending] = useState(0);
  const [completed, setCompleted] = useState(0);

  async function handleClick() {
    setPending(pending + 1);
    await delay(3000);
    // 这里的pending第一次取到的是0
    setPending(pending - 1);
    setCompleted(completed + 1);
  }

  return (
    <>
      <h3>
        Pending: {pending}
      </h3>
      <h3>
        Completed: {completed}
      </h3>
      <button onClick={handleClick}>
        Buy     
      </button>
    </>
  );
}
```

</div>
连续点击会怎么样，提示:setPending(pending + 1)和setPending(pending - 1)的值永远相同，在同一次点击中;