# learn-webpack

webpack是一个打包工具

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