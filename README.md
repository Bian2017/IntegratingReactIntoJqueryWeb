# jQueryWithReact

## 项目背景

公司原有项目是典型的MVC结构，页面主要采用jQuery，采用NodeJS做路由跳转。

jQuery项目维护起来比较痛苦，就在原有公司项目上采用React，使用蚂蚁金服的Ant Design做UI库。

## 采用atool-build



四、踩坑指南
---

### 4.1 React踩坑指南
#### 4.1.1 使用ref属性注意点

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

**错误原因：**

【摘自[链接](https://react.docschina.org/docs/refs-and-the-dom.html)】如果你之前使用过 React ，你可能了解过之前的API中的 string 类型的 ref 属性，比如 “textInput” ，你可以通过 this.refs.textInput 访问DOM节点。我们不建议使用它，因为 String 类型的 refs 存在问题。它已过时并可能会在未来的版本被移除。如果你目前还在使用 this.refs.textInput 这种方式访问 refs ，我们建议用回调函数的方式代替。

注：Refs 提供了一种访问在 render 方法中创建的 DOM 节点或 React 元素的方式，但不要过度使用Refs，

[参考链接](https://reactjs.org/warnings/refs-must-have-owner.html)

**修改代码：**

```JS
handleScroll = () => {
  const scrollTop = this.messageList.scrollTop

  if (0 === scrollTop) {
    props.getNextWxRecords()
  }
}

<div className="scroll-wrapper">
  <div ref={inst=>this.messageList = inst} className="box_bd"   onScroll={() => this.handleScroll()}>
    xxxx
  </div>
</div>
```


#### 4.2 

报错如下：“Can't call setState (or forceUpdate) on an unmounted component. This is a no-op, but it indicates a memory leak in your application.”。

起因：当载入一个新页面时，页面的组件会挂载并卸载好几次。若在componentDidMount生命周期发起服务器请求，可能会存在服务器响应到来前就被卸载。此时若在收到服务器响应的时候设置setState就会报如上错误。

[参考链接](https://www.youtube.com/watch?v=8BNdxFzMeVg)


#### 4.3 Ant Design使用注意点---Tabs

报错如下：TypeError: Cannot read property 'ownerDocument' of undefined

![](https://raw.githubusercontent.com/Bian2017/jQueryWithReact/master/doc/img/QQ20180815-174847.png)

**起因：**

```JS
render() {
  const tabTitle = ["未处理", "待学习", "已学习", "忽略"]

  const tabPanes = () => {
    const content = []
    for (let i = 0; i < 4; i++) {
        content.push(
            <TabPane tab={`${tabTitle[i]}`} key={i}>
              <DialogTable
                operateStatus={this.state.key}
                data={this.state.data}
                dataLength={this.state.dataLength}
                current={this.state.current}
                totalPages={this.state.totalPages}
                robot_id={this.state.robot_id}
                setModalStatus={this.setLearnModalStatus}
                setUserSpeech={this.setUserSpeech}
                setRepositoryId={this.setRepositoryId}
                modifyOperateStatus={this.modifyOperateStatus}
                setPagination={this.setPagination}
              />
            </TabPane>
        )
    }
    return content
  }

  return <Tabs onChange={(key) =>this.handleStatus(key)} type="card">
        {tabPanes()}
    </Tabs>
}
```

猜测(待验证)：antd项目使用了开源项目[rc-tabs](https://github.com/react-component/tabs)，而开源项目rc-tabs使用了refs。在如上代码上，tabPanes()可以认为是一个功能组件(functional component)，但是ref是不能应用于[功能组件](https://reactjs.org/docs/refs-and-the-dom.html)上的。故上述情况导致refs被设置为null，所以报如上错误。


主要参考链接：[链接1](https://github.com/reactjs/react-autocomplete/issues/287)、[链接2](https://github.com/facebook/react/issues/9244)。


**修改：**

```JS
render() {
  const tabTitle = ["未处理", "待学习", "已学习", "忽略"]

  return  <Tabs defaultActiveKey="0" onChange={(key) => this.handleStatus(key)} type="card">
    {
      tabTitle.map((v, index) => <TabPane tab={v} key={index}>
          <DialogTable
              operateStatus={this.state.key}
              data={this.state.data}
              dataLength={this.state.dataLength}
              current={this.state.current}
              totalPages={this.state.totalPages}
              robot_id={this.state.robot_id}
              setModalStatus={this.setLearnModalStatus}
              setUserSpeech={this.setUserSpeech}
              setRepositoryId={this.setRepositoryId}
              modifyOperateStatus={this.modifyOperateStatus}
              setPagination={this.setPagination}
          />
        </TabPane>)
    }
    </Tabs>
}

```

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