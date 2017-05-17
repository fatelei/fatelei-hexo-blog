title: Webpack 2.0 执行流程
date: 2017-05-16 23:46:31
tags: [webpack]
---

Webpack 2.0 的文档 [https://webpack.js.org/concepts/](https://webpack.js.org/concepts/)，介绍了其最为核心的几个概念

<!-- more -->

主要有：

- entry
- output
- loaders
- plugins

以下面的代码为例：

- foo.js

```javascript
export default () => {
  console.log('foo')
}
```

- bar.js

```javascript
import foo from './foo'

foo()
```

- webpack.config.js

```javascript
const webpack = require('webpack'); //to access built-in plugins

module.exports = {
  entry: 'bar.js',
  output: {
    filename: 'bundle.js'
  },
  module: {
    rules: [
      {
        test: /\.js$/,
        use: 'babel-loader'
      }
    ]
  },
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
  ]
}
```

在执行 `webpack` 命令后，其流程如下：

![webpack process](/images/webpack.process.png)

1. 首先根据配置的 entry，找到 bar.js
2. 在 bar.js 中发现加载了 module foo，这个时候执行 module 的 rules
3. 在执行 module 中的 rules 之后，会执行 plugins
4. 2、3 两步会重复执行，直到没有 module
5. 最后通过指定的 output，输出到 bundle.js

