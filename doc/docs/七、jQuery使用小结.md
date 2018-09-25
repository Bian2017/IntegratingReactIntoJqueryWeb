
jQuery使用小结
---

jQuery()方法的第二个参数详解(https://blog.csdn.net/bobo_93/article/details/53323237)

### 二、jQuery常用API

#### 2.1 data()用法

data()用来将数据绑定在DOM元素上，在实际项目中用来**存储服务器端数据**和传递到JavaScript，可以说是JavaScript使用服务器端数据的一个桥梁。

在JavaScript里，data()修改数据可以这样：data(key, value)和data(obj)。后者等同于data(key1, value1).data(key2, value2)。

如果在HTML里静态绑定了数据，通过data()来获取数据时，key必须全小写，比如绑定data-AGE="31"，只能通过data('age')而不是data('AGE')。另外注意data-last-value="43"，只能通过data('lastValue')或者data('last-value')。

[更多知识](http://www.html-js.com/article/1747)

### 三、jQuery常用插件

#### 3.1 jQuery Validate插件

jQuery Validate 插件为表单提供了强大的验证功能，让客户端表单验证变得更简单，官网[链接](https://jqueryvalidation.org/)。

**踩坑：**

原页面是采用jQuery开发的，并采用Validate插件进行表单的验证。当在这个页面引入Ant Design的Form表单时，每次对Ant Design的Input组件输入数据，就会提示如下错误：

> Uncaught TypeError: Cannot read property 'settings' of undefined.

后经排查，这个错误是由Validate插件引起的，猜测Validate插件可能与Ant Design的Input组件存在某种冲突。后将Input组件使用原生Input标签进行替代，则无上述问题。