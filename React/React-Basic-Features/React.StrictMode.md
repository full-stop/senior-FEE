# React. StrictMode 

`StrictMode` 是一个用于检测或标记其子组件树中潜在问题的工具，就像 `Fragment` 一样，它不会渲染任何真实的UI，它为其后代元素触发额外的检查和警告。
`StrictMode` 可以被使用在任何需要启用严格模式的地方：

``` jsx
<Heard>
    <React.StrictMode>
        <div>
            <ComponentOne/>
            <ComponentTow/>
        </div>
    </React.StrictMode>
</Heard>
```

## 作用

* 识别具有不安全生命周期的组件
* 有关旧式字符串ref用法的警告
* 关于已弃用的findDOMNode用法的警告
* 检测意外的副作用
* 检测遗留 context API

## 注意

在开发环境，为了检测组件的副作用，会重复两次调用以下生命周期方法：

* constructor()
* componentWillMount()
* render()
* setState(f=>{}) //第一个函数参数
* componentWillReceiveProps()
* getDerivedStateFromProps()
* shouldComponentUpdate()
* componentWillUpdate() 
* componentDidUpdate()

``` jsx
class ComponentOne extends React.Component {
  constructor(props) {
    super(props);
    SharedApplicationState.recordEvent('ExampleComponent');
  }
}
```

乍一看, 这段代码似乎没有问题。但是如果 `SharedApplicationState.recordEvent` 不是幂等, 那么多次实例化此组件可能会导致无效的应用程序状态。这种微妙的 bug 可能不会在开发过程中显现出来, 或者它可能会不一致, 因此被忽略。

通过有意的双调用方法 (如组件构造函数), 严格模式使得这样的行为更容易被发现。
