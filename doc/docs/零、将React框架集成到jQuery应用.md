React是一个JavaScript Web UI框架，允许您为Web应用程序构建基于组件的用户界面控件。使用框架（例如React）的一个好处是，您创建的每个组件都保持其自己的状态，与其他组件无关。它还使得开发单独的UI部件变得容易和干净，与Web应用程序的其余部分分开，并且可以在应用程序的其他区域甚至在其他Web应用程序中重复使用。

由于许多现有的Web应用程序以前是使用JQuery开发的，因此可能需要构建一个可以与JQuery Web应用程序集成和通信的新React组件或Web应用程序。例如，您可能希望从可能在页面上呈现的基于JQuery的Web应用程序中读取信息。然后，您希望在React组件中显示该数据。为此，您的React组件需要某种方式从JQuery应用程序请求数据，然后将其存储在其状态中。

同样，也许您有一个现有的JQuery应用程序，并且您希望您的React组件将数据发送到JQuery应用程序以在Web页面上呈现。您需要一个类似的集成点才能与JQuery应用程序通信，告诉它更新页面上的数据区域。

幸运的是，从React到JQuery以及从JQuery到React的沟通实际上非常简单易行。我们将看看3个模型，包括直接引用JQuery对象，使用中间管理器，最后使用简单的发布者/订阅者（即pubsub）模型。

在本文中，我们将介绍将React组件与现有JQuery Web应用程序集成的步骤。我们将介绍如何在JQuery和React之间来回通信，允许两个部分相应地更新其状态并在网页上呈现数据。我们还将演示如何访问JQuery $（this）上下文，以便更新部分UI并直接从React组件中读取JQuery控件中的数据。

## 从JQuery到React

React组件与JQuery通信存在两个基本方法。首先，我们假设有如下简单的HTML文档。

```HTML
<div class='container'>
  <h1>JQuery to React using $(this)</h1>
  <div id='box' style='border: 1px solid #333; background-color: red; width: 100px; height: 100px;'></div>
  <br />
  <div id='root' class='mt-3'></div>
</div>
```

如上，JQuery应用程序包含标题h1和彩色方块div，同时还包含了一个div作为React组件在其中呈现的基础。让我们看看允许React与外部应用程序的UI元素交互的几种方法。

我们将实现3种不同的方法来与React中的JQuery进行交互。将讨论以下方法：

+ 从React中引用JQuery上下文；
+ 使用传递给React的中间帮助器类。
+ 使用传递给React的发布者/订阅者模型。
我们将从最直接的交互开始，即将JQuery对象简单地传递给React组件。

直接从React引用JQuery上下文
第一种方法涉及在最初创建React组件时，在构造函数中传递React组件JQuery上下文$（this）的副本。通过这种方式，React可以直接操作现有的Web应用程序UI元素。

您可以在下图中看到此设计。

![]()

如图所示，JQuery App最初使用对React.createElement的调用创建React组件，并将JQuery应用程序的上下文context传递给组件构造函数。然后，组件可以在state中存储引用（通过props参数获得），并使用它来更新网页上的关键元素。这就实现了组件可在其自己组件区域之外更改网页元素。

```JS
ReactDOM.render(React.createElement(MyComponent, { context: $('body') }), document.getElementById('root'));
});
```

上面代码示例显示了最初如何创建组件。组件的构造函数传递给JQuery主体对象的引用，该主体对象可以在组件构造函数的props中访问。

### React组件

代码如下所示，React组件存储JQuery上下文，并直接操作UI元素。注意构造函数如何为props创建一个参数，它将包含我们在创建时传入的JQuery上下文。上下文存储在组件的状态中。

```JS
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    
    this.state = {
      context: props.context
    };
    this.onClick = this.onClick.bind(this);
  }
  
  onClick() {
    // Get reference to a JQuery object in parent app.
    var box = this.state.context.find('#box');
    
    // Change the color of the box.
    box.css('background-color', (box.css('background-color') === 'rgb(255, 0, 0)' ? 'green' : 'red'));
  }
  render() {
    return (
      <div className='alert alert-success' role='alert'>
        <h3>Hello, from React!</h3>
        <button type='button' className='btn btn-default' onClick={ this.onClick }>Click Me</button>
      </div>
    );
  }
}
```

上面定义的组件呈现简单的消息和按钮。单击该按钮时，组件将访问JQuery上下文以更改其正常范围之外的UI元素上的背景颜色。该应用程序的屏幕截图如下所示。



## 二、Using an Intermediate Javascript Class

该方法创建一个Javascript对象，作为现有JQuery应用程序与React组件之间通信的媒介，并将对象引用传递给React组件。React组件可以通过封装的对象请求数据，或者使用新数据更新页面部分区域。

下方流程图展示了React组件如何通过封装对象与现有JQuery应用程序进行通信。

![]()

如图所示，JQuery应用程序和React组件分别控制自己的网页元素。React组件接收封装Javascript对象的副本，以便它可以从父JQuery应用程序接收通知，以及请求更新主网页（在其自己的元素部分范围之外）。

