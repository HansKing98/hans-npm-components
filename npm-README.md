# 一文搞懂前端组件发布 npm 库

前端的组件都是基于 javascript 开发，然后用 node.js 打包，发布到 npm 的。 所以我们要做组件发布，首先要了解 npm 包的开发与发布。

## npm 包的开发与发布

### 项目初始化

我们常常用 npm init 命令来初始化 node 项目；如果使用默认的设置，则可加入参数 -y。 下面，我们新建一个 npm-components 文件夹，然后初始化项目：

```bash
bash
复制代码$ mkdir npm-components
$ cd npm-components
$ npm init -y
```

此时，会在项目文件夹中出现一个 package.json 文件：

```json
json
复制代码{
  "name": "npm-components", // 组件名
  "version": "1.0.0", // 组件版本
  "description": "", // 组件描述
  "main": "index.js", // 组件入口
  "scripts": { // 组件脚本
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "", // 组件作者
  "license": "ISC" // 开源协议
}
```

package.json 文件中的 name、version、main、script 是常用的几个配置。

name 和 version 顾名思义是包名和版本。name 命名要注意最好是小写加中划线（-）的写法，version 规范可以参考下一小节。

script 则是可以运行在命令行工具的脚本。对 linux 脚本不熟的同学，可以先在命令行工具中调试完成，再复制到 script 里。

main 会作为整个组件的入口，默认是 index.js，可以根据实际需要进行修改。

我们先按默认设置，在项目文件夹中，新建一个 index.js：

```javascript
javascript
复制代码// index.js
const hello = 'hello';
const world = 'world';
module.exports.log = function() {
	console.log(hello + world);
};
```

一个最简单的 npm 项目就完成了，这里仅包含 `index.js` 和 `package.json`两个文件。

package.json 除了以上配置之外，我们还可能会配置到

- private 属性，是否私有项目，私有项目一般不对外公布。我们现在要把 npm 发布，所以要把这个 private 删除，或者设为 false。非开源项目，加上`private: false`可以避免将其发布到公网上。
- files，需要发布到 npm 的文件。
- dependencies 项目依赖库，这里会显示通过 `npm install` 安装到项目中的库；
- devDependencies 项目开发环境依赖库，这里会显示通过 `npm install --save-dev` 安装到项目中的库。

### npm version 与版本规范

开发完成就可以给组件打上版本号了。

按语义化版本控制规范 SemVer，版本格式为：major.minor.patch，版本号递增规则如下：

1. 主版本号 major：当你做了不兼容的 API 修改，
2. 次版本号 minor：当你做了向下兼容的功能性新增，
3. 修订号 patch：当你做了向下兼容的问题修正。

先行版本号及版本编译信息可以加到“major.minor.patch”的后面，作为延伸。如：1.0.0-alpha.0

每次发布前，需要确定一下更新的内容，选择版本号，使用 npm version 打上版本号

```bash
bash
复制代码$ npm version [patch|minor|major]
```

此时，package.json 中的 version 会在相应的版本上加 1，有的命令行工具中也会看到版本号的变化。

```bash
bash
复制代码~/npm-components  (npm-components@1.0.0) 
$ npm version patch
v1.0.1

~/code/npm-components  (npm-components@1.0.1) 
$ 
```

### 发布到 npm

开发完成，标记版本号之后，发布到 npm 只需要按部就班即可：

1. 如果是公网的开源项目，需要去 npm 官网注册个账号；

2. 如果贵公司有内部的 npm 库，只需要找到 npm 管理人员给你添加即可；

3. 如果平时使用其他源，需要切换到 npm（可安装 nrm 来管理多个源）：

   ```bash
   bash
   复制代码$ nrm ls
   
     npm -------- https://registry.npmjs.org/
     yarn ------- https://registry.yarnpkg.com/
     cnpm ------- http://r.cnpmjs.org/
     taobao ----- https://registry.npm.taobao.org/
   
   $ nrm use npm
   ```

4. 在项目根目录下的命令行工具，运行`npm login`，会提示输入个人信息，完成登录。

5. 运行，会进行上传；

   ```bash
   bash
   复制代码$ npm publish
   ```

