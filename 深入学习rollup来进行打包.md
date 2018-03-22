## 深入学习rollup来进行打包
+ 什么是Rollup?
rollup是一款用来es6模块打包代码的构建工具(支持css和js打包)。当我们使用ES6模块编写应用或者库时，它可以打包成一个单独文件提供浏览器和Node.js来使用。

+ 它的优点有如下：
 > 1. 能组合我们的脚本文件。
 > 2. 移除未使用的代码(仅仅使用ES6语法中)。
 > 3. 在浏览器中支持使用 Node modules。
 > 4. 压缩文件代码使文件大小尽可能最小化。

 + Rollup最主要的优点
它是基于ES2015模块的，相比于webpack或Browserify所使用的CommonJS模块更加有效率，因为Rollup使用一种叫做tree-shaking的特性来移除模块中未使用的代码，这也就是说当我们引用一个库的时候，我们只用到一个库的某一段的代码的时候，它不会把所有的代码打包进来，而仅仅打包使用到的代码(webpack2.0+貌似也引入了tree-shaking)。

> 注意：Rollup只会在ES6模块中支持tree-shaking特性。目前按照CommonJS模块编写的jquery不能被支持tree-shaking.

+ rollup 的应用场景
现在目前流行的打包有 gulp 和 webpack，那么与前面两个对比，我觉得rollup更适合打包js库，但是对于打包一个项目的整个应用的话，我到觉得webpack更适合，比如打包一些图片，字体等资源文件的时候，webpack很适合，目前貌似没有看到rollup可以做到这些。之所以我来研究rollup，是因为最近在看vuex的源码的时候，看到它的js库就是使用rollup来进行打包的。


## 如何使用Rollup来处理并打包JS文件？
2-1 安装Rollup并创建配置文件，通过如下命令安装：
      进入项目根目录后，
> 运行命令： npm install --save-dev rollup

2-2 在项目的根目录下新建一个新文件 rollup.config.js, 之后再在文件中添加如下代码：

```
export default {
  input: './src/main.js',
  output: {
    file: './dist/js/main.min.js',
    format: 'iife'
  }
}
```

下面再来了解一下各个配置的含义：
input： rollup先执行的入口文件。
output：rollup 输出的文件。
output.format: rollup支持的多种输出格式(有amd,cjs, es, iife 和 umd, 具体看 http://www.cnblogs.com/tugenhua0707/p/8150915.html)
sourceMap —— 如果有 sourcemap 的话，那么在调试代码时会提供很大的帮助，这个选项会在生成文件中添加 sourcemap，来让事情变得更加简单。

我们在package.json代码下 添加如下脚本。

```
"scripts": {
  "build": "rollup -c"
}
```

因此我们只要在命令行中 输入命令：npm run build 即可完成打包；

我们再看下各个文件下的代码：

src/js/a.js 代码如下：
```
export function a(name) {
  const temp = `Hello, ${name}!`;
  return temp;
}
export function b(name) {
  const temp = `Later, ${name}!`;
  return temp;
}
```

src/js/b.js代码如下：

```
/**
 * Adds all the values in an array.
 * @param  {Array} arr an array of numbers
 * @return {Number}    the sum of all the array values
 */
const addArray = arr => {
  const result = arr.reduce((a, b) => a + b, 0);
  return result;
};
export default addArray;
```

src/main.js代码如下：
```
import { a } from './js/a';
import addArray from './js/b';

const res1 = a('kongzhi');
const res2 = addArray([1, 2, 3, 4]);

console.log(res1);
console.log(res2);
```

最终会在项目的根目录下生成文件 dist/js/main.min.js, 代码如下：

```
(function () {
'use strict';

function a(name) {
  const temp = `Hello, ${name}!`;
  return temp;
}

/**
 * Adds all the values in an array.
 * @param  {Array} arr an array of numbers
 * @return {Number}    the sum of all the array values
 */
const addArray = arr => {
  const result = arr.reduce((a, b) => a + b, 0);
  return result;
};

const res1 = a('kongzhi');
const res2 = addArray([1, 2, 3, 4]);

console.log(res1);
console.log(res2);

}());
```

如上可以看到 在 src/js/a.js 下的 b函数没有被使用到，所以打包的时候没有被打包进来。

注意：在上面代码打包后，只有现代浏览器会正常工作，如果要让不支持ES2015的旧版本浏览器下也正常工作的话，我们需要添加一些插件。

## 设置Babel来使旧浏览器也支持ES6的代码

如上打包后的代码，我们可以在现代浏览器下运行了，但是如果我们使用老版本的浏览器的话，就会产生错误。幸运的是，Babel已经提供了支持。
我们首先需要安装一些依赖项如下命令：

```
npm install --save-dev
babel-core babel-preset-env babel-plugin-external-helpers
babel-plugin-transform-runtime babel-preset-stage-2
babel-register rollup-plugin-babel
```
注意：Babel preset 是一个有关Babel插件的集合，它会告诉Babel我们需要转译什么。

3.2 创建 .babelrc文件

接下来需要在项目的根目录下创建 .babelrc的新文件了，它内部添加如下JSON代码：
```
{
  "presets": [
    ["env", {
      "modules": false,
      "targets": {
        "browsers": ["> 1%", "last 2 versions", "not ie <= 8"]
      }
    }],
    "stage-2"
  ],
  "plugins": ["transform-runtime", "external-helpers"]  // 配置runtime，不设置会报错
}
```

