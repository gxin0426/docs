#### redux原理

当然，让我们用一个简单的故事来理解 React 中的 Redux 如何工作：

想象你在经营一家快餐店，你需要处理顾客的订单（Actions），更新订单板上的信息（State），并确保所有员工（Components）都知道最新的订单信息。

在没有 Redux 的情况下，每当有新的订单来时，你会手写订单信息，并走到每个员工那里告诉他们。这在忙碌的时候很容易出错，因为员工可能忘记了最新的信息，或者信息传递得不够快。

引入 Redux 后，你有了一个中央订单板（Store），所有的订单信息都在这里更新。这个订单板有三个主要部分：

1. **订单（Actions）**：
   - 每当顾客点餐时，你会写下他们想要的食物，并将这个请求放到一个特定的地方（Dispatching an action）。

2. **订单处理规则（Reducers）**：
   - 你有一套规则来决定如何更新订单板上的信息。例如，“如果顾客要加薯条，就在订单板上加上薯条。”这些规则确保订单板的信息始终是正确的。

3. **订单板（Store）**：
   - 所有的订单更新都发生在这里。无论是添加新的订单还是修改订单，都会根据你的订单处理规则来更新板上的信息。

现在，所有员工都可以直接看到这个中央订单板上的信息，知道他们需要准备什么。如果他们需要更新订单信息，比如顾客改主意不要薯条了，员工就会根据规则更新订单板。

在 React 应用中：

- **Actions**：用户的行为，比如点击按钮增加计数器。
- **Reducers**：函数，定义如何根据 Actions 更新应用的状态。
- **Store**：保存整个应用状态的地方，相当于中央订单板。

##### 实际例子

让我们把故事变成代码。假设我们有一个计数器应用，它只有一个数字和两个按钮，一个增加数字，一个减少数字。

1. **Actions**：
```javascript
// 这个 Action 告诉 Redux 我们要增加计数器的数字
const increaseAction = { type: 'INCREASE' };
// 这个 Action 告诉 Redux 我们要减少计数器的数字
const decreaseAction = { type: 'DECREASE' };
```

2. **Reducer**：
```javascript
// 这个函数定义了订单板应该如何根据顾客的订单（Actions）来更新
function counter(state = 0, action) {
  switch (action.type) {
    case 'INCREASE':
      return state + 1;
    case 'DECREASE':
      return state - 1;
    default:
      return state;
  }
}
```

3. **Store**：
```javascript
import { createStore } from 'redux';
// 创建订单板
let store = createStore(counter);

// 打印初始状态
console.log(store.getState());

// 每次状态更新时，打印日志
// 注意 subscribe() 返回一个函数用来注销监听器
const unsubscribe = store.subscribe(() => console.log(store.getState()));

// 发送订单（Dispatching actions）
store.dispatch(increaseAction); // 输出：1
store.dispatch(increaseAction); // 输出：2
store.dispatch(decreaseAction); // 输出：1

// 停止监听状态更新
unsubscribe();
```

这里的 `store.getState()` 就像是查看订单板，`store.dispatch()` 就像是放一个新订单到订单板上。

这就是 Redux 的基本工作原理。希望这次解释更清晰易懂！

#### useState useEffect useRef 用法

在 React 中，`useState`, `useEffect`, 和 `useRef` 是常用的 Hooks，每个都有其特定的用途和应用场景。以下是对每个 Hook 的解释和示例，以及一些易于记住的方法。

##### 1. useState

###### 用法
- `useState` 用于在函数组件中添加状态。
- 它返回一个状态变量和一个用于更新该状态的函数。

###### 示例
假设我们要制作一个简单的计数器：

```javascript
import React, { useState } from 'react';

function Counter() {
    const [count, setCount] = useState(0);

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

在这个例子中，`count` 是状态变量，`setCount` 是用于更新 `count` 的函数。每当按钮被点击时，`count` 的值增加 1。

###### 记忆方法
- **"State":** 用 `useState` 来 "记住" 组件的状态。
- **"[value, setValue]":** 类似数组解构，第一个是值，第二个是设置这个值的函数。

##### 2. useEffect

###### 用法
- `useEffect` 用于处理副作用，如数据获取、订阅或手动更改 DOM。
- 它接受一个函数（副作用函数）作为参数，该函数在组件渲染后运行。

###### 示例
假设我们要在组件加载后获取数据：

```javascript
import React, { useState, useEffect } from 'react';

