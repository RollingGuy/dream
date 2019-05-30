# 什么是babel
babel是一个JavaScript编译器，

# babel-cli
babel-cli只是使用babel的一种方式，因为有的项目不太大，用不到webpack等构建工具，所以在发布之前使用babel-cli即可。
## 基本用法
```shell
  $ npm install -g babel-cli
```
然后可以将某个文件编译后输出：
```shell
  $ babel index.js --out-file index.js
  # 简写为
  $ babel index.js -o index.js
  # 如果想要输出到某个新的目录
  $ babel src --out-dir lib
  # 或者简写为
  $ babel src -d lib
```

## 本地安装
```shell
  $ npm install -D babel-cli
```
然后可以使用`./node_modules/.bin/babel`命令运行即可。

如果嫌太麻烦，可以直接写到`package.json`的scripts字段中：
```json
  {
    "start": "./node_modules/.bin/babel src -d lib"
  }
```

# babel-core
有许多在线babel转译的网站，这个就是用babel-core做到的：
```javascript
  const babel = require('babel-core');
  const code = `
    class Foo {
      constructor(props) {
        console.log(1)
      }
    }
  `;
  // 其他配置文档很清楚，这里只举个栗子
  const result = babel.transform(code, {
    code: true, // 生成代码
    presets: [],  // 这个和.babelrc文件里面的presets一样，具体请看下文
    plugins: [], // 同上
  })；
  console.log(result.code);
  // 结果就是转译后的code
```
> 可以看到，比起上面的`babel-cli`，`babel-code`提供了使用编程式的转译功能.

