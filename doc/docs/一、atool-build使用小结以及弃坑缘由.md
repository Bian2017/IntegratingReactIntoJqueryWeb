atool-build使用小结以及弃坑缘由
---

项目背景：原项目是典型的MVC结构，前端页面开发采用的是jQuery + EJS，通过NodeJS进行了路由跳转以及中间层处理。

个人对jQuery用起来不是特别熟练，维护起来比较痛苦。其次，产品经理设计产品原型采用的是蚂蚁金服的Ant Design，使用jQuery + Bootstrap很难做到与产品原型的统一。

由于我个人对React比较熟练，就想在原项目的基础上引入React框架。而原项目比较庞大，短期内对项目代码进行重构不太现实，故只能针对产品后续需求开发采用React。在实践初期，由于工期较紧，为了实现快速开发，便采用了Ant Design提供的[atool-build](https://ant-tool.github.io/index.html)来进行项目的构建与调试。

### 1. 配置输入、输出

有别于传统的SPA应用，本项目的路由分发在Node端进行，并采用了EJS模板引擎进行渲染。要在原有项目上进行兼容，并使用React进行后续需求开发，目前想到的办法是针对每个页面都设置entry，然后生成不同的bundle文件，最后每个页面通过script标签把自己的bundle文件引进来。

```html
<link rel="stylesheet" type="text/css" href="/dist/work/pageA.css">   <!-- 手动注入css -->
<div>
    <ol class="breadcrumb">
        <li>
            <a>
                <%=title%>
            </a>
        </li>
        <li class="pull-right">
            <a class="btn btn-sm btn-default" href="#/wechat">
                <i class="fa fa-chevron-left"></i>
                返回列表
            </a>
        </li>
    </ol>
    <div class="wechat" id="wcManage"></div>
</div>
<script type="text/javascript" src="/dist/work/pageA.js"></script>   <!-- 手动注入react代码 --> 
```

简单概括为：每个页面都有一个对应的React JS脚本和CSS文件，为了防止JS文件重名，需将JS脚本和CSS文件输出到不同目录下，atool-build就需配置成多入口多输出的形式。

#### 1.1 package.json

设置entry字段：entry字段atool-build要求需配置在package.json文件中。

```JS
{
  "entry": {
    "robot/fileA/fileA": "./src/work/robot/fileA/index.js",     // 通过修改key值实现不同文件置于不同目录下 
    "robot/fileB/fileB": "./src/work/robot/fileB/index.js"
  }
}
```

#### 1.2 webpack.config.js

```JS
var path = require('path')

module.exports = function (webpackConfig) {
    webpackConfig.output = {
        path: path.join(__dirname, './public/dist/work'),
        filename: "[name].js",
        chunkFilename: "[name].js"
    }

    return webpackConfig
}
```

### 2. 默认不开启CSS Modules 

atool-build与dva-cli在CSS这块存在一些差异，dva-cli默认开启了css-modules，而atool-build则并没有默认开启这一功能。

```JS
import styles from './myfile.css'

...
<div className={styles.someclass} />    // CSS Modules语法(dva默认开启)
```

此时可以通过如下方式引用CSS：

```JS
import './myfile.css'

...
<div className={someclass} />          // CSS Modules语法，但没有设置modules，所以classNames会被导出到全局范围。
```

注意：[CSS Modlues](http://www.ruanyifeng.com/blog/2016/06/css_modules.html)与[CSS in JS](http://www.ruanyifeng.com/blog/2017/04/css_in_js.html)的区别。

### 3. atool-build弃坑

#### 3.1 watch模式无法解决错误："Cannot read property 'call' of undefined"

在开发过程中，会运行脚本atool-build -w，用来监测文件变化并进行自动编译。但在实践中，每当文件改动时，脚本自动编译后就会报如下错误：

> Uncaught TypeError: Cannot read property 'call' of undefined。

而直接运行脚本atool-build进行编译，则不会报上述错误。也就是说上述错误只出现在watch模式下。

经排查，这个问题与atool-build使用了CommonsChunkPlugin插件(用于提取公共文件)有关，GitHub相关issue见[链接](https://github.com/webpack/webpack/issues/959)。

问题起因：

```
For everyone running into this,
The problem is usually the the order if the files and webpack's module wrap when you use CommonsChunkPlugin or HtmlWebpackPlugin

for HtmlWebpackPlugin you can manually sort your files with chunksSortMode

new HtmlWebpackPlugin({
  // ...
  chunksSortMode: function (a, b) {
    const entryPoints = ["inline","polyfills","sw-register","styles","vendor","main"];
    return entryPoints.indexOf(a.names[0]) - entryPoints.indexOf(b.names[0]);
  },
  // ...
})

for CommonsChunkPlugin the order of the when webpack creates chunks matter since they inject webpackJsonp in the main file. 

```

建议处理方式：

```
So, as a temporary solution before this bug has fixed:
1.If you are using both CommonChunksPlugin and ExtractTextPlugin, make sure the allChunks option of ExtractTextPlugin is set to true;

2.If you are using HtmlWebpackPlugin, set options chunksSortMode to 'dependency'

3.If you did all operations before, trouble is still alive, try solve the dependencies by your hand.(change the order you import the scripts in html)
```

如上所述，需进行插件的配置，而atool-build很难进行插件的私有配置，故只能舍弃atool-build。