# learn-webpack

webpack是一个模块打包工具  
通过学习，最终生成一个基本的webpack配置文档。

## 核心概念

1. 入口(entry)：webpack创建依赖关系时的起点。
1. 输出(output)：打包后的代码存放信息。
1. Loader：把不同类型的文件转换为可以加载的模块。
1. 插件(plugins)：在打包过程中执行操作和自定义功能。

### 入口(entry)

在config对象中以entry属性来定义。

1. 简写语法：`entry: string|Array<string>`  
    entry属性的值可以是一个字符串或者一个数组。
    ```js
    entry: './path/to/my/entry/file.js'
    entry: ['./path/to/my/entry/file1.js','./path/to/my/entry/file2.js']
    ```
1. 对象语法：`entry: {[entryChunkName: string]: string|Array<string>}`  
    对象语法可以给每个条目一个名字；每个条目又可以是一个字符串或者一个数组。
    ```js
    entry: {
        app: './src/app.js',
        vendors: './src/vendors.js'
    }
    ```

### 输出(output)

1. 单个输出文件
    ```js
    output: {
        filename: 'bundle.js',
        path: '/home/proj/public/assets'
    }
    ```
1. 多个输出文件  
    使用了[name]变量，其值是entry中定义的名字。
    ```js
    entry: {
        app: './src/app.js',
        search: './src/search.js'
    },
    output: {
        filename: '[name].js',
        path: __dirname + '/dist'
    }
    ```

### Loader

在module属性的rules属性定义什么样的文件需要什么样的loader处理；每个不同的loader都需要单独安装。
    ```js
    module: {
        rules: [
            { test: /\.css$/, use: 'css-loader' },
            { test: /\.ts$/, use: 'ts-loader' }
        ]
    }
    ```
Loader支持链式传递，一个文件可以被多个loader依次处理。

### 插件（Plugins）

向plugins属性传入new实例；插件也支持选项配置。
    ```js
    plugins: [
        new webpack.optimize.UglifyJsPlugin(),
        new HtmlWebpackPlugin({template: './src/index.html'})
    ]
    ```

## 安装

```sh
mkdir learn-webpack         #创建空目录
npm init -y                 #初始化npm项目
npm i webpack --save-dev    #安装webpack依赖
webpack -v                  #确认webpack版本
```

## 起步

### 一个普通项目，没有用到webpack。

```sh
mkdir src                    #创建代码存储源目录
编辑 index.html              #编辑页面内容，内容见下
编辑 src/index.js            #编辑index.js代码，内容见下
用浏览器查看 index.html       #确认成果
```

index.html

```html
<html>
  <head>
    <title>Getting Started</title>
    <script src="https://unpkg.com/lodash@4.16.6"></script>
  </head>
  <body>
    <script src="./src/index.js"></script>
  </body>
</html>
```

src/index.js

```js
function component() {
    let element = document.createElement('div')             //创建div
    element.innerHTML = _.join(['Hello', 'webpack.'], ' ')  //通过lodash库组合字符串后存入div
    return element
}
document.body.appendChild(component())                      //div加入document
```

### 通过模块的方式使用库，引入webpack进行打包

```sh
mkdir dist                          #创建存放发布代码的目录
mv index.html dist                  #移动首页html到最终发布目录
npm i lodash                        #安装lodash库
编辑 src/index.js                    #修改index.js，通过import引入lodash库
编辑 dist/index.html                 #修改index.html，去除对于loash和index.js的直接引用，增加对于打包后代码的引用
webpack src/index.js dist/bundle.js #通过命令行传入参数执行打包操作
用浏览器查看 dist/index.html          #确认成果
```

src/index.js

```js
import _ from 'lodash'  //使用import引入lodash模块，这样webpack才能找到依赖关系。
function component() {
    let element = document.createElement('div')
    element.innerHTML = _.join(['Hello', 'webpack.'], ' ')
    return element
}
document.body.appendChild(component())
```

dist/index.html

