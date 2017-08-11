# vue-pages
## Build Setup
``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```

> A Vue.js project

**基于vue-cli修改配置成一个多页面运用的脚本**

### vue-cli多页面配置

>  对于vue-cli的基础配置可以参考我的这篇blog  [vue-cli的基础运用](https://github.com/huangchucai/My-Note-Blog/issues/7)

阅读本文需要了解的知识点

1. webpack的基本配置 `entry`, ``output`` ,`plugin`
2. ES6 的基本用法
3. 文件的导入和导出

#### 1.目录结构的调整


**源文件下面的src分三个部分**

1. components :  存放单页面组件和整体公用的的组件
2. modules :  存放整体的模块部分
3. pages :  存放页面，每一个页面的内容按照就近原则进行匹配
   - 把原来的单页面的入口文件`main.js`改成`index.js`，并放入到pages中的index文件夹
   - index文件夹中存放的是首页的以前单页面的内容（router，assets, `APP.vue`）

#### 2.修改单页面的webpack的入口文件

**修改之前**

`webpack.base.conf.js`的基本配置

```javascript
// webpack编译的基本配置
module.exports = {
  entry : {
    app: './src/main.js'  //单页面的入口文件
  },
  ...
}
```

*下面用到的glob模块解释*

```javascript
// js/try js文件夹下面的try文件夹里面的a.js b.js c.js 不能含有子目录(try里面的子目录的js读取不了)
var PAGE_PATH = path.resolve(__dirname,'./js');
const entryFiles = glob.sync(PAGE_PATH + '/*/*.js'); //输出的是一个数组

[ 'C:/Users/Z7/Desktop/glob/js/tyr/a.js',
  'C:/Users/Z7/Desktop/glob/js/tyr/b.js',
  'C:/Users/Z7/Desktop/glob/js/tyr/c.js' ]

```

**在utils.js中添加方法，来配置多个入口**

```javascript
// 引入读取文件名的glob模块 (不用安装，已经被隐形安装过)
var glob = require('glob')  // 读取文件名的模块
var PAGE_PATH = path.resolve(__dirname,'../src/pages') //入口文件页面路径

exports.entries = function() {
  var entryFiles = glob.sync(PAGE_PATH + '/*/*.js')  // 读取pages下面的文件夹里面的.js文件
  return entryFiles.reduce((obj,filePath) => {
    var filename = filePath.substring(filePath.lastIndexOf('\/') + 1, filePath.lastIndexOf('.'))
    obj[filename] = filePath
  },{})
}
```

**修改webpack.base.conf.js的入口**,这样就有多个页面的入口了，下面修改多个页面的输出

```javascript
module.exports = {
  entry: utils.entries(),
  .... 
}  
```

#### 3.修改页面的输出配置

**修改之前**，`webpack.dev.conf.js`的配置和`webpack.prod.conf.js`的配置

```javascript
// 页面的输出通过webpack的html-webpack-plugin插件
var HtmlWebpackPlugin = require('html-webpack-plugin')

plugins：[
  ...
  new HtmlWebpackPlugin({
    filename: 'index.html',
    template: 'index.html',
    inject: true
  })
]
```

**我们需要修改它的输出多个页面**。在utils.js中添加方法，来配置多个入口

```javascript
// 多页面的输出配置
exports.htmlPlugin = function() {
  let entryHtml = glob.sync(PAGE_PATH + '/*/*.html');
  let arr = [];
  entryHtml.forEach((filePath) => {
    let filename = filePath.substring(filePath.lastIndexOf('\/')+1, filePath.lastIndexOf('.'))
    var conf = {
      template: filePath,
      filename: filename+ '.html',
      chunks: [filename],
      inject: true
    }
    if(process.env.NODE_ENV === 'production') {
      conf = merge(conf, {
        chunks: ['manifest','vendor',filename],
        minify: {
          removeComments: true,
          collapseWhitespace: true,
          removeAttrubuteQuotes: true
        },
        chunksSortMode: 'dependency'
      })
    }
    arr.push(new HtmlWebpackPlugin(conf))
  })
  return arr;
}
```

**修改build/webpack.dev.conf.js和build/webpack.prod.conf.js的配置**：把原有的页面模板配置注释或删除，并把多页面配置添加到plugins

webpack.dev.conf.js：

```javascript
plugins: [
  ...
  // new HtmlWebpackPlugin({
  //   filename: 'index.html',
  //   template: 'index.html',
  //   inject: true
  // }),
].concat(utils.htmlPlugin())
```

webpack.prod.conf.js

```javascript
plugins: [
  ...
  // new HtmlWebpackPlugin({
  //   filename: config.build.index,
  //   template: 'index.html',
  //   inject: true,
  //   minify: {
  //     removeComments: true,
  //     collapseWhitespace: true,
  //     removeAttributeQuotes: true
  //     // more options:
  //     // https://github.com/kangax/html-minifier#options-quick-reference
  //   },
  //   // necessary to consistently work with multiple chunks via CommonsChunkPlugin
  //   chunksSortMode: 'dependency'
  // }),
].concat(utils.htmlPlugin())
```

#### 4.运用页面

> 1. npm run dev
> 2. npm run build

打包后的文件：




