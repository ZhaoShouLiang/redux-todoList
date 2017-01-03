# redux-todoList

### 3. 理解 Redux 的核心概念

### 3.1 Action & Action Creator

在 Redux 中，改变 State 只能通过 action，它是 store 数据的唯一来源。一般来说你会通过 store.dispatch() 将 action 传到 store。。并且，每一个 action 都必须是 Javascript 的简单对象，例如：

```
{
  type: 'ADD_TODO',
  text: 'Learn Redux'
}
```

### 3.2 Reducer

我们先来看一下 Javascript 中 Array.prototype.reduce 的用法：

```js
const initState = '';
const actions = ['a', 'b', 'c'];
// 传入当前的 state 和 action ，返回新的 state
const newState = actions.reduce((curState, action) => curState + action);
console.log( newState );
```

对应的理解，Redux 中的 reducer 是一个纯函数，传入state和action，返回一个新的state tree，简单而纯粹的完成某一件具体的事情，没有依赖，简单而纯粹是它的标签。

### 3.3 Store

Store 就是用来维持应用所有的 state 树 的一个对象。 改变 store 内 state 的惟一途径是对它 dispatch 一个 action。

Store 是一个具有以下四个方法的对象：

- getState()
- dispatch(action)
- subscribe(listener)
- replaceReducer(nextReducer)

#### 3.3.1 getState()

返回应用当前的 state 树。 它与 store 的最后一个 reducer 返回值相同。

返回值：应用当前的 state 树。

#### 3.3.2 dispatch(action)

dispatch 分发 action。这是触发 state 变化的惟一途径。

会使用当前 getState() 的结果和传入的 action 以同步方式的调用 store 的 reduce 函数。返回值会被作为下一个 state。从现在开始，这就成为了 getState() 的返回值，同时变化监听器(change listener)会被触发。

在 Redux 里，只会在根 reducer 返回新 state 结束后再会调用事件监听器，因此，你可以在事件监听器里再做 dispatch。惟一使你不能在 reducer 中途 dispatch 的原因是要确保 reducer 没有副作用。如果 action 处理会产生副作用，正确的做法是使用异步 action 创建函数。

#### 3.3.3 subscribe(listener)

添加一个变化监听器。每当 dispatch action 的时候就会执行，state 树中的一部分可能已经变化。这是一个底层 API。多数情况下，你不会直接使用它，会使用一些 React（或其它库）的绑定。


### 4.1 createStore

调用方式：createStore(reducer, [initialState])

创建一个 Redux store 来以存放应用中所有的 state，应用中应有且仅有一个 store。 这个API返回一个store，这个store中保存了应用所有 state 的对象。改变 state 的惟一方法是 dispatch action。你也可以 subscribe 监听 state 的变化，然后更新 UI。
### 4.2 combineReducers

调用方式：combineReducers(reducers)

随着应用变得复杂，需要对 reducer 函数进行拆分，拆分后的每一块独立负责管理 state 的一部分。把一个由多个不同 reducer 函数作为 value 的 object，合并成一个最终的 reducer 函数，然后就可以对这个 reducer 调用 createStore。

### 4.3 applyMiddleware

调用方式：applyMiddleware(...middlewares)

使用包含自定义功能的 middleware 来扩展 Redux 是一种推荐的方式。Middleware 可以让你包装 store 的 dispatch 方法来达到你想要的目的。同时， middleware 还拥有“可组合”这一关键特性。多个 middleware 可以被组合到一起使用，形成 middleware 链。其中，每个 middleware 都不需要关心链中它前后的 middleware 的任何信息。
### 4.4 bindActionCreators

调用方式：bindActionCreators(actionCreators, dispatch)

惟一使用 bindActionCreators 的场景是当你需要把 action creator 往下传到一个组件上，却不想让这个组件觉察到 Redux 的存在，而且不希望把 Redux store 或 dispatch 传给它。

### 4.5 compose

