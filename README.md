# **WHAT**

> Modular programming is a software design technique that emphasizes separating the functionality of a program into independent, interchangeable modules, such that each contains everything necessary to execute only one aspect of the desired functionality.

> 模块化编程是一种强调将程序的功能分离成独立的可互换模块的软件设计技术，使得每个模块包含仅执行所需功能的一个方面所需的一切。

可以理解为，开发一个程序功能的过程中，需要用到一些变量、函数以及代码之间的逻辑调用。将这些用于同一个功能的内容结合在一起，并且同其他不相关的内容分离开来。这样一个编程开发的过程就是模块化编程。

# **WHY**
- **代码复用**
- **代码解耦**
- **合作开发**
- **单元测试**
- **...**

# **HOW**
## **JavaScript对象**
~~~
var moduleA = {
  paramA: 'This is paramA',
  paramB: 'This is paramB',
  
  funcA: function() {
    return 'This is funcA';
  },
};

moduleA.paramA;   // 'This is paramA'
moduleA.paramB;   // 'This is paramB'
moduleA.funcA();  // 'This is funcA'
~~~
## **立即执行函数IIFE**
~~~
var moduleA = (function() {
  var paramA = 'paramA';
  var paramB = 'paramB';

  var funcA = function() {
    return 'funcA';
  };

  return {
    paramA: paramA,
    funcA: funcA,
  };
})();

moduleA.paramA;   // 'paramA'
moduleA.funcA();  // 'funcA'
moduleA.paramB;   // undefined
~~~
## **CommonJS**

适用于服务器端的规范，采用同步加载模块。
~~~
/* math.js */
exports.increment = function(a) {
  return ++a;
};

/* index.js */
var increment = require('./math').increment;

increment(1);  // 2
~~~
## **AMD/require.js & CMD/sea.js**

相同点：适用于浏览器端的规范，采用异步加载模块。

不同点：**AMD依赖前置，提前执行**；**CMD依赖就近，延迟执行**。

- **AMD**
~~~
/**
 * math.js
 * 依赖jQuery,和自定义的myLib模块
 */
define(['jQuery', './myLib'], function($, myLib) {
  // jQuery和myLib模块已经下载并执行完毕
  $.get('http://www.example.com');

  function increment(a) {
    myLib.doSomething();

    return ++a;
  }

  return {
    increment: increment
  };
});

/* index.js */
require(['jQuery', './math'], function(math) {
  console.log(math.increment(1));
});

/* index.html */
<script src="./require.js" data-main="./index"></script>

// 2
~~~

- **CMD**
~~~
/**
 * math.js
 * 依赖jQuery
 */
define(function(require, exports, module) {
  var $ = require('jQuery'); // 同步下载并执行jQuery模块
  
  $.get('http://www.example.com');

  function increament(a) {
    return ++a;
  }

  module.exports = {
    increament: increament
  };
});

/* index.js */
seajs.use(['jQuery', './math'], function($, math) {
  console.log(math.increament(1));
});

/* index.html */
<script src="sea.js"></script>
<script src="index.js"></script>

// 2
~~~

## **UMD**

复合类型，跨平台解决方案，支持AMD和CommonJS模块方式。

其原理如下：
1. 判断**exports**是否存在，若存在则为CommonJS模块。
2. 判断**define**是否存在，若存在则为AMD模块。
3. 若都不存在，则将模块公开至全局**window**。

## **Browserify & Webpack**
- **Browserify**

Browserify可将遵循CommonJS规范的模块包装后，直接在浏览器端使用，因此在编写模块代码的时，可做到一套代码，跨平台执行

~~~
/* 安装依赖 */
$ npm i -g browserify

/* 打包模块 */
$ browserify main.js -o bundle.js

/* 实时监测 */
$ npm i -g watchify

$ watchify main.js -o bundle.js -v

/* 模块引用 */
<script src="bundle.js"></script>

~~~

- **Webpack**

> webpack 是一个现代 JavaScript 应用程序的模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成少量的 bundle - 通常只有一个，由浏览器加载。

> webpack**四个核心概念**为：**入口(entry)、输出(output)、loader、插件(plugins)**。

~~~
/* webpack.config.js */
var path = require('path');
var webpack = require('webpack');
var args = require('minimist')(process.argv.slice(2));

var config = {
  entry: './app/index.js',
  output: {
    path: path.resolve(__dirname, 'build/assets'),
    filename: 'bundle.js'
  },
  module: {
    preLoaders: [{
      test: /\.js$/,
      loader: 'eslint-loader',
      configFile: './.eslintrc',
      exclude: /node_modules/
    }],
    loaders: [{
      test: /\.js$/,
      loader: 'babel-loader',
      query: {
        presets: ['es2015']
      },
      exclude: /node_modules/
    }, {
      test: /\.css|scss?$/,
      loader: 'style-loader!css-loader!sass-loader?outputStyle=expanded'
    }, {
      test: /\.(png|gif|jpe?g|svg)$/i,
      loader: 'url-loader',
      query: {
        limit: 10000
      }
    }]
  },
  plugins: []
};

config.plugins = (args.env === 'dev') ? [] : [ new webpack.optimize.UglifyJsPlugin() ];

module.exports = config;
~~~
~~~
/* package.json */

{
  "name": "webpack-es2015",
  "version": "1.0.0",
  "description": "webpack+es2015",
  "main": "index.js",
  "scripts": {
    "start": "node server.js --env=dev",
    "serve": "node server.js --env=dev",
    "test": "echo \"Error: no test specified\" && exit 1",
    "dist": "npm run clean && npm run copy & webpack --env=dist",
    "copy": "copyfiles -f ./app/index.html ./build",
    "clean": "rimraf build/*"
  },
  "author": "343938328@qq.com",
  "license": "MIT",
  "devDependencies": {
    "babel-eslint": "^6.1.2",
    "babel-loader": "^6.2.5",
    "babel-preset-es2015": "^6.14.0",
    "copyfiles": "^1.0.0",
    "css-loader": "^0.25.0",
    "eslint": "^3.5.0",
    "eslint-loader": "^1.5.0",
    "file-loader": "^0.9.0",
    "minimist": "^1.2.0",
    "node-sass": "^3.9.3",
    "sass-loader": "^4.0.2",
    "style-loader": "^0.13.1",
    "url-loader": "^0.5.7",
    "webpack": "^1.13.2",
    "webpack-dev-server": "^1.15.1"
  },
  "dependencies": {
    "normalize.css": "^4.2.0"
  }
}

~~~

## **ES6**
export输出变量
~~~
/* math.js */
export var a = 1;

/* index.js */
import { a } from './math.js';

a; // 1
~~~

export输出函数
~~~
/* math.js */
export function increatment(a) {
  return ++a;
};

/* index.js */
import { increatment } from './math.js';

increatment(1); // 2
~~~

export输出对象
~~~
/* math.js */
export class MyMath {
  constructor(a) {
    this.a = a;
  }

  increatment() {
    return ++this.a;
  }
}

/* index.js */
import { MyMath } from './math.js';

const m = new MyMath(1);

m.increatment(); // 2
~~~

---

[Modular programming](https://en.wikipedia.org/wiki/Modular_programming)

[CommonJS](http://wiki.commonjs.org/wiki/Modules/1.1)

[AMD](https://github.com/amdjs/amdjs-api/wiki/AMD)

[UMD](https://github.com/umdjs/umd)

[Browserify](http://browserify.org/)

[Webpack](https://webpack.js.org/)

[ES6 module](https://babeljs.io/learn-es2015/#ecmascript-2015-features-modules)
