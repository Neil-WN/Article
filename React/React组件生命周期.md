# 系统了解React组件生命周期

## 前言

组件的生命周期是各大框架中的重要一环，本文主要介绍React的组件生命周期，以及v16.3后所出现的新的生命周期钩子函数，希望对新手同学有帮助。

## 1. 生命周期的三个阶段

React组件的生命周期分为三个阶段：

- Mounting：加载过程，第一次把组件渲染到DOM树的过程；
- Updating：更新过程，组件进行渲染更新DOM树的过程；
- Unmounting：卸载过程，组件从DOM树中移除的过程。

React在这三个阶段中共有8个钩子函数供我们调用，其中render函数会在Mounting和Updating时都会调用。下面介绍各阶段中使用的钩子函数。本文主要使用ES6的class关键字来定义React组件，当然你也可以通过create-react-class模块进行组件的创建。



### 1.1. Mounting

组件加载的时候会先进行state和props的初始化，然后执行componentWillMount、render 和componentDidMount。组件卸载之前，Mounting阶段的钩子函数只会被执行一次。

#### constructor(props)

构造函数。该函数用于组件加载前调用，主要用于state的初始化和事件方法的绑定。

```javascript
constructor (props) {
    super(props)
    
    // 初始化state
    this.state = {
        data: 'React组件生命周期'
    }
    
    this.eventHandle = this.eventHandle.bind(this)	//绑定事件
}
```

对于state的初始化声明，如果不使用ES6的Class语法，React也提供了支持ES5的方法：getInitialState()，可以在组件加载前先调用该方法进行state初始化。

```javascript
getInitialState: function () {
    return {
        data: 'React组件生命周期'
    }
}
```

关于constructor()，有几点需要注意的地方：

（1）当不需要初始化state和事件绑定时，constructor()方法可以不用声明；

（2）使用constructor()方法时，方法内的super()必须放在其他语句之前，否则会导致报错；

（3）除了在constructor()方法中直接使用`this.state = {}`进行直接的state初始化声明，在其他的方法中都不能这样使用，而需要使用`this.setState()`进行赋值。

#### componentWillMount()

预加载函数。该函数会在组件渲染到页面前调用，由于是在render函数前执行，所以在此处改变state，不会发生重新渲染。

```javascript
componentWillMount () {
    console.group('---------组件预加载---------')
    console.log('state:', this.state)
    console.log('props:', this.props)
}
```

#### render()

渲染函数。该函数是组件中必不能省略掉函数，且必须有返回值（JSX），当返回null或者false则表示不渲染DOM元素。

```javascript
render () {
    console.group('---------组件渲染---------')
    console.log('state:', this.state)
    console.log('props:', this.props)
    return (
       <div ref='text'>{this.props.text}</div>
    )
}
```

注意，切记不能在render()中执行this.setState()，因为this.setState()会触发render()，这样会导致循环调用，使得浏览器内存占满后崩溃。  

#### componentDidMount()

加载成功函数。该函数在render()完成之后调用，此时页面中存在真实的DOM元素，可以在这里进行DOM操作、更改state或者请求异步数据。

```javascript
componentDidMount () {
    console.group('---------组件加载完成---------')
    console.log('state:', this.state)
    console.log('props:', this.props)
    console.log('refs:', this.refs.text.innerText)
}
```



### 1.2. Updating

当组件DOM加载完成后，props或者state的改变，会引起组件的更新，更新的过程会触发以下几种钩子函数：

#### componentWillReceiveProps(nextProps)

该函数会在Mounting过程结束后，当接收到新的props时触发，nextProps代表要更新的props。组件在接收新的props时，会存在props值没有发生变化的情况，而此时componentWillReceiveProps(nextProps)同样会触发，因此建议比较一下this.props和nextProps的值，确保是否需要进行组件重新渲染。

```javascript
componentWillReceiveProps (nextProps) {
    console.group('---------组件props更新---------')
    console.log('state:', this.state)
    console.log('props:', this.props)
    console.log('nextProps:', nextProps)
}
```

#### shouldComponentUpdate(nextProps, nextState)

该函数会在componentWillReceiveProps(nextProps)后触发。shouldComponentUpdate(nextProps, nextState)接收两个参数，前者表示的是要更新的props，后者则表示要更新的state。

该函数返回的是一个boolean，用来判断组件是否要更新，true表示要更新，false表示不作更新。当没有手动编写shouldComponentUpdate(nextProps, nextState)函数时，该函数默认返回true。

```javascript
shouldComponentUpdate (nextProps, nextState) {
    console.group('---------组件是否更新---------')
    console.log('state:', this.state)
    console.log('props:', this.props)
    console.log('nextState:', nextState)
    console.log('nextProps:', nextProps)
    return true
}
```

