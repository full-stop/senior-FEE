# React-Life-Cycle

## 旧版生命周期

![img](. /images/old-life-cycle. jpg)

React 的生命周期主要可以分为三种阶段：挂载时、更新时、卸载时。

**挂载时(Mounting)**
当组件初次挂载时，会依次触发以下的生命周期函数：

1. constructor()
2. componentWillMount()
3. render()
4. componentDidMount()

**更新时(Updating)**
当组件的状态(state)以及属性(props)发生变更时，会触发组件的更新生命周期，其顺序如下：

1. componentWillReceiveProps()
2. shouldComponentUpdate()
3. componentWillUpdate()
4. render()
5. componentDidUpdate()

**卸载时(UnMounting)**
当组件执行完毕被销毁时，会触发该类型的生命周期

1. componentWillUnmount()

## 生命周期流程

React 生命周期的执行过程类似于 `Express` 的洋葱模型，也类似于 React Diff 算法中的深度优先。

当父组件执行到 render 的时候会进入子组件的整个生命周期，当子组件的整个生命周期都执行完毕则会跳出到父组件 render 之后的生命周期继续执行，如果存在多层嵌套的父子孙组件，则依次类推。

当父子组件 Mounting 时：

![img](. /images/Mounting. jpg)

当父子组件 Updating 时:

![img](. /images/Updating. jpg)

需要注意的是：

* 当父组件 `Updating` 时，其子组件也会 `Updating` 。
* 当子组件 `Updating` 时，其父组件不受任何影响。
* setState() 方法的第二个参数即回调函数会在整个 Updating 生命周期都执行完毕后才会执行。

## 新版生命周期

![img](. /images/new-life-cycle. jpg)

新版的生命周期对比于旧版的生命周期，主要废除了以下三个生命周期方法：

* componentWillMount()
* componentWillReceiveProps()
* componentWillUpdate()

废除的原因是因为这三个生命周期经常会被误解和滥用。
对比于废弃了的生命周期，React 新增了两个生命周期，分别是：

* getDerivedStateFromProps()
* getSnapshotBeforeUpdate()

新版本生命周期的每个阶段：

**挂载时(Mounting)**

1. constructor()
2. static getDerivedStateFromProps()
3. render()
4. componentDidMount()

**更新时(Updating)**

1. static getDerviedStateFromProps()
2. shouldComponentUpdate()
3. render()
4. getSnapshotBeforeUpdate()
5. componentDidUpdate()

下面对新版本生命周期进行详细讲解

**getDerivedStateFromProps(nextProps, prevState)**

翻译过来的意思就是源于父组件的 props 来获得当前组件的 state。而专业一点解释则是将父组件传递的 props 映射到当前组件的 state 上，这样子组件内部就不需要每次都是通过 `this.props.xxx` 获取，而是直接从自身的状态上读取。

在 `getDerivedStateFromProps` 生命周期中根据父组件的 props 派发组件状态，如果父组件的 props 没有发生改变，则可以返回 null，需要注意的派发组件状态是合并覆盖而不是单纯的覆盖。

``` jsx
static getDerivedStateFromProps(nextProps,preveState){
    if(nextProps.gaid != preveState.gaid){
        return {
            gaid:nextProps.gaid
        }
    }

    return null; //表示父组件的props没有变化无需更改子组件的状态。
}
```

**shouldComponentUpdate(nextState, nextProps)**
该生命周期只能用于 `Component` 类型的组件中，返回 `false` 拦截组件渲染。不在执行 `render` 及之后的生命周期方法。
对比于 `diff` 算法只是用于确定新旧节点树之间的区别，然后进行节点更新，其特点就是替代了传统 jquery 时期手动 DOM 节点的操作，因此 `diff` 算法的主要功能还是保证 DOM 节点不会被随意更新，并不能确保一些 `updating` 状态下的生命周期方法不会再被执行一遍，因此为了更大的提升性能，通常我们都会在此生命周期方法中进行 `state` 或 `props` 变更的判断。

``` jsx
shouldComponentUpdate(nextState,nextProps){
    return (!equal(this.props,nextProps) || !equal(this.state,nextState))
}
```