6. 如果上传过程顺利，没报出红色错误信息，在 [www.npmjs.com/](https://link.juejin.cn/?target=https%3A%2F%2Fwww.npmjs.com%2F) 就可以看到你发布的包了。

7. 发布的包在72小时内是可以删除的，过了72小时就永远无法删除了，所以记得不要随意发一些没有意义的包。 如果需要卸载，在发布后72小时内执行：

   ```bash
   bash
   复制代码# npm unpublish  <pkg>[@<version>]
   $ npm unpublish npm-components@1.0.1
   ```

至此，一个简单项目的 npm 包发布完成。

## 模块化

真实的项目中，我们往往不可能仅仅写一个 js，一个 js 也不可能写太长，多了就要拆分到多个js。这个时候，就需要用到封装了。

我们把常用的内容封装在一起，就是一个简单的模块。这整个过程可以成为是一个模块化的过程。

模块化帮我们实现了代码复用，也帮我们做到了模块内的数据、方法私有。

### 前端模块化演进

前端的模块化是一个演进的过程，经历了 4 个阶段：

- **全局 function模式 : 将不同的功能封装成不同的全局函数**
- **namespace模式 : 简单对象封装**
- **IIFE模式：匿名函数自调用(闭包)**
- **IIFE模式增强: 引入依赖**

基本原理是将模块挂载在 window 属性下。到 IIFE 增强阶段，现在的模块化规范基本已经成型，有了明显的引入导出。如下代码的引入 jQuery 和暴露 myModule：

```javascript
javascript
复制代码// module.js文件
(function(window, $) {
  let data = 'www.baidu.com';
  // 操作数据的函数
  function foo() {
    // 用于暴露有函数
    console.log(`foo() ${data}`);
    $('body').css('background', 'red');
    otherFun(); // 内部调用
  }
  function otherFun() {
    // 内部私有的函数
    console.log('otherFun()');
  }
  // 暴露行为
  window.myModule = { foo };
})(window, jQuery) // jQuery 作为参数引入
```

在此基础上，逐渐演化出了 AMD、CommonJS、CMD、UMD 等规范。

### AMD（Asynchromous Module Definition - 异步模块定义）

AMD 更多用于浏览器端，需要异步加载各模块，然后再去执行内部代码。是 RequireJS 在推广过程中对模块定义的规范化产出，推崇依赖前置。

```javascript
javascript
复制代码// 定义没有依赖的模块
define(function(){ return 模块 });

// 定义有依赖的模块
define(['module1', 'module2'], function(m1, m2){ return 模块 });

// 引入使用模块
require(['module1', 'module2'], function(m1, m2){ 使用m1/m2 });
```

### CommonJS 规范

CommonJS 是服务端模块的规范，由于Node.js被广泛认知。根据CommonJS规范，一个单独的文件就是一个模块。加载模块使用require方法，该方法读取一个文件并执行，最后返回文件内部的module.exports对象。

```javascript
javascript
复制代码//module1.js
moudle.exports = { value: 1 };

//module2.js
var module1 = require('./module1');
var value2 = module1.value + 2;
module.exports ={ value: value2 };
```

CommonJS 加载模块是同步的，所以只有加载完成才能执行后面的操作。

### CMD（Common Module Definition - 公共模块定义）

CMD 是 SeaJS 在推广过程中对模块定义的规范化产出，同时 CMD 也是延自 CommonJS Modules/2.0 规范。对于模块的依赖，CMD 是延迟执行，推崇依赖就近。

```javascript
javascript
复制代码define((require, exports, module) => {
  module.exports = {
    fun1: () => {
       var $ = require('jquery'); // 执行 fun1 时，再加载
       return $('#test');
    } 
  };
});
```

如上代码，只有当真正执行到fun1方法时，才回去执行jquery。

### UMD 规范

那有没有一种规范同时同时兼容 AMD 和 CommonJS，既可以适用浏览器端，有可以适用于服务器端？

有的，就是 UMD 规范。UMD 规范甚至都不能称作一个规范，它是 AMD 和 CommonJS 的一个糅合。是一段固定的代码写法。如下的工厂模式：

```javascript
javascript
复制代码((root, factory) => {
  if (typeof define === 'function' && define.amd) {
    //AMD
    define(['jquery'], factory);
  } else if (typeof exports === 'object') {
    //CommonJS
    var $ = requie('jquery');
    module.exports = factory($);
  } else {
    //都不是，浏览器全局定义
    root.testModule = factory(root.jQuery);
  }
})(this, ($) => {
  //do something...  这里是真正的函数体
});
```

### ES module 规范

另一种支持服务端和浏览器端的规范就是 ES module 了，即我们现在最常用的`import` 和 `export`：

```javascript
javascript
复制代码export default ...;
import xxx from '';

export ...;
import { xxx } from '';
```

但目前也仅是大于 13.2 的 Node.js 版本才支持 ES 模块化，还需要等待很长的时间全面使用。

所以，出于兼容性考虑，我们仍然选择 UMD 规范进行开发。**（处于淘汰边缘的UMD、CMD、AMD这些规范，大家不需要理解，只需要知道有这么一回事即可）**

## Webpack 打包

写 UMD 的过程重复且繁琐，这时候就需要用工具来完成了，Webpack 也就出现了。Webpack 配置相当简单，以我们的项目为例。

先安装 webpack

```bash
bash
复制代码$ npm install --save-dev webpack webpack-cli
```

新建 webpack 配置文件，webpack.config.js。用 webpack 给组件库打包输出符合 UMD 规范的代码，只需要在基本配置稍作修改即可：

```javascript
javascript
复制代码// webpack.config.js
const path = require('path');

module.exports = {
    mode: 'production',
    entry: './index.js',
    externals: 'lodash', // library包中有引入lodash，打包时不将lodash打进去，用户在引入该library时，需自己再引入lodash，避免用户重复引入lodash，导致文件过大。
    output: {
        path: path.resolve(__dirname, 'dist'),
        filename: 'library.js',
        library: 'library', // 全局挂载包的引用名
        libraryTarget: 'umd',  //通用模式：支持用户通过es、common.js、AMD的方式引入npm包
        globalObject: 'this' // node 等环境运行时需要设置为 this
    }
}
```

要修改的地方如下：

1. filename：打包产物library的名称；
2. externals: ‘lodash’, library 包中有引入 lodash，打包时不将 lodash 打进去，用户在引入该 library 时，需自己再引入 lodash，避免用户重复引入 lodash，导致文件过大；
3. libraryTarget: ‘umd’ 使用 `UMD规范` 支持用户通过 es、common.js、AMD 的方式引入 npm 包；
4. library: ‘library’ 会在全局变量中增加一个liabray的变量，用于挂载该包，主要用于通过脚本形式全局引入时的配置；
5. globalObject: 'this'，为 webpack 4 新增属性，需要指定 global 的值为 ’this‘，否则会为默认值 ’self‘，无法在 nodejs 环境中使用。

在 package.json 中加上脚本

```json
json
复制代码{
...
    script: {
        "build": "webpack" 
    }
}
```

运行脚本

```bash
bash
复制代码npm run build
```

此时，webpack就会开始打包。 我们打开打包后的文件`library.js`，就会发现我们前面10行 UMD 规范的代码。以下是不压缩不混淆（Webpack.config.js 设置`optimization: { minimize: false }`）打包后的代码：

```javascript
javascript
复制代码(function webpackUniversalModuleDefinition(root, factory) {
        if(typeof exports === 'object' && typeof module === 'object')
                module.exports = factory();
        else if(typeof define === 'function' && define.amd)
                define([], factory);
        else if(typeof exports === 'object')
                exports["library"] = factory();
        else
                root["library"] = factory();
})(this, function() {
return /******/ (() => { // webpackBootstrap
/******/ 	var __webpack_modules__ = ({
/***/ 10:
/***/ ((module) => {
const hello = 'hello';
const world = 'world';
module.exports.log = function() {
	console.log(hello + world);
};
/***/ })
...
```

至此，一个支持浏览器端和服务端调用的组件打包完成，可以再次打个版本号，发布到 npm 库了。

## 本地测试

我们可以使用 npm 的包进行测试，也可以在开发阶段引入本地打好的包，进行简单的测试。 我们遵守了 UMD 规范，因此可以在 node 环境和浏览器环境都验证一下：

### node 环境验证

项目根目录新建一个 index.test.js

```javascript
javascript
复制代码let library = require('./dist/library.js');
library.log();
```

本地执行

```bash
bash
复制代码$ node index.test.js
helloworld
```

### 浏览器环境验证

项目根目录新建 public 文件夹，再在此文件夹下新建一个 index.html

```html
html
复制代码<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>测试页面</title>
</head>
<body>
<script src='../dist/library.js'></script>
<script>
library.log(); // 控制台输出 helloworld
</script>
</body>
</html>
```

此时，打开此文件夹，即可以看到 控制台输出 helloworld。

如果还要更真实的环境，还可以使用本地服务器做验证。

首先，安装 webpack 开发服务器：

```bash
bash
复制代码$ npm install --save-dev webpack-dev-server
```

然后，修改package.json，新增 webpack 服务器脚本：

```json
json
复制代码{
...
    "script":{
        "serve": "webpack serve --open",
        ...
    }
}
```

修改 webpack.config.js

```javascript
javascript
复制代码const path = require('path');
module.exports = {
    devServer: {
        static: path.join(__dirname, "./") // 将项目根目录设为静态目录
    },
    ...
}
```

运行服务器

```bash
bash
复制代码$ npm run serve
```

webpack 会默认打开浏览器。可以看到浏览器控制台输出 helloworld。

验证通过。大功告成！！！

作者：陈佬昔没带相机
链接：https://juejin.cn/post/7044102165400387597
来源：稀土掘金
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。