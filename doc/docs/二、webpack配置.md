
Webpack 配置
---
项目背景：原项目是典型的MVC结构，前端页面的开发采用的是jQuery + ejs，通过NodeJS进行了路由跳转以及服务端渲染。

个人对jQuery用起来不是特别熟练，维护起来分外痛苦。其次，产品经理设计产品原型采用的是Ant Design，使用jQuery很难做到与产品设计的统一。

由于原项目已经很庞大，对项目代码进行重构不太现实，故只能针对后续需求的迭代开发采用React。针对这种多页面的情况，需配置Webpack，Webpack的配置过程如下所述。

## 1. 输入配置

有别于传统的SPA应用，本项目的路由分发在Node端进行，并采用了ejs模板引擎进行渲染。在这样的场景下采用react框架进行后续需求迭代开发，目前想到的方法是针对每个页面产生一个单独输出，并将该输出文件通过script标签置于文件底部，如下所示。

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

需求可以简单概括为：每个页面都有一个对应的JS脚本和CSS文件，为了防止页面重名，需将JS脚本和CSS文件输出到不同目录下。webpack此时就需配置成多入口多输出的形式。

**entry的配置**

```JSON
{
  "entry": {
    "robot/fileA/fileNameA/nameA": "./src/work/robot/fileA/fileNameA/nameA.js",
    "robot/fileB/fileNameB/nameB": "./src/work/robot/fileB/fileNameB/nameB.js",
  }
}
```

将entry的key由原来的"nameA"修改成"robot/fileA/fileNameA/nameA"，就可将生成的JS文件、CSS输出到robot/fileA/fileNameA目录下。

## 2. 输出配置

```JS
output: {
  path: path.join(__dirname, '../public/dist/work'),
  filename: "[name].js",
  chunkFilename: "[name].[chunkhash:8].js"            //不宜设置成[name].[hash:8].js
}
```

由于不同的页面对应着不同的JS脚本，只能通过手动注入，无法通过插件自动注入，所以filename采用[name].js，而不采用[name].[hash:8].js，这样就不会因为每次文件变动而需手动重新再次注入的问题存在。

## 3. Resolve

Resolve 配置 Webpack 如何寻找模块所对应的文件。 

```JS
resolve: {
    alias: {
      Component: path.resolve(__dirname, '../src/component/'),
      Util: path.resolve(__dirname, '../src/util/'),
      Redux: path.resolve(__dirname, '../src/redux'),
      Api: path.resolve(__dirname, '../src/api')
    },
    extensions: ['.js', '.jsx', '.json'],
  },
```

#### 3.1 alias

通过别名来把原导入路径映射成一个新的导入路径。当通过import componentA from 'Components/componentA'时，会被等价替换成import componentA from '../src/component'

#### 3.2 extensions

在导入语句没带文件后缀时，Webpack 会自动带上后缀后去尝试访问文件是否存在。

**附：path.resolve方法用于将相对路径转为绝对路径。**

```Shell
path.resolve('foo/bar', '/tmp/file/', '..', 'a/../subfile')
# 上面代码的实例，执行效果类似下面的命令。

$ cd foo/bar
$ cd /tmp/file/
$ cd ..
$ cd a/../subfile
$ pwd
```
### 4 (待补充)

### 5. 插件splitChunks

webpack 4 移除了 CommonsChunkPlugin, 取而代之的是两个新的配置项 optimization.splitChunks 和 optimization.runtimeChunk。

```JS
optimization: {
  splitChunks: {
    chunks: 'all',
    name: 'common',
  }
}
```

### 4 webpack 4


#### 3.2.2 css-module问题

```JS
import React from 'react'
import style from './index.less'

class App extends React.Component {
    render() {
        return <div className={style.main}>My Text</div>
    }
}
```

其less文件如下：

```CSS
.main {
    color: black;
}
```

开发中发现设置的样式没有任何效果，并且console.log(style.main)的结果是undefined。