调用方式：compose(...functions)

compose 用来实现从右到左来组合传入的多个函数，它做的只是让你不使用深度右括号的情况下来写深度嵌套的函数，仅此而已。


## 5. 使用 React-redux 连接 react 和 redux

### 5.1 没有react-redux的写法

封装一个组件，将组件和Redux做基本的组合
```js
import { createStore } from 'redux';
import React, { Component } from 'react';
import ReactDOM from 'react-dom';

// reducer 纯函数，具体的action执行逻辑
const counter = (state = 0, action) => {
  switch (action.type) {
      case 'INCREMENT':
        return state + 1;
      case 'DECREMENT':
        return state - 1;
      default:
        return state;
  }
}

const store = createStore(counter);

// Counter 组件
class Counter extends Component {
  render(){
    return (
      <div>
        <h1>{this.props.value}</h1>
        <button onClick={this.props.onIncrement}>点击加1</button>
        <button onClick={this.props.onDecrement}>点击减1</button>
      </div>
    )
  }
}

const PureRender = () => {
  ReactDOM.render(
      <Counter
        value={store.getState()}
        onIncrement={ () => store.dispatch({type: "INCREMENT"}) }
        onDecrement={ () => store.dispatch({type: "DECREMENT"}) }
      />, document.getElementById('app')
  );
}

// store subscribe 订阅或是监听view（on）
store.subscribe(PureRender)
PureRender()
```

### 5.2 React-redux 提供的 connect 和 Provider

<Provider store> 使组件层级中的 connect() 方法都能够获得 Redux store。正常情况下，你的根组件应该嵌套在 <Provider> 中才能使用 connect() 方法。

```js
ReactDOM.render(
  {/*  使组件层级中的 connect() 方法都能够获得 Redux store */}
  <Provider store={store}>
    {/* 这里传入的组件MyRootComponent是组件层级的根组件 */}
    <MyRootComponent />
  </Provider>,
  rootEl
);
connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])
```

connect方法是用来连接 React 组件与 Redux store，连接操作不会改变原来的组件类，反而返回一个新的已与 Redux store 连接的组件类。

使用react-redux的一个简单完整示例

```js
import React, { Component, PropTypes } from 'react'
import ReactDOM from 'react-dom'
import { createStore } from 'redux'
import { Provider, connect } from 'react-redux'

// 这是一个展示型组件 Counter
class Counter extends Component {
  render() {
    const { value, onIncreaseClick } = this.props
    return (
      <div>
        <span>{value}</span>
        <button onClick={onIncreaseClick}>戳我加1</button>
      </div>
    )
  }
}

Counter.propTypes = {
  value: PropTypes.number.isRequired,
  onIncreaseClick: PropTypes.func.isRequired
}

// Action
const increaseAction = { type: 'increase' }

// Reducer
function counter(state = { count: 0 }, action) {
  let count = state.count
  switch (action.type) {
    case 'increase':
      return { count: count + 1 }
    default:
      return state
  }
}

// Store
let store = createStore(counter)

// Map Redux state to component props
function mapStateToProps(state) {
  // 这里拿到的state就是store里面给的state
  console.log(state);
  return {
    value: state.count
  }
}

// Map Redux actions to component props
function mapDispatchToProps(dispatch) {
  // dispatch(action) { }
  return {
    onIncreaseClick: () => dispatch(increaseAction)
  }
}

class App extends Component {
  render() {
    // store里的state经过connect连接后给了根组件的props
    console.log(this.props);
    return (
      <div>
        <h1>学习使用react-redux</h1>
        <Counter {...this.props} />
      </div>
    )
  }
}

// Connected Component
let RootApp = connect(
  mapStateToProps,
  mapDispatchToProps
)(App)

ReactDOM.render(
  <Provider store={store}>
    <RootApp />
  </Provider>,
  document.getElementById('app')
)

```

实际应用中，connect这个部分会比较复杂
