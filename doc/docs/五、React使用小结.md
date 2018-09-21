
React使用小结
---

### 1. ref属性注意点

**报错：**

![](https://raw.githubusercontent.com/Bian2017/jQueryWithReact/master/doc/img/QQ20180822-161454.png)

**原代码：**

```JS
handleScroll = () => {
  const scrollTop = this.refs.messageList.scrollTop

  if (0 === scrollTop) {
    props.getNextWxRecords()
  }
}

<div className="scroll-wrapper">
  <div ref="messageList" className="box_bd"   onScroll={() => this.handleScroll()}>
    xxxx
  </div>
</div>
```

**错误原因：**

【摘自[链接](https://react.docschina.org/docs/refs-and-the-dom.html)】如果你之前使用过 React ，你可能了解过之前的API中的 string 类型的 ref 属性，比如 “textInput” ，你可以通过 this.refs.textInput 访问DOM节点。我们不建议使用它，因为 String 类型的 refs 存在问题。它已过时并可能会在未来的版本被移除。如果你目前还在使用 this.refs.textInput 这种方式访问 refs ，我们建议用回调函数的方式代替。

注：Refs 提供了一种访问在 render 方法中创建的 DOM 节点或 React 元素的方式，但不要过度使用Refs，

[参考链接](https://reactjs.org/warnings/refs-must-have-owner.html)

**代码修改：**

```JS
handleScroll = () => {
  const scrollTop = this.messageList.scrollTop

  if (0 === scrollTop) {
    props.getNextWxRecords()
  }
}

<div className="scroll-wrapper">
  <div ref={inst => this.messageList = inst} className="box_bd"   onScroll={() => this.handleScroll()}>
    xxxx
  </div>
</div>
```

### 2. 尽量不要在render函数中定义无状态组件

**出错现象：**

在render函数中定义了一个无状态组件，该无状态组件有个Input输入，每次输入一个字符Input就立马失去焦点。

```JS
class ComponentA extends React.Component() {
  render() {
    const { arrayData } = this.props
    const FormItem =  () => arrayData.map(d => <Input key={d.id} value={d.value} />)
    
    return <div>
      <FormItem />
      xxx
      </div>
  }
}
```

**错误原因：**

由于Input标签的值是存储在Redux中的，所以每当Input值发生变化，就会引起props发生变化，从而引起重新渲染。而无状态组件定义在render中，就会重新生成一个新的无状态组件，由于是新的组件，所以就会失去焦点。

**代码修改：**

```JS
const FormItem =  () => arrayData.map(d => <Input key={d.id} value={d.value} />)

class ComponentA extends React.Component() {
  render() {
    const {arrayData} = this.props
    
    return <div>
      <FormItem />
      xxx
      </div>
  }
}
```

### 1.2 

报错如下：“Can't call setState (or forceUpdate) on an unmounted component. This is a no-op, but it indicates a memory leak in your application.”。

起因：当载入一个新页面时，页面的组件会挂载并卸载好几次。若在componentDidMount生命周期发起服务器请求，可能会存在服务器响应到来前就被卸载。此时若在收到服务器响应的时候设置setState就会报如上错误。

[参考链接](https://www.youtube.com/watch?v=8BNdxFzMeVg)



### 4.4 语法错误

报错如下："Uncaught TypeError: Cannot convert undefined or null to object"。

![](https://raw.githubusercontent.com/Bian2017/jQueryWithReact/master/doc/img/QQ20180815-184910.png)

**起因：**

```JS
function func(params) {
  this.setState({...params})
}

func()              
func({offset:1, limit:10})
```

分析：func()中params值为undefined。对undefined进行扩展，就会如上错误。

**修改：**

```JS
function func({offset=1, limit=10}) {
  this.setState({offset, limit})
}

func()              
func({offset:1, limit:10})
```