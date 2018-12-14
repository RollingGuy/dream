# 压缩优化
+ `mode:production`开启代码压缩，减小 `bundle`体积。
+ 

# 生产环境优化
+ 避免使用`inline-*`、`eval-*`，因为他们会增加`bundle`大小，降低整体性能。
+ 

# 懒加载
因为有时候只有完成某些操作之后才需要用到某个模块，这就意味着某些模块可能永远也不会被加载。这个时候可以使用`import();`
```javascript
element.on('type', () => {
  import('...')
    .then(module => {
      // some actions;
    })
    .catch(error => {
      // ...
    });
});
```

> 这样某些需要按需引入的模块就不必每次都被请求了，可以减少main bundle的体积，提高初次加载速度。

# 配置优化
可以将原先的`webpack.config.js`，分割为以下形式：
+ `webpack.prod.js`
+ `webpack.dev.js`
+ `webpack.common.js`

将公共配置抽离到`webpack.common.js`中，然后在其他两个文件中分别引入此公共配置，`module.exports = merge(common, newConfig);`并在`package.json`中的`scripts`里对生产环节和开发环境做不同的处理：
```javascript
{
  "scripts": {
    "start": "webpack --config webpack.dev.js",
    "build": "webpack --config webpack.prod.js"
  }
}
```