function MyComponent() {
    const [data, setData] = useState(null);

    useEffect(() => {
        fetchData().then(setData);
    }, []); // 空数组表示这个 effect 仅在组件挂载时运行一次

    // ...
}
```

在这个例子中，`useEffect` 用于在组件首次渲染后调用 `fetchData` 函数获取数据。

###### 记忆方法
- **"Effect":** 用 `useEffect` 来处理副 "效应"。
- **"[] 一次, [deps] 多次, 无则每次":** 依赖数组控制 effect 的执行频率。

##### 3. useRef

###### 用法
- `useRef` 返回一个可变的 ref 对象，其 `.current` 属性被初始化为传递的参数（`initialValue`）。
- `useRef` 常用于访问 DOM 元素，但也可以用来保存任何可变值。

###### 示例
假设我们需要在表单中聚焦一个输入框：

```javascript
import React, { useRef, useEffect } from 'react';

function Form() {
    const inputRef = useRef();

    useEffect(() => {
        inputRef.current.focus();
    }, []);

    return (
        <input ref={inputRef} type="text" />
    );
}
```

在这个例子中，`useRef` 用于创建一个 ref 对象 `inputRef`，然后将其附加到输入框上。`useEffect` 确保在组件加载后输入框自动获得焦点。

###### 记忆方法
- **"Ref":** 用 `useRef` 来 "引用" DOM 元素或保存任意可变值。
- **"Ref.current":** 访问 ref 的 `.current` 属性来获取当前的引用值。

这些 Hooks 在 React 开发中非常常见，理解它们的基本用法和应用场景对于构建有效和高效的 React 应用至关重要。





#### npm用法

`npm install` 命令用于安装Node.js项目中的依赖项。这个命令可以接受多个参数和选项来调整其行为。以下是一些常用的参数和选项：

1. **无参数** - 直接在项目根目录运行 `npm install`（或`npm i`，`i`是`install`的简写），将会安装`package.json`文件中列出的所有依赖。

2. **包名** - 你可以指定一个包名来安装最新版本的那个包，例如 `npm install express`。

3. **-g** - 全局安装一个包，而不是安装到当前项目目录下，例如 `npm install -g express`。

4. **--save** - 在安装依赖时更新`package.json`文件，并将依赖添加到`dependencies`中。在npm 5.0.0及以上版本中，默认行为已经改为保存依赖项。

5. **--save-dev** - 和`--save`相似，但是将依赖项添加到`devDependencies`中，这些依赖通常只在开发时需要，例如测试框架或构建工具。

6. **--save-exact** - 安装精确版本的依赖包，不使用版本范围。

7. **--no-save** - 安装依赖包但不将其保存到`package.json`文件中。

8. **--save-optional** - 将依赖项保存到`optionalDependencies`中，这些依赖项不是必须的。

9. **--save-peer** - 如果你在开发一个可以被其他包依赖的包，使用这个选项可以指定peer依赖。

10. **--production** - 只安装`dependencies`中的依赖，忽略`devDependencies`。

11. **--force** 或 **-f** - 强制重新下载依赖，即使它们已经在本地缓存中。

12. **--dry-run** - 模拟安装过程，显示将要发生的变化，但不实际安装依赖。

13. **--silent** 或 **--quiet** - 减少安装过程中输出的信息。

这些参数和选项可以根据需要组合使用来达到你想要的效果。注意，随着npm版本的更新，这些参数和选项也可能会有变化。



#### await和async

```js
const onConfirm = async (data) => {
    console.log('删除点击了', data)
    
    await delArticleAPI(data.id)
    //在这等待delArticleAPI完成并且返回promise，才继续执行下面的代码
    setReqData({
      ...reqData
    })
} 
```

`await delArticleAPI(data.id)`: `await`用于等待一个`Promise`的结果。`delArticleAPI`很可能是一个返回`Promise`的异步API调用，用于删除一个文章。通过在这个调用前加上`await`，你可以“暂停”函数的执行，直到`Promise`被解决（即API请求完成）。这样做的好处是代码的后续部分，像是`setReqData({...reqData})`调用，将在`delArticleAPI`完成后执行，这可以保证状态更新是在文章删除操作之后进行的。

##### 第二种写法

```js
const onConfirm = (data) => {
  console.log('删除点击了', data);
  delArticleAPI(data.id)
    .then(() => {
      setReqData({
        ...reqData
      });
    })
    .catch((error) => {
      console.error('删除文章出错:', error);
    });
};
```