还可以对文件进行转译：
```javascript
babel.transformFile(filename, options, (err, result) => {
  //
});
```
> 还有其他一些东东，具体看[文档](https://www.babeljs.cn/docs/core-packages/#options)去🙃。

# 说好的转译呢？
通过上面几部，会发现，都只是将代码原封不动的挪了个地儿。不是说可以转译么。

> 因为`babel`一通过各种花样去玩儿的编译器，但是默认情况下它反而球都不干。所有就需要对`babel`进行配置，也就是眼熟的`.babelrc`文件。

## .babelrc
在项目根目录下创建`.babelrc`文件，然后进行配置：
```json
{
  "presets": [],
  "plugins": [],
}
```
> `.babelrc`只是配置babel的一种方式，也可以通过其他方式给babel传递参数，但是`.babelrc`是约定的最好方式。

> 还可以将这个文件的内容写在`package.json`的`babel`字段中：
```json
{
  "babel": {
    "presets": [],
    "plugins": [],
  }
}
```

### 查找
`Babel` 会在正在被转录的文件的当前目录中查找一个 `.babelrc` 文件。 如果不存在，它会遍历目录树，直到找到一个 `.babelrc` 文件，或一个 `package.json` 文件中有 `"babel": {}` 。

在选项中使用 `"babelrc": false` 来停止查找行为，或者提供`--no-babelrc CLI` 标志。

## 现在该说怎么转译了
通过使用`babel-preset-es2015`就可以将ES2015+的语法转译为ES5了：
```shell
$ npm install -D babel-preset-es2015
```
```json
{
  "presets": [
    "es2015"
  ]
}
```
这下什么箭头函数、let、const就都被转译为ES5对应的同等功能的语法了。

## babel-polyfill
执行了上面的操作会发现，只有ES2015+的语法被转译了，其他新的API还是原封不动。是不是有种被骗的感觉，我都支持API了，语法能不会？QNMLGB。。

所以还需要一个东东：`babel-polyfill`
```shell
$ npm install --save babel-polyfill
```
> 看仔细，是save，不是save-dev。因为这是一个 polyfill （它需要在你的源代码之前运行），我们需要让它成为一个 dependency, 而不是一个 devDependency.

接下来只需要在使用到新API的文件顶部引入它就可以了。
```javascript
import 'babel-polyfill';
```
`babel-polyfill`的两个主要缺点：
1. 使用`babel-polyfill`会导致打出来的包非常大，因为`babel-polyfill`是一个整体，它把所有的静态方法都加到了原型链上(也包括没有用到的方法)，这个问题可以通过使用`core-js`的某个类库来解决，`core-js`都是分开的。
2. `babel-polyfill`会污染全局环境，因为它对许多类的原型都做了修改。

## babel-plugin-transform-runtime
在不使用这个插件的时候，每个文件所用到的公共方法都会生成一次，而使用之后就会将这些公共方法放到一个文件，然后每个文件`require`就可以了。

也就是说，这个插件把重复定义变为重复引用，解决了代码重复问题。
> 但是它不支持实例方法，比如：`[].includes();`，而且它需要`babel-runtime`作为依赖。`babel-plugin-transform-runtime`是按需引入静态方法，对于实例上的方法是无能为力的，解决办法是：`babel-polyfill`或`babel-preset-env`,推荐后者，开启useBuiltIns。

### babel-runtime和babel-plugin-transform-runtime的区别
之前一直错误的以为babel-runtime也是一个plugin，现在发现babel-runtime只是一个库，一个模块，如果环境不支持Promise，可以在项目中加入：
```js
require('@babel/runtime/core-js/promise');
```

而`babel-plugin-transform-runtime`的工作就是判断文件需要引入`core-js`中的哪个，并引入其需要的方法库。



## preset
因为`es2015`是一套规范，包含几十个转译插件，如果需要一个个添加安装，那配置文件就没法看了，而且下载时间也比较长。

所以`babel`提供了一组插件集合。

`preset`分为以下几种：
1. 官方内容: `env, react, flow, minify`等
2. `stage-x`, 每年更新，是当年的最新规范草案，细分为:
    1. stage0-稻草人：只是一个idea，经过TC39成员提出即可。
    2. stage1-提案：初步尝试
    3. stage2-初稿：完成初步规范
    4. 完成，被添加到下一年发布。
3. `es2015+`, `latest`: 已经纳入标准规范中的语法。`latest`的`env`的出行，他是每年更新的`preset`，目的是包含所有的`2015+`，但是更加灵活的`env`的出现，现在它已经废弃。

## 插件和preset的配置
通常情况下，只需要列出字符串格式的名字即可。如果某个preset或plugin需要一些配置，那就需要将该项改为数组。

+ 第一个元素：表示自己名字的string
+ 第二个元素：一个配置对象
```json
{
  "presets": [
    [
      "env",
      {
        "module": false,
      }
    ],
    "stage-0",
  ]
}
```

## env
最常用，也最重要。

`env`的核心目的是通过配置得知目标环境，然后只做必要的转换。如果不写任何配置，`env`等于`latest`，就是`es2015+`的所有集合（不包括`stage`）。

env示例配置：
```js
{
  "presets": [
    [
      "env",
      {
        "targets": {
          // 如果是浏览器的最新2个版本或者safari7.0+，则不做转换
          "browser": ["last 2 versions", "safari >= 7"]
        },
        "useBuiltIns": false, // 默认为false，具体功能看下面
      }
    ]
  ]
}
```
还可以是`node`环境
```js
{
  "presets": [
    [
      "env",
      {
        "targets": {
          "node": "6.10", // current是最新稳定版
        }
      }
    ]
  ]
}
```
上面提到一个参数`useBuiltIns`,这个参数意思是：是否为每个文件自动添加插件兼容，或者把`import "@babel/polyfill"`根绝不同目标环境拆分为多个兼容插件。如：
```js
import '@babel/polyfill';
```
根据不同的环境，输出：
```js
import "core-js/modules/es7.string.pad-start";
import "core-js/modules/es7.string.pad-end";
```

这个参数在`babel6`中只有`true、false`,在`babel7`中取值为：`"usage"|"entry"|false`,默认为`false`<br>详细配置请看：https://babeljs.io/docs/en/babel-preset-env#usebuiltins

#执行顺序
+ `plugin`会运行在`preset`之前
+ `plugin`会从前到后执行(和`webpack`的`loader`相反)
+ `preset`的执行顺序是从后向前，和`webpack`的`loader`相同。
> `preset`的逆向顺序是为了保证向后兼容，因为大多数的编写顺序是`['es2015', 'stage-0']`


# 参考
`babel`的文档写的我感觉是真的有点搓。下面是一些我比较喜欢的文章：

[Babel 用户手册](https://blog.csdn.net/sinat_34056695/article/details/74452558)

[一口(很长的)气了解 babel](https://juejin.im/post/5c19c5e0e51d4502a232c1c6)

[深入Babel，这一篇就够了](https://juejin.im/post/5c21b584e51d4548ac6f6c99)

[click me to fuck babel](https://github.com/jamiebuilds/babel-handbook/blob/master/translations/zh-Hans/README.md)