#### componentWillUpdate(nextProps, nextState)

该函数会在shouldComponentUpdate(nextProps, nextState)触发后，同样是接收两个参数。

```javascript
componentWillUpdate (nextProps, nextState) {
    console.group('---------组件预更新---------')
    console.log('state:', this.state)
    console.log('props:', this.props)
    console.log('nextState:', nextState)
    console.log('nextProps:', nextProps)
}
```

#### render()

在组件的更新阶段，同样会触发render()函数。在Updating过程中，主要是通过改变props和state来更新组件的，props可以通过组件间的传值改变，而state则可以通过setState()方法来更新，并触发render()函数来重新渲染。

同Mounting阶段一样，在render()函数中，不能使用setState()方法。

#### componentDidUpdate(prevProps, prevState)

该函数会在render()完成后触发。componentDidUpdate(prevProps, prevState)接收两个参数，前者表示的是更新前的props，后者则表示更新前的state。

```javascript
componentDidUpdate (prevProps, prevState) {
    console.group('---------组件更新完成---------')
    console.log('state:', this.state)
    console.log('props:', this.props)
    console.log('prevState:', prevState)
    console.log('prevProps:', prevProps)
}
```



### 1.3. Unmouting

#### componentWillUnmount()

该函数会在组件卸载之前调用。该函数可以在组件销毁之前进行一些必要的清理工作，如timer、interval等。



### 1.4. 小结一下

现在我们已经了解React组件的整个生命周期，让我们再重新梳理一下：

（1）组件加载时，会执行componentWillMount、render 和componentDidMount函数。该过程只会执行一次；
（2）当组件的props或者state更新的时候，组件会进入更新过程，依次执行componentWillReceiveProps（props更新时才会触发）、shouldComponent-
Update、componentWillUpdate、render 和 componentDidUpdate。在整个组件加载完成后，只要props或state更新后，该过程就会执行，因此千万不要在shouldComponentUpdate、componentWillUpdate和render函数中进行setState()操作，这样会引起循环渲染问题；
（3）componentWillUnmount钩子函数是组件卸载前执行的，与组件加载一样，整个生命周期只会执行一次；
（4）render函数是最特殊的一个，它是不仅在组件加载过程中调用，也会在更新过程中调用，甚至是在无生命周期（无状态组件）中也是必不可缺的函数。

以下附上完整的测试代码：

```javascript
class LifeCycle extends React.Component {
    constructor (props) {
        super(props)
        console.group('---------组件初始化state---------')
        console.log('state:', this.state)
        console.log('props:', this.props)
        this.state = {
            data: 'React组件生命周期'
        }
    }
    componentWillMount () {
        console.group('---------组件预加载---------')
        console.log('state:', this.state)
        console.log('props:', this.props)
    }
    componentDidMount () {
        console.group('---------组件加载完成---------')
        console.log('state:', this.state)
        console.log('props:', this.props)
        console.log('refs:', this.refs.text.innerText)
    }
    componentWillReceiveProps (nextProps) {
        console.group('---------组件props更新---------')
        console.log('state:', this.state)
        console.log('props:', this.props)
        console.log('nextProps:', nextProps)
        console.log('refs:', this.refs.text.innerText)
    }
    shouldComponentUpdate (nextProps, nextState) {
        console.group('---------组件是否更新---------')
        console.log('state:', this.state)
        console.log('props:', this.props)
        console.log('nextState:', nextState)
        console.log('nextProps:', nextProps)
        return true
    }
    componentWillUpdate (nextProps, nextState) {
        console.group('---------组件预更新---------')
        console.log('state:', this.state)
        console.log('props:', this.props)
        console.log('nextState:', nextState)
        console.log('nextProps:', nextProps)
    }
    componentDidUpdate (prevProps, prevState) {
        console.group('---------组件更新完成---------')
        console.log('state:', this.state)
        console.log('props:', this.props)
        console.log('prevState:', prevState)
        console.log('prevProps:', prevProps)
    }
    render () {
        console.group('---------组件渲染---------')
        console.log('state:', this.state)
        console.log('props:', this.props)
        return (
            <div ref='text'>{this.props.text}</div>
		)
	}
}
ReactDOM.render(
    <LifeCycle text='Hello World' />,
    document.getElementById('root')
);
setTimeout(() => {
    ReactDOM.render(
        <LifeCycle text='I have changed' />,
        document.getElementById('root')
    );
}, 1000);
```

以下分别是组件加载和更新的执行结果：