```html
<html>
<head>
    <title>Document</title>
</head>
<body>
    <!-- 只需要引入打包后的代码即可了 -->
    <script src="bundle.js"></script>
</body>
</html>
```

### 通过配置文件执行打包

```sh
编辑webpack.config.js #创建配置文件
webpack               #不带参数时会按照默认值寻找配置文件
```

webpack.config.js

```js
const path = require('path');       //node的方式引入模块
module.exports = {                  //导出的对象
  entry: './src/index.js',          //入口点
  output: {                         //输出对象
    filename: 'bundle.js',          //输出的文件名
    path: path.resolve(__dirname, 'dist')   //输出的目录
  }
};
```

## 管理资源

webpack可以通过loader引入任何其他类型的文件。

### 加载CSS

```sh
编辑 src/style.css
编辑 src/index.js 引入css并应用
npm install --save-dev style-loader css-loader      #为能在js中引用css安装loader
编辑 webpack.config.js
重新执行 webpack 进行打包
用浏览器查看 dist/index.html 可以看到字体颜色变为红色的了。
```

src/style.css

```css
.hello {
    color:red
}
```

dist/index.js

```js
import _ from 'lodash'
import './style.css'        //整个引入css文件

function component() {
    let element = document.createElement('div')

    element.innerHTML = _.join(['Hello', 'webpack.'], ' ')
    element.classList.add('hello')      //给元素增加css类

    return element
}

document.body.appendChild(component())
```

webpack.config.js

```js
const path = require('path')

module.exports = {
    entry: './src/index.js',
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'dist')
    },
    module: {   //增加module字段
        rules: [        //增加规则字段
            { test: /\.css$/, use: ['style-loader', 'css-loader'] } //css结尾的文件通过两个loader依次处理
        ]
    }
}
```

如果在浏览器中查看页面源代码会发现没什么变化，  
但是如果用F12调试窗口查看Elements标签页就可以看到如下内容：  
`<head>`标签中增加了`<style>`标签，并且用来显示hello的`<div>`标签增加了`class`属性。

```html
<head>
    <title>Document</title>
    <style type="text/css">
        .hello {
            color: red
        }
    </style>
</head>
<body>
    <script src="bundle.js"></script>
    <div class="hello">Hello webpack.</div>
</body>
</html>
```

### 加载图片

```sh
npm i file-loader --save-dev
编辑webpack.config.js，图片文件由file-loader来处理。
项目中添加一个图片。
编辑src/index.js，引入图片文件，并创建Image元素使用图片。
编辑src/style.css，也引入图片文件。
执行webpack。图片会被复制到dist目录，并且以hash值命名；index.js中的文件路径会更新；css中的路径会被css-loader更新。
```

webpack.config.js

```js
const path = require('path')
module.exports = {
    entry: './src/index.js',
    output: {
        filename: 'bundle.js',
        path: path.resolve(__dirname, 'dist')
    },
    module: {
        rules: [
            { test: /\.css$/, use: ['style-loader', 'css-loader'] },
            { test: /\.(png|svg|jpg|gif)$/, use: ['file-loader'] }   //文案由file-loader来处理
        ]
    }
}
```

src/index.js

```js
import _ from 'lodash'
import './style.css'
import Icon from './icon.png'       //引入图片文件

function component() {
    let element = document.createElement('div')

    element.innerHTML = _.join(['Hello', 'webpack.'], ' ')
    element.classList.add('hello')

    let myIcon = new Image()        //创建图像元素
    myIcon.src = Icon               //图像元素的图片内容
    element.appendChild(myIcon)     //加入Div

    return element
}

document.body.appendChild(component())
```

src/style.css

```css
.hello {
    color:red;
    background: url('./icon.png');  /*引用图片*/
}
```

浏览器中最终加载的index.html

```html
<html>
<head>
    <title>Document</title>
    <style type="text/css">
        .hello {
            color: red;
            background: url(489a185362f234e8cdb207396865e11c.png);
        }
    </style>
</head>
<body>
    <script src="bundle.js"></script>
    <div class="hello">Hello webpack.
        <img src="489a185362f234e8cdb207396865e11c.png">
    </div>
</body>
</html>
```

