
# atool-build使用小结

### 2.1 webpack.config.js

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

### 2.2 package.json

atool-build 要求在 package.json 文件里面增加 entry 字段。

区别于传统的SPA应用，公司项目的路由跳转是在Node端实现的。要在原有项目上进行兼容，并使用React进行后续需求开发，目前想到的办法是针对每个页面都设置entry，然后生成不同的bundle文件。最后每个页面通过script标签把自己的bundle文件引进来。

此时有个问题：如何针对不同目录下的入口js，编译生成的bundle文件也存放在不同目录下。**解决：**

```JSON
{
  "entry": {
    "robot/fileA/fileA": "./src/work/robot/fileA/index.js",
    "robot/fileB/fileB": "./src/work/robot/fileB/index.js"
  }
}
```

## 四、atool-build弃坑缘由

### 错误："Cannot read property 'call' of undefined"

项目背景：entry---存在多个entry文件，output---每个entry对应一个output，并且需提取公共文件。

在开发过程中，会运行脚本atool-build -w，用来监测文件变化并进行自动编译。但在实践中，每当文件改动时，脚本自动编译后就会报如下错误：

> Uncaught TypeError: Cannot read property 'call' of undefined。

而直接运行脚本atool-build进行编译，则不会报上述错误。也就是说上述错误只出现在watch模式下。

经排查，这个问题可能与atool-build使用了CommonsChunkPlugin插件有关，GitHub相关issue见[链接](https://github.com/webpack/webpack/issues/959)。

问题起因：

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

如上所述，需进行插件的配置，而atool-build很难进行插件的私有配置，故只能舍弃atool-build。