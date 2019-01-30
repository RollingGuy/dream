# 简介
`@babel/preset-env`是一个智能预设，允许使用最新的`JavaScript`，而不需要关注你的目标环境需要哪些语法转换（以及可选的浏览器`polyfill`），因为这些事情`@babel/preset-env`都自动地帮你完成了，不仅如此，由于这些优点还令`JavaScript`的`bundle`更小。

> 其实preset也算是plugin，只不过关于ES2015+的新特性对应的插件太多了，babel官方将这些新特性全部都弄到一起就成了preset的preset-es2015+了。那今天要说的这个@babel/preset-env也是一个道理，本质上也是插件，可以理解为集成的插件集合。

## 背景
最初为了能够让开发者早点用上新js特性，而不必等到主流浏览器支持，babel团队开发了`babel-preset-latest`，它是多个preset的集合(ES2015+)，随着ECMA的规范更新而更新。

但是如果照着这样下去插件会越来越多，编译的速度会变的非常慢；与此同时浏览器的支持度也越来越高，编译到低版本的必要性减少。

为了解决上述问题，babel官方又推出了`babel-preset-env`插件，他可以gun局开发者的需要（配置）来按需加载插件，比如：
+ 需要支持的平台
+ 平台的版本等

> 在没有任何配置选项的情况下，`@babel/preset-env` 与 `@babel/preset-latest`（或者`@babel/preset-es2015，@babel/preset-es2016和@babel/preset-es2017一起`）的行为完全相同。所以他们就全部都被`@babel/preset-env`淘汰了。

# 安装
```shell
npm install -D @babel/preset-env    # babel7
npm install -D babel-preset-env     # babel6, 以下没有特殊说明都使用babel7
```

# 使用
```json
{
  "presets": [
    ["@babel/preset-env", {
      "targets": {
        "browsers": "name version",
      },
      "useBuiltIns": "entry"
    }],
  ]
}
```

## targets
`string | Array<string> | { [string]: string }`, 默认为`{}`。
用于描述项目的目标环境。
### 支持特定版本的浏览器
如果只需要支持某个版本，比如Chrome 63，配置就可以这么写：
```json
{
  "presets": [
    ["@babel/preset-env", {
      "targets": "chrome 63",
    }],
  ]
}
```
### 支持特定范围的浏览器
大部分的时候我们需要针对的都是特定范围的浏览器，比如IE8+,这个时候配置：
```json
{
  "presets": [
    ["@babel/preset-env", {
      "targets": "ie >= 8",
    }]
  ]
}
```
### 支持多种浏览器
```json
{
  "presets": [
    ["@babel/preset-env", {
      "targets": ["ie >= 8", "chrome >= 63"]
    }]
  ]
}
```
或者
```json
{
  "presets": [
    ["@babel/preset-env", {
      "targets": {
        "chrome": "63",
        "ie": "8",
      }
    }]
  ]
}
```

### targets.esmodules
还可以直接定位`支持ES6模块`的浏览器：
```json
{
  "presets": [
    ["@babel/preset-env", {
      "targets": {
        "esmodules": true,
      }
    }]
  ]
}
```
> 这个时候会忽略掉targets的关于浏览器的其他选项。

### targets.node
`string | "current" | true`
以上都是针对浏览器的，如果目标环境是node，这个时候可以：
```js
{
  "presets": [
    ["@babel/preset-env", {
      "targets": {
        "node": "current",
      }
    }]
  ]
}
```
> `current、true`都他娘的指的是`process.version.node`

> `@babel/preset-env`借助了[browserslist](https://github.com/browserslist/browserslist)这个库。如果没有指定targets，`@babel/preset-env`会转换所有ES2015+的代码。


## useBuiltIns
`"usage" | "entry" | false`, 默认为false
一种将`polyfill`应用于`@babel/preset-env`中的方法。

默认为false，意为不对polyfill做处理。
> 这种方式不会像普通的`babel-polyfill`那样的实验性`polyfill/stage-x`内置插件。

### entry
会把`@babel/polyfill`分成好多小块引入。文档上说在项目入口引入一次多次引入会报错（但是我多次引入也没🤗❌啊？？）

插件`@babel/preset-env`会将把`@babel/polyfill`根据实际需求打散，只留下必须的，也就是说，会根据你的targets配置来判断。

### usage
跟上面的不同，这个选项更加智能，它会判断每个文件用到了哪些新特性，如果需要添加`polyfill`则引入，否则不引入。体积更小。

## include
`Array<string>`，默认为[]

如果原生的环境有个bug，或者不支持的功能与支持的功能的组合不起作用，这个选项将会是一个非常有用的选项。例如， <br>
`Node 4` 支持原生类但不支持类扩展。如果 `super` 与扩展参数一起使用，那么需要 `include` 选项 `transform-es2015-classes` 因为如果不是以 `super` 方式进行传播，则不可能进行转译。

## exclude
`Array<string>`，默认为[]，总是移出的插件，取值和include相同

当使用useBuiltIns时，可以包含不适用的插件，这样在转译的时候就不会对这些东西进行转译了。

> include和exclude只适用于preset中包含的插件，如果要使用其他插件请直接添加到config中。

## modules
这个本来是用来将代码转换成不同的模块机制的，但是webpack已经把这件事情做了，所以babel就不需要做了，但是有可能会有冲突，因为modules默认为commonjs，如果有冲突，需要手动将modules设置为false。

# @babel/preset-env实现原理
1.  使用[compat-tabe](https://github.com/kangax/compat-table)来确定浏览器的支持情况
2.  将js的新特性跟特定的babel插件建立[映射关系](https://github.com/babel/babel-preset-env/blob/master/data/plugin-features.js)(不包括stage-x)
3.  根据开发者的配置来确定至少要包含哪些插件。

# 总结
其他的一些参数请看[文档](https://www.babeljs.cn/docs/plugins/preset-env/)