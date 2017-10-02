# learn-webpack

webpack是一个打包工具

## 安装

安装webpack
npm init -y
npm i webpack --save-dev
webpack -v

一个普通项目
mkdir src
touch index.html
touch src/index.js
编辑 index.html
编辑 src/index.js
用浏览器查看 index.html

通过模块的方式使用库
mkdir dist
mv index.html dist
npm i lodash
编辑 src/index.js
编辑 dist/index.html
webpack src/index.js dist/bundle.js
用浏览器查看 dist/index.html
PS:如果index.js中不使用import引入lodash模块，那么webpack打包时就不能找到依赖关系，就不会打包lodash的代码，页面就无法浏览了。

通过webpack配置文件打包
touch webpack.config.js
编辑webpack.config.js
webpack
