# `Component`中的`setState`理解 

## `setState`简介

[官方文档](https://reactjs.org/docs/react-component.html#setstate)

这是UI更新最常用的方法，合并新的state到现有的state。

api定义：

```js
setState<K extends keyof S>(
    state: ((prevState: Readonly<S>, props: P) => (Pick<S, K> | S | null)) | (Pick<S, K> | S | null),
    callback?: () => void
): void;
```

简单点就是：

```js
setState(updater, [callback])
```

## 参数详解

* `state`参数 : 可以是对象，可以是函数
* `callback`参数(可选) ：`state`更新完毕的回调函数
	
	> `setState()`的第二个参数是一个可选地回调函数，其将会在`setState`执行完成同时组件被重渲之后执行。通常，对于这类逻辑，我们推荐使用`componentDidUpdate`

## 常规使用

我们可以选择性地传递一个对象作为 `setState()`的第一个参数而不是一个函数：

```js
setState(stateChange, [callback])
```

`state`作为一个对象，包含0个或多个要更新的key。

```js
this.setState({  
  key1: value1, 
  key2: value2
});
```

## 状态更新可能是异步的
React 可以将多个`setState()` 调用合并成一个调用来提高性能。

看看下面这种情况：

```js
this.setState({  
  count: this.state.count + 1
});
this.setState({  
  count: this.state.count + 1
});
```

最后得到的`count`却是不可控的。因为`setState`不会立即改变`this.state`，而是挂起状态转换，调用`setState`方法后立即访问`this.state`可能得到的是旧的值。

> `setState`方法不会阻塞`state`更新完毕

第二个`setState`可能还没等待第一次的`state`更新完毕就开始执行了，所以最后`count`可能只加了1。

这时`setState`的第二个参数就派上用场了，第二个参数是state更新完毕的回调函数.

```js
this.setState({  
  count: this.state.count + 1
}, () => {
  this.setState({
    count: this.state.count + 1
  });
});
```

不过看起来很怪，`es6`中可以使用`Promise`更优雅的使用这个函数，封装一下`setState`

```js
function setStateAsync(nextState){  
  return new Promise(resolve => {
    this.setState(nextState, resolve);
  });
}
```

上面的例子就可以这样写

```js
async func() {  
  ...
  await this.setStateAsync({count: this.state.count + 1});
  await this.setStateAsync({count: this.state.count + 1});
}
```

顺眼多了。

## state参数为：函数方式

```js
(prevState, props) => stateChange
```

`nextState`也可以是一个`function`，称为状态计算函数，结构为`function(prevState, props) => newState`。

这个函数会将每次更新加入队列中，执行时通过当前的`state`和`props`来获取新的`state`。那么上面的例子就可以这样写:

```js
this.setState((state, props) => {  
  return {count: state.count + 1};
});
this.setState((state, props) => {  
  return {count: state.count + 1};
});
```

每次更新时都会提取出当前的state，进行运算得到新的state，就保证了数据的同步更新。

## 控制渲染

默认调用`setState`都会重新渲染视图，但是通过`shouldComponentUpdate()`函数返回`false`来避免重新渲染。

如果可变对象无法在`shouldComponentUpdate()`函数中实现条件渲染，则需要控制`newState`与`prevState`不同时才调用`setState`来避免不必要的重新渲染。

## 参考
[ReactJS setState 详解](https://juejin.im/entry/58b5285a5c497d00560fa290)

[官方文档](https://reactjs.org/docs/react-component.html#setstate)