`React.PureComponent` 已经自带了该生命周期方法，但只做到了对基本数据类型值的变更判断，如果是引用类型的数据变更，哪怕对象的值是完全相同的依然会触发 `updating` 阶段的生命周期方法。

**getSnapshotBeforeUpdate(preveState, prevProps)**
该生命周期处于 `render` 之后 `componentDidUpdate` 之前调用，所以在该生命周期中可以获取即将销毁的上一次DOM节点。它接收两个参数，分别表示之前的状态与之前的属性（基于 render 渲染之后来理解），该方法可以返回一个值作为 `componentDidUpdate` 生命周期的第三个参数，如果不想有返回值，可以直接返回 `null` 。
注意，该生命周期方法必须要与 `componentDidUpdate` 一起使用，否则控制台会报错。

``` jsx
getSnapshotBeforeUpdate(preveState,prevProps){
    return null;
}

componentDidUpdate(prevState,prevProps,sanpshotValue){
    //....
}
```

**componentDidUpdate(prevState, prevProps, sanpshotValue)**

该生命周期位于 `render` 与 `getSanpshotBeforeUpdate` 之后，它接收三个参数，分别是：prevState、prevProps、sanpshotValue。
在该生命周期中，我们可以进行以下操作：

* 发起请求获取数据。
* 操作DOM节点。
* setState

但需要注意的是要进行合理的 `if` 判断以防止进入死循环。

## 相关问题

### shouldComponentUpdate 的调用时机

如果明确知道组件不会轻易被修改，那么使用 `PureComponent` 类型的组件，进行自动的浅比较即可。
如果明确知道组件会被修改，那么使用 `Component` 类型的组件再配合 `shouldComponentUpdate` 方法来优化性能将是最佳的选择。
最后为什么不能在 `shouldComponentUpdate` 中进行深度比较？很简单，因为深度比较所耗费的时间都会比整个 `updating` 阶段执行完毕还要长，明显得不偿失。

### 如果 setState 更新的值不变，组件还会触发所有 Updating 阶段的钩子吗？

答案是要分清当前组件所继承的组件类型，如果是 `Component` 类型，那么不论值有没有变都会被执行一遍，这个时候可以在 `shouldComponentUpdate` 钩子中进行优化，如果是 `PureComponent` 类型，因为内部自带了浅比较，那么基本数据类型的值不变时则不会更新，但对于引用类型的值，一旦对象的引用发生了变化，那么便会触发所有钩子的执行。

### getDerivedStateFromProps 与组件内的 this 指向问题。

由于 `static getDerivedStateFromProps` 是组件的静态方法，因此不会被组件的实例对象所继承，从而该生命周期钩子内部无法通过 `this` 对象获取组件实例本身。更无法通过调用 `this.setState()` 方法来改变组件的状态。

为了规避这个限制，通常我们在 `getDerivedStateFromProps` 生命周期中调用的方法要么遵循单一原则，是一个独立的函数，要么也作为一个静态方法通过组件的类名来调用。

``` jsx
import Utils from './tools/utils.js';

class Demo extends React.Component{

    static getDerivedStateFromProps(nextState,nextProps){
        Utils.execFn1();
        Demo.execFn2();
    }

    static execFn2(){
        //....
    }

}

```

### 根据外部 props 的变更来重新请求数据或变更状态

**使用 componentWillReceiveProps**
注：该生命周期方法已经不被建议使用。

**使用 getDerivedStateFromProps + componentDidUpdate**
在 `getDerivedStateFromProps` 中基于 nextProps 与 preveState 的值进行判断，然后加一个标记，最后在 `componentDidUpdate` 中判断标记是否成立来获取数据。

**只用 componentDidUpdate**
直接在 `componentDidUpdate` 中判断当前的 `this.props.xxx` 与 `this.state.xxx` 的值是否相同，如果不同则请求数据，并将 props 中的值覆盖到 state 中。

**使用key**
如果很难判断组件什么时机下需要基于props变更从新请求数据，那么直接重新渲染组件无疑更有效。