最终的效果看上去就是这样：
![添加图片效果图](./doc/添加图片效果图.png)

### 加载字体

TODO:woff、ttf、otf etc.

### 加载数据

TODO:JSON、XML、CSV etc.

## 管理输出

1.代码可以不必都打在一个bundle中
2.自动生成的index.html
	原来的index.html内容会怎么处理？彻底覆盖？
	如何通过模板来生成index.html
3.清理dist文件夹
4.webpack是如何实现文件改名后仍然在index.html中有正确的映射关系的？
	有个manifest

## 开发环境下使用webpack

1.source map
	inline-souce-map。和ts配合时呢？
2.自动编译
	watch模式；
	自动刷新页面的server；
3.和文本编辑器的配合

## 生产环境下使用webpack：

Tree Shaking：移除没使用的代码
	webpack会不导出
	uglify负责移除

生产环境配置：
	config拆分共用
	merge是否可以用最新的assign方法来替代
	npm脚本
    代码压缩
    source map
    指定环境

## 代码分离

两个entry配合[name]变量？
防止重复。
    CommonsChunkPlugin
        针对entry中的每个条目定义一个CommonsChunkPlugin对象
        如果entry中的某个条目是数组形式的，这个条目也只需要一个CommonsChunkPlugin对象
        提取node_modules下公共库到一个文件中，常用的配置写法：(这样岂不要把依赖的模块都写上？就麻烦了)
            new webpack.optimize.CommonsChunkPlugin({
                name: "vendor",         /*在entry中一般用vendor名称的数组来指明vendor*/
                minChunks: function(module){
                return module.context && module.context.indexOf("node_modules") !== -1;
                }
            }),
        如果给插件传递的参数中，name字段在entry中不存在同名条目，那么就是提取webpack的脚手架代码，并且脚手架代码要在最后提取（也就是最后一个CommonsChunkPlugin对象）
            new webpack.optimize.CommonsChunkPlugin({
                name: "manifest",       /*一般用manifest来作为脚手架的名称，除非和entry中名字冲突*/
                minChunks: Infinity
            })
        如果传递的参数中不传递name或者names字段，但是把children设置为true，是不是可以解决要明确写明白所有使用到的库的问题？

    webpack-bundle-analyzer bundle体积分析
        webpack.config.js
            var BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin;
            // ...
            plugins: [new BundleAnalyzerPlugin()]
            第一步：
            webpack --profile --json > stats.json
            第二步：
            webpack --profile --json | Out-file 'stats.json' -Encoding OEM

## 懒加载

## 缓存

通过output中的[name][chunkhash]变量
处理webpack的脚手架代码，避免干扰chunkhash的值
    entry中没有的name
模块标识符解决新模块引入干扰库的chunkhash值计算
    new webpack.HashedModuleIdsPlugin(),

## 创建库

有需求的时候再研究

## Typescript

typescript和awesome-typescript-loader
module.exports = {
 
  // Currently we need to add '.ts' to the resolve.extensions array. 
  resolve: {
    extensions: ['.ts', '.tsx', '.js', '.jsx']
  },
 
  // Source maps support ('inline-source-map' also works) 
  devtool: 'source-map',
 
  // Add the loader for .ts files. 
  module: {
    loaders: [
      {
        test: /\.tsx?$/,
        loader: 'awesome-typescript-loader'
      }
    ]
  }
};

### 用typescript写配置文件

npm install --save-dev typescript ts-node @types/node @types/webpack

ts-node用来通过node执行ts文件

import * as webpack from 'webpack';
import * as path from 'path';
declare var __dirname;

const config: webpack.Configuration = {
  entry: './foo.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'foo.bundle.js'
  }
};

export default config;

然后同样执行webpack就会找到此配置文件

## 全局变量方式使用库

 externals: {
        "react": "React",
        "react-dom": "ReactDOM"
    },