这是因为上述的用法是css-module。

待深究什么是css-module(貌似CSS-loader要设置options: { modules: true })。


或采用如下方式。

```JS
import React from 'react'
import './index.less'

class App extends React.Component {
  render() {
    return <div className="main">My Text</div>
  }
}
```

### 3.4 webpack配置

```JS
const path = require('path')
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
const autoprefixer = require('autoprefixer')
const rucksack = require('rucksack-css')
const webpack = require('webpack')
const entryFile = require('./entry.json')

const postcssOptions = {
  sourceMap: true,
  plugins: [
    rucksack(),
    autoprefixer({
      browsers: ['last 2 versions', 'Firefox ESR', '> 1%', 'ie >= 8', 'iOS >= 8', 'Android >= 4'],
    }),
  ],
}

module.exports = {
  mode: 'production',
  entry: entryFile.entry,
  output: {
    path: path.join(__dirname, '../public/dist/work'),
    filename: "[name].js",         // filename对应于entry里面生成出来的文件名

    // chunkname可以理解为未被列在entry中，却又需要被打包出来的文件命名配置。什么场景需要呢？在按需加载(异步)模块的时候，这样的文件是没有被列在entry中的，如使用CommonJS的方式异步加载模块：
    // require.ensure(["modules/tips.jsx"], function(require) {
    //   var a = require("modules/tips.jsx");
    //   ...
    // }, 'tips');
    // 异步加载的模块是要以文件形式加载哦，所以这时生成的文件名是以chunkname配置的，生成出的文件名就是tips.min.js。
    //（require.ensure() API的第三个参数是给这个模块命名，否则chunkFilename: "[name].min.js" 中的[name]是一个自动分配的、可读性很差的id。
    chunkFilename: "[name].js",
  },
  resolve: {
    modules: ['node_modules', path.join(__dirname, './node_modules')],
    extensions: ['.web.tsx', '.web.ts', '.web.jsx', '.web.js', '.ts', '.tsx', '.js', '.jsx', '.json'],
  },
  module: {
    noParse: /node_modules\/(moment\.js)/, //不解析
    rules: [
      {
        test: /\.css$/,
        use: [
          { loader: MiniCssExtractPlugin.loader },
          {
            loader: 'css-loader',
            options: {
              minimize: true, //压缩
            },
          },
          {
            loader: 'postcss-loader',
            options: postcssOptions
          },
        ],
      },
      {
        test: /\.less/,
        use: [
          // { loader: 'style-loader' },
          { loader: MiniCssExtractPlugin.loader },
          {
            loader: 'css-loader',
          },
          {
            loader: 'postcss-loader',
            options: postcssOptions
          },
          { loader: 'less-loader' }
        ]
      },
      /**处理图片**/
      {
        test: /\.(png|svg|jpg|gif)$/,
        use: [
          'file-loader'
        ]
      },
      /**加载字体**/
      {
        test: /\.(woff|woff2|eot|ttf|otf)$/,
        use: [
          'file-loader'
        ]
      },
      {
        test: /\.jsx?$/,
        exclude: /node_modules/,
        loader: 'babel-loader',
        query: {
          cacheDirectory: true,
          presets: [
            'babel-preset-es2015-ie',
            'babel-preset-react',
            'babel-preset-stage-0',
          ].map(require.resolve),
          plugins: [
            'babel-plugin-add-module-exports',
            'babel-plugin-transform-decorators-legacy'
          ].map(require.resolve)
        }
      }
    ]
  },
  stats: {
    children: false,
    warningsFilter: "mini-css-extract-plugin"     //权宜之计，该插件存在Bug，待该插件版本升级
  },
  optimization: {
    splitChunks: {
      chunks: 'all',
      name: 'common',
    }
  },

  plugins: [
    new MiniCssExtractPlugin({
      filename: "[name].css"
    })
  ]
}
```