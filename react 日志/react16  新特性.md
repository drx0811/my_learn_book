# react新特性

## 懒加载lazy和Suspense

> lazy首字母是小写，Suspense的首字母是大写；lazy是一个方法里面传一个函数，函数的返回值是 `import` ('要引入文件的路径，可以是绝对路径也可以是相对路径') Suspense 有一个callback 属性，里面可以是一个node节点，当加载中的时候显示的fallback，成功之后就是Susepense里面的节点了；

### 问题总结

- 该方法的的fallback方法状态的更改值是在 代码加载完之后改变，至于代码内部的异步执行并不关心；
- 这种加载与执行之间的状态改变可能导致loading不一致；

### import()方法的原理

> import（） 方法的原理是通过动态创建script标签，onload结束之后再除去script标签

## Portals特性

> Portals就像一个逃生舱，把我们所有的节点从root根节点的里面跳出来；我们可以在创建modal框的时候用到，在之前创建的时候要用un_sdadsfasdfasd的方法，这种方法是不稳定的，
>
> - 该方法是定义在ReactDOm上面的方法；
> - 该方法传递两个参数，一个是内容children，一个是要挂载的节点 modal-root

  ```javascript
 ReactDOM.createPortal(
   this.props.children,
   this.el
 )
  ```

### 问题总结

- 在使用的时候我们要在**componentDidMount**中把创建的元素`appendChild`进去，在**componentWillUnmount**中我们还要去`removeChild`

## context

> Context 之前就存在的不过不推荐使用，我们一直使用的props，但是我们再父子孙等多层级组件的传递时候写的太麻烦，而现在使用context可以很容易的解决这个问题

  ```javascript
const RiderContext = React.createContext(1) // 这里为默认值
function Page(props) {
  const riderId = props.riderId
  return (
    <RiderContext.Provider value={riderId}>
      <RiderDetail />
    <RiderContext.Provider/>
  )
}

function RiderDetail() {
  return <RiderLevel />
}

class RiderLevel extends React.Component {
  static contextType = RiderContext
  render() {
    return(
      <RiderContext.Provider>
       {
         (content)=>{
            return(
              <div>{content}<div/>
            )
          }
       }
      <RiderContext.Provider/>    
    )
  }
}
  ```

### 问题总结

- 我们的context不会再props中传递，也就是打印this.props是得不到context的
- 我们必须是用***provider***和***consumer***两个方法进行包裹



## 自定义属性

> - 自定义属性通过在dom中以 **data-** 开头，在使用的时候通过  **e.currentTarget** 得到该属性；
>
> - 自定义属性必须是字符创，也就是如果用对象的话 ，我们必须通过 JSON.Stringfy  和JSON.parse进行转换

### 问题总结

1. 其实该方法比较鸡肋；

## 新增生命周期

### getDrivedStateFromProps

> 1. getDerivedStateFromProps 是一个静态方法，里面没有this，该方法会在每次render之前执行；
>
> 2. 废弃的componentWillReceiveProps，之前在此生命周期函数中调用的含有this的逻辑，我们要在 componentDidUpdate中去调用；

#### 老的写法：

```javascript
class ExampleComponent extends React.Component {
  state = {
    externalData: null,
  };
  componentDidMount() {
    // this.props.id === undefined
    this._loadData(this.props.id);
  }
  componentWillReceiveProps(nextProps) {
      //当某个props中的值发生变化时（此处是id属性）
      if (nextProps.id !== this.props.id) {
        //初始化state中的externalData值为null
        this.setState({externalData: null});
        //基于新的id属性，异步获取数据，并在完成时设置externalData的值
        this._loadData(nextProps.id);
    }
  }
  render() {
    if (this.state.externalData === null) {
      // Render loading state ...
    } else {
      // Render real UI ...
    }
  }
  _loadData(id) {
    if (!id) {return;}
    loadData(id).then(
      externalData => {
        this.setState({externalData});
      }
    );
  }
}
```