这种方法的一个主要好处是React组件对页面上UI元素的责任与JQuery应用程序对其自己的UI元素的责任之间的关注点分离。该组件可以维护自己独立的网页部分，同时允许JQuery继续拥有其余部分。通过封装对象发出请求，组件可以请求对其控件之外的显示进行更新，JQuery应用程序也会相应地做出响应。

下面显示了实例化React组件并将其传递给中间类的示例。

```JS
ReactDOM.render(React.createElement(MyComponent, { context: UIManager }), document.getElementById('root'));
```

注意，在上面的代码中，我们现在正在传递对JavaScript类的引用，而不是像第一个示例中那样传递JQuery上下文。该类对象将包含在React组件本身范围之外操作各种UI元素和状态的方法。

UIManager类可以定义如下。

```JS
var UIManager = {
  getColor: function(parent, callback) {
    callback($('#box').css('background-color'), parent);
  },
  setColor: function(name) {
    $('#box').css('background-color', name);
  }
};
```

请注意，上面的类只包含两种处理页面中正方形颜色的方法。我们可以检索方块的当前颜色值或更改其颜色。

**React组件**

与第一个示例类似，我们可以将外部应用程序的上下文存储在组件的状态state中。但是，我们现在存储对中间UIManager类的引用，而不是上下文是对JQuery本身的引用。

代码如下所示。


```JS
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    
    this.state = {
      context: props.context
    };
    this.onClick = this.onClick.bind(this);
  }
  
  onClick() {
    this.state.context.getColor(this, function(color, that) {
      that.state.context.setColor(color === 'rgb(255, 0, 0)' ? 'green' : 'red');
    });
  }
  render() {
    return (
      <div className='alert alert-success' role='alert'>
        <h3>Hello, from React!</h3>
        <button type='button' className='btn btn-default' onClick={ this.onClick }>Click Me</button>
      </div>
    );
  }
}
```

请注意，在上面的代码中，我们将UIManager引用（称为props.context）存储在React组件的状态中。当用户单击组件中的按钮时，我们通过上下文调用UIManager来检索框的当前颜色。然后我们再次调用UIManager来改变颜色。


## 二、外部直接调用React组件方法

除了上面用于React和JQuery或外部应用程序之间通信的模型之外，您还可以通过直接调用React组件本身内的方法，以相反的方式从jQuery到React进行通信。

当React组件被rendered时，它返回React组件的实例。通过该实例您可以直接调用React组件中的方法。通过这种方式，您可以通过调用React类上的各种方法来控制外部（例如，来自JQuery）的React组件，这些方法可以更改React应用程序的状态，UI和其他行为。


### 2.1 外部调用React组件方法

看一个简单示例：外部（JQuery）应用程序如何调用React组件方法。

```HTML
<button id='btnShow' type="button" class="btn btn-primary">Show</button>
<button id='btnHide' type="button" class="btn btn-primary">Hide</button>
<div id="root" class='mt-5'></div>
<div id="output"></div>
```

在上面的HTML中，我们有两个用于显示和隐藏的按钮。我们还有一个root div，它是渲染React组件的地方。最后，我们有一个output div，用于展示来自React组件的文本。

#### a) Rendering the React Component

首先在指定的div中渲染React组件，代码如下所示：

```JS
var myComponent = ReactDOM.render(React.createElement(MyComponent), document.getElementById('root'))
```

注意：在调用render命令之后，我们将React组件实例的副本存储在名为myComponent的变量中，**这是调用React组件的内部方法的关键**。

现在添加一些jQuery代码来处理show和hide两个按钮的Click事件。当按钮的Click事件触发时，我们将调用React组件中的show和hide方法。

```JS
$(function() {
  $('#btnShow').click(function() {
    $('#output').text('');
    
    myComponent.show('Hello World!', function(text) {
      $('#output').text(text);
    });
  });
  
  $('#btnHide').click(function() {
    myComponent.hide();
  });
});
```

上面的jQuery代码为show和hide两个按钮添加了事件回调。单击每个按钮时，我们调用React组件的内部方法并在 output div中显示结果。

**React组件**

与上面的其他示例类似，我们的React组件有show和hide的两个公共方法，代码如下所示。

```JS
class MyComponent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      text: '',
      isVisible: false
    };
    
    this.onSpeak = this.onSpeak.bind(this);
  };

  show(text, callback) {
    this.setState({ isVisible: true, text: text, callback: callback });
  }

  hide() {
    this.setState({ isVisible: false });
  }
  
  onSpeak() {
      this.state.callback('Hello world! This is the MyComponent control.');
  }
  
  render() {
    return (
      <div className={ 'my-component ' + (this.state.isVisible ? '' : 'hidden-xs-up') } >
        <div className="card">
          <div className="card-header">
            My Component
          </div>
          <div className="card-block">
            <h4 className="card-title"></h4>
            <p className="card-text">
              { this.state.text }
            </p>
            <a href="#" className="btn btn-primary" onClick={ this.onSpeak }>Speak</a>
          </div>
        </div>
      </div>
    );
  }
};
```