![](https://github.com/Neil-WN/MarkdownPhotos/blob/master/images/react_life_cycle_mount.png?raw=true)

![](https://github.com/Neil-WN/MarkdownPhotos/blob/master/images/react_life_cycle_update.png?raw=true)



## 2. React的新生命周期

React从v16.3开始，对组件的生命周期做了相应的调整，以下将对React新生命周期中的内容进行说明，有兴趣的同学可以参考官方（https://reactjs.org/docs/react-component.html）的具体内容。



### 2.1. Mounting

#### static getDerivedStateFromProps(props, state)

在新版本的组件加载阶段，新增了getDerivedStateFromProps()钩子函数。这是一个静态方法，参数props是新接收的props，state是当前的state值，通常将props和state进行比较，判断是否需要更新，如果需要则返回一个对象来更新state，否则返回null。

getDerivedStateFromProps()除了在加载的阶段会被调用，在更新的阶段也会被调用，替代了原有的componentWillReceiveProps()，以下是一个官方的例子：

```javascript
// Before
class ExampleComponent extends React.Component {
  state = {
    isScrollingDown: false,
  };

  componentWillReceiveProps(nextProps) {
    if (this.props.currentRow !== nextProps.currentRow) {
      this.setState({
        isScrollingDown:
          nextProps.currentRow > this.props.currentRow,
      });
    }
  }
}
```



```javascript
// After
class ExampleComponent extends React.Component {
  // Initialize state in constructor,
  // Or with a property initializer.
  state = {
    isScrollingDown: false,
    lastRow: null,
  };

  static getDerivedStateFromProps(props, state) {
    if (props.currentRow !== state.lastRow) {
      return {
        isScrollingDown: props.currentRow > state.lastRow,
        lastRow: props.currentRow,
      };
    }

    // Return null to indicate no change to state.
    return null;
  }
}
```



### 2.2. Updating

#### getSnapshotBeforeUpdate(prevProps, prevState)

getSnapshotBeforeUpdate()会在render()之后，componentDidUpdate()之前调用。该函数接收两个参数，prevProps表示更新前的props，preState表示更新前的state，通常通过比较prevProps和props，判断是否需要返回一个snapshot（快照），如果不需要则返回null，返回的值将会作为componentDidUpdate()的第三个参数使用。因此该函数会在componentDidUpdate()前调用，并需要与componentDidUpdate()配合使用。

以下是一个官方例子：

```javascript
class ScrollingList extends React.Component {
  constructor(props) {
    super(props);
    this.listRef = React.createRef();
  }

  getSnapshotBeforeUpdate(prevProps, prevState) {
    // Are we adding new items to the list?
    // Capture the scroll position so we can adjust scroll later.
    if (prevProps.list.length < this.props.list.length) {
      const list = this.listRef.current;
      return list.scrollHeight - list.scrollTop;
    }
    return null;
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    // If we have a snapshot value, we've just added new items.
    // Adjust scroll so these new items don't push the old ones out of view.
    // (snapshot here is the value returned from getSnapshotBeforeUpdate)
    if (snapshot !== null) {
      const list = this.listRef.current;
      list.scrollTop = list.scrollHeight - snapshot;
    }
  }

  render() {
    return (
      <div ref={this.listRef}>{/* ...contents... */}</div>
    );
  }
}
```



### 2.3. 其他

除了新增了两个钩子函数外，还有如下几个钩子函数发生了名称上的变化：

`componentWillMount() / UNSAFE_componentWillMount()`

`componentWillReceiveProps() / UNSAFE_componentWillReceiveProps()`

`componentWillUpdate() / UNSAFE_componentWillUpdate()`

为了向下兼容，以上的方法会在v16中同时存在，而在v17中将会删除原有的componentWillMount()、componentWillReceiveProps()、componentWillUpdate() 这三个函数，保留相对应的UNSAFE_XX函数。

最后，贴上完整的生命周期图：

![](https://github.com/Neil-WN/MarkdownPhotos/blob/master/images/react_new_life_cycle.jpg?raw=true)

## 3. 总结

组件的生命周期是React的一个重要概念，只有掌握它才能更好的了解React组件的执行，避免一些不必要的错误。React v16为了未来的异步渲染，提供了新的钩子函数，虽然React为了向下兼容保留了旧的函数，但推荐使用新的生命周期来替换。



## 参考

- React.component(https://reactjs.org/docs/react-component.html)
- 「1分钟」React新的生命周期？了解一下(https://zhuanlan.zhihu.com/p/35347230)
- 重新认识 React 生命周期(https://blog.hhking.cn/2018/09/18/react-lifecycle-change/)