#### 新的写法

  ```javascript
class ExampleComponent extends React.Component {
  state = {
    externalData: null,
    prevId: this.props.id
  };
	static getDerivedStateFromProps(nextProps, prevState) {
    if (nextProps.id !== prevState.prevId) {
      return {
        externalData: null,
        prevId: nextProps.id,
      };
    }
    return null;
  }
  componentDidUpdate(prevProps, prevState) {
    if (this.state.externalData === null) {
      this._loadData(this.props.id);
    }
  }
  componentDidMount() {
    this._loadData(this.props.id);
  }
  render() {
    if (this.state.externalData === null) {
      // Render loading state ...
    } else {
      // Render real UI ...
    }
  }
  _loadData(id) {
    if (!id) {return;}
    loadData(id).then(
      externalData => {
        this.setState({externalData});
      }
    );
  }
}
  ```

```

```

### getSnapshortBeforeUpdate

> 该方法是在更新阶段调用，在该方法中返回一个内容可以在componentDidUpdate的第三个参数去接收

```javascript
export default  class App extends React.Component{
  constructor(props) {
    super(props);
    this.state = {
      messages: [],//用于保存子div
    }
  }
   handleMessage () {//用于增加msg
    this.setState( pre => ({
      messages: [`msg: ${ pre.messages.length }`, ...pre.messages],
    }))
  }
  componentDidMount () {
    for (let i = 0; i < 20; i++) this.handleMessage();//初始化20条
​    this.timeID = window.setInterval( () => {//设置定时器
​      if (this.state.messages.length > 200 ) {//大于200条，终止
​        window.clearInterval(this.timeID);
​        return ;
​      } else {
​        this.handleMessage();
​      }
​    }, 1000)
  }
  componentWillUnmount () {//清除定时器
​    window.clearInterval(this.timeID);
  }
  getSnapshotBeforeUpdate () {//很关键的，我们获取当前rootNode的scrollHeight，传到componentDidUpdate 的参数perScrollHeight
​    return this.rootNode.scrollHeight;
  }
  componentDidUpdate (perProps, perState, perScrollHeight) {
​    const curScrollTop= this.rootNode.scrollTop;
​    if (curScrollTop < 5) return ;
​    this.rootNode.scrollTop = curScrollTop + (this.rootNode.scrollHeight  - perScrollHeight);
​    //加上增加的div高度，就相当于不动
  }
  render () {

​    return (
​      <div className = 'wrap'  ref = { node => (  this.rootNode = node)} >
​        { this.state.messages.map( msg => (
         <div>{ msg } </div>
​        ))}
      </div>
   );
  }
}
  
  
  
  
```

## React.memo与PureComponent

> - React.memo是一个高阶函数，用来修饰函数组件
>
> - PureComponent 用来创建类组件，通过shouldComponentUpdate进行比较，来确定是否需要进行渲染
> - 两者都是通过浅比较来判断是否重新渲染

### PureComponent

  ```javascript
export default class extends React.Component {
    constructor(props){
        super(props);
        this.state = {
            date : new Date()
        }
    }
    componentDidMount(){
        setInterval(()=>{
            this.setState({
                date:new Date()
            })
        },1000)
    }
    render(){
        return (
            <div>
                <Child seconds={1}/>
                <div>{this.state.date.toString()}</div>
            </div>
        )
    }
}
// 如果不使用PureComponent，那么只要父组件更新，子组件都是重新渲染，性能不好
class Child extends React.PureComponent {
    render(){
        console.log('I am rendering');
        return (
            <div>I am update every {this.props.seconds} seconds</div>
        )
    }
}
  ```

### memo

> `React.memo` 是一个高阶组件, 它使无状态组件拥有有状态组价中的 `shouldComponentUpdate()` 以及 `PureComponent` 的能力。
>
> 可以接受两个参数，第一个是我们的函数组件，第二个是一个函数，用来判断是否需要更新

  ```javascript
import React from "react";
function Child({seconds}){
    console.log('I am rendering');
    return (
        <div>I am update every {seconds} seconds</div>
    )
};

function areEqual(prevProps, nextProps) {
    if(prevProps.seconds===nextProps.seconds){
        return true
    }else {
        return false
    }

}
export default React.memo(Child,areEqual)
  ```