# 如何用webpack优化moment.js的体积

当你在代码中写了`var moment = require('moment')` 然后用webpack打包, 打出来的包会是很大的，因为打包结果包含了各地的local文件.

![](https://raw.githubusercontent.com/jmblog/how-to-optimize-momentjs-with-webpack/master/source-map-explorer.png)

解决方案是下面的两个webpack插件，任选其一:

1.  `IgnorePlugin`
1.  `ContextReplacementPlugin`

## 方案一：使用 `IgnorePlugin`

我们可以借助`IgnorePlugin`来移除所有本地moment文件，因为我们很多时候在开发中根本不会使用到。
具体做法是在webpack的插件配置项中增加它：

```js
const webpack = require('webpack');
module.exports = {
  //...
  plugins: [
    // 忽略 moment.js的所有本地文件
    new webpack.IgnorePlugin(/^\.\/locale$/, /moment$/),
  ],
};
```

那么你可能会有疑问，所有本地文件都被移除了，但我想要用其中的一个怎么办。不用担心，你依然可以在代码中这样使用：

```js
const moment = require('moment');
require('moment/locale/ja');

moment.locale('ja');
...
```

这个方案被用在 [create-react-app](https://github.com/facebookincubator/create-react-app/blob/a0030fcf2df5387577ced165198f1f0264022fbd/packages/react-scripts/config/webpack.config.prod.js#L350-L355).

## 方案二：使用 `ContextReplacementPlugin`

这个方案其实跟方案一有点像。原理是我们告诉webpack我们会使用到哪个本地文件，具体做法是在插件项中这样添加`ContextReplacementPlugin`：

```js
const webpack = require('webpack');
module.exports = {
  //...
  plugins: [
    // 只加载 `moment/locale/ja.js` 和 `moment/locale/it.js`
    new webpack.ContextReplacementPlugin(/moment[/\\]locale$/, /ja|it/),
  ],
};
```

值得注意的是，这样你就**不需要**在代码中再次引入本地文件了：

```js
const moment = require('moment');
moment.locale('ja');
...
```

## 对比

* webpack: v3.10.0
* moment.js: v2.20.1

|                             | 文件大小 | Gzipped |
| :-------------------------- | --------: | ------: |
| 默认                     |    266 kB |   69 kB |
| 使用IgnorePlugin             |   68.1 kB | 22.6 kB |
| 使用ContextReplacementPlugin |   68.3 kB | 22.6 kB |

