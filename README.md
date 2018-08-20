# jQueryWithReact

## 项目背景

公司原有项目是典型的MVC结构，页面主要采用jQuery，采用NodeJS做路由跳转。

jQuery项目维护起来比较痛苦，就在原有公司项目上采用React，使用蚂蚁金服的Ant Design做UI库。

## 采用atool-build



四、踩坑指南
---

#### 4.3 Ant Design使用注意点

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

func()              //此处调用函数func，params值为undefined。对undefined进行扩展，就会如上错误。
func({offset:1, limit:10})
```

**修改：**

```JS
function func({offset=1, limit=10}) {
  this.setState({offset, limit})
}

func()              
func({offset:1, limit:10})
```