上面的React组件不仅定义了show和hide方法，并呈现了自己的UI。 UI包含一个带标题的简单框，一行文本和一个用于更改文本的按钮。当React组件显示时，用户可以单击“Speak”按钮在React组件的UI中显示一行文本。

当调用show方法时，我们存储回调，客户端（JQuery）代码可以使用它来处理输出的结果文本。在我们的例子中，当用户单击React组件中的“Speak”按钮时，我们的React组件会调用回调并向其传递一些文本。我们在自己的输出div中渲染该文本，该输出div位于React组件之外。通过这种方式，我们可以通过直接调用React组件类从React回传到JQuery，同时仍允许React组件相应地处理自己的事件和UI。


## 四、如何在Web页面中包含React

之前我们讨论了如何创建可以与外部JQuery Web应用通信的React组件，让我们回顾一下在Web页面中实际托管React组件的几种不同方法。

React要求您包含用于解释组件的脚本标记。您可能还需要包含脚本标记，以便将较新的JavaScript语句转换为标准的浏览器兼容格式。

要查看该页面，您需要运行本地Web服务器（例如http-server）。这将允许React正确呈现，而不是简单地在浏览器中将本地index.html文件作为file：// url打开。

### 4.1 在单页面中加载React

Web页面中加载React组件最简单的一个方法就是通过script标签包含React库和编写的React组件。由于React组件使用了JSX语法，因此还需要引用一个库来解释它们，称为“babel”。使用JSX创建的任何React脚本都需在script标签中指定type为“text/babel”。这告诉浏览器允许babel使用AJAX XHR请求加载脚本，而不是浏览器中的标准加载脚本。以这种方式，脚本可以由babel预处理并随后在浏览器中正确呈现。

在单页面中实现加载React，需在页面中包含以下script标签：

**index.html**

```HTML
<script src="https://cdnjs.cloudflare.com/ajax/libs/react/15.5.4/react.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/react/15.5.4/react-dom.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/6.24.0/babel.min.js"></script>
<script src="MyComponent.js" type="text/babel"></script>
```

最终的脚本标记“MyComponent.js”指的是您创建的React组件。请注意定义为text / babel的脚本标记的类型，表示该脚本将由babel使用AJAX请求加载和预处理。

**index.js**

除了配置index.html页面之外，您还需在页面上呈现您的React组件，可以通过如下代码加载和显示您的组件。

```JS
$(function() {
  setTimeout(function() {
    ReactDOM.render(React.createElement(MyComponent, { context: LogManager }), document.getElementById('root'));
  }, 0);
});
```

在上述代码中，我们使用setTimeout来实现页面完全呈现后再展示React组件。如果您的Web页面也使用JQuery，可以将代码包含在JQuery document-ready function中。

## 4.2 托管模块中的React
在网页中包含React的更方便的方法是将React组件打包到可分发的npm模块中。

您可以按照以下步骤使用帮助工具执行此操作。



上面的命令将设置一个React模板项目，您可以从中创建组件。组件准备就绪后，可以使用命令npm run build打包它们。

注意，Windows用户需要编辑文件package.json来更新“脚本”部分，如下所示：


构建之后，您应该有一个dist文件夹，其中包含组件的javascript文件，包括缩小版本。

## 在页面中包含Minified React Component
您可以通过在网页中包含以下脚本标记来引用React组件的缩小版本。

请注意，就像在独立示例中一样，我们已经包含了React库的脚本标记引用。但是，这次没有必要包括babel，因为分发过程为我们处理了这个问题。您可以直接引用组件的javascript文件，并保留text / babel type属性。

index.js
渲染React组件的代码与我们的其他示例相同，如下所示。

## 使用Require Statements托管React
在网页中托管React组件的另一种方法是使用由生成器工具（位于example / dist / bundle.js中）生成的bundle.js文件。在运行npm start之后，最初运行您的React应用程序，将在项目的目录中创建一个示例文件夹，其中包含src和dist文件夹。您可以在dist文件夹中引用文件bundle.js，以将您的组件和React引用包含在单个文件中。使用此方法，无需在页面中引用React脚本标记。此外，您可以选择性地要求渲染特定组件。

的index.html
您的网页只需要引用common.js和bundle.js，如下所示。

index.js
渲染组件的脚本页面需要从bundle.js中获取所需的组件，如下所示。


结论
我们已经讨论了三种不同的方法，用于在React组件中与现有的JQuery Web应用程序进行交互。通过直接引用JQuery上下文，中间UI管理器类和发布者/订阅者模型等方法，我们可以提供一系列设计方法，允许React组件与其内部组件范围之外的UI元素进行交互。 。

当然，有许多不同的方法可以实现类似的设计。您甚至可以直接从React组件中引用JQuery并随意选择元素。但是，除非将状态值存储在HTML元素的数据属性中，否则这将不允许您访问外部应用程序本身的上下文/范围。否则，通过将React组件传递给应用程序上下文的实例，您可以允许将React组件集成到现有的JQuery应用程序中，而对应用程序的其余部分几乎没有中断。
