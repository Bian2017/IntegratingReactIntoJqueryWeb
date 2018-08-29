
# webpack 配置

## 一、webpack 4

### 1.1 新特性

+ webpack 4 将移除 CommonsChunkPlugin, 取而代之的是两个新的配置项 optimization.splitChunks 和 optimization.runtimeChunk。



## 三、 场景需求

### 3.1 多个文件输出到多个目录

**原entry配置**

```JSON
{
  "entry": {
    "nameA": "./src/work/robot/fileA/fileNameA/nameA.js",
    "nameB": "./src/work/robot/fileB/fileNameB/nameB.js",
  }
}

修改成如下的entry配置，就能将文件输出到指定目录下。

```JSON
{
  "entry": {
    "robot/fileA/fileNameA/nameA": "./src/work/robot/fileA/fileNameA/nameA.js",
    "robot/fileB/fileNameB/nameB": "./src/work/robot/fileB/fileNameB/nameB.js",
  }
}
```

### 3.2 css-module问题

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

开发中发现设置的样式没有任何效果，而console.log(style.main)的时候，结果是undefined。

究其原因，上述的用法是css-module。待深究什么是css-module(貌似CSS-loader要设置options: { modules: true })。


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



## 四、webpack配置

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