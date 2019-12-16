# React

## createReactClass

create-react-class模块

**代替使用ES6 class定义react组件**

1. 声明默认属性：getDefaultProps()函数

2. 初始化state：getInitialState方法返回state

3. 自动绑定：不需要在constructor中显示调用bind(this)

4. mixin：解决完全不同的组件有相似的功能产生的“横切关注点cross-cutting concerns"问题

   ```jsx
   var SetIntervalMixin = {
       componentWillMount: function() {
           this.intervals = [];
       },
       setInterval: function() {
           this.intervals.push(setInterval.apply(null, arguments));
       },
       componentWillUnmount: function() {
           this.intervals.forEach(clearInterval);
       }
   };
   
   var TickTock = createReactClass({
       mixins: [SetIntervalMixin],
       getInitialState: function() {
           return {seconds: 0};
       },
       componentDidMount: function() {
           this.setInterval(this.tick, 1000);
       },
       tick: function() {
           return <p>React has been running for {this.state.seconds} seconds.</p>
       }
   })
   ```

## 高阶组件HOC

一种基于React的组合特性而形成的设计模式。参数为组件，返回值为新组件的函数。

不会修改传入的组件，也不会使用继承来复制其行为。是纯函数，没有副作用。

```jsx
function withSubscription(WrappedComponent, selectData) {
     class withSubscription extends React.Component {
        constructor(props) {
            super(props);
            this.handleChange = this.handleChange.bind(this);
            this.state = {
                data: selectData(DataSource, props)
            };
        }
        componentDidMount() {
            DataSource.addChangeListener(this.handleChange)；
        }
        componentWillUnmount() {
            DataSource.removeListener(this.handleChange);
        }
        handleChange() {
            this.setState({
                data: selectData(DataSource, this.props);
            })
        }
        render() {
            return <WrappedComponent data={this.state.data} {...this.props} />
        }
    }
    withSubscription.displayName = `withSubscription(${getDisplayName(WrappedComponent)})`;
    return withSubscription;
}

function getDisplayName(WrappedComponent) {
    return WrappedComponent.displayName || WrappedComponent.name || 'Component';
}
```

被包装组件接收来自容器组件的所有prop，和一个新的用于render的data prop;

- 不要改变原始组件。使用组合
- 将不相关的props传递给被包裹的组件：应该透传与自身无关的props。
- 最大化可组合性
- 包装显示名称以便调试：常见方式是用HOC包住被包装组件的显示名称，如WithSubscription(CommentList)
- 不要在render方法中使用：性能问题/状态丢失
- 务必复制静态方法：可以使用hoist-non-react-statics自动拷贝所有非React静态方法。
- Refs不会被传递：因为ref实际上并不是一个prop，像key，是由React专门处理的。如果添加到HOC返回组件中，则ref引用指向容器组件，而不是被包装组件。解决方案：React.forwardRef 

## cloneElement()

```js
React.cloneElement(element, [props], [...children])
```

以element元素为样板克隆并返回新的React元素。返回元素的Props是将新的props与原始元素的Props浅层合并后的结果。新的子元素将取代现有子元素，但来自原始元素的key和ref将被保留。

## static getDerivedStateFromProps()

react版本：16.0+   新的生命周期

在调用render方法之前调用，并且在初始挂载及后续更新时都会被调用。应返回一个对象来更新state，如果返回null则不更新任何内容；适用于state的值在任何时候都取决于props。

