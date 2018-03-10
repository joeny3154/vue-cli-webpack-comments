
``` js
'use strict'
const path = require('path')
const config = require('../config')
const ExtractTextPlugin = require('extract-text-webpack-plugin')
const packageConfig = require('../package.json')

exports.assetsPath = function (_path) {
  const assetsSubDirectory = process.env.NODE_ENV === 'production'
    ? config.build.assetsSubDirectory
    : config.dev.assetsSubDirectory
  // `path.posix` 属性提供了 path 方法针对 POSIX 的实现
  // http://nodejs.cn/api/path.html#path_path_delimiter
  return path.posix.join(assetsSubDirectory, _path)
}
// 示例： ({ sourceMap: config.dev.cssSourceMap, usePostCSS: true })
exports.cssLoaders = function (options) {
  options = options || {}

  const cssLoader = {
    loader: 'css-loader',
    options: {
      sourceMap: options.sourceMap
    }
  }

  const postcssLoader = {
    loader: 'postcss-loader',
    options: {
      sourceMap: options.sourceMap
    }
  }

  // generate loader string to be used with extract text plugin
  // 生成与提取文本插件一起使用的加载器字符串
  function generateLoaders (loader, loaderOptions) {
    // 设置默认loader
    const loaders = options.usePostCSS ? [cssLoader, postcssLoader] : [cssLoader]

    if (loader) {
      loaders.push({
        loader: loader + '-loader',
        // 生成 options 对象
        options: Object.assign({}, loaderOptions, {
          sourceMap: options.sourceMap
        })
      })
    }

    // Extract CSS when that option is specified
    // (which is the case during production build)
    // 生产模式中提取css， 如果 options 中的 extract 为 true 配合生产模式
    if (options.extract) {
      // loaders: [ { loader: 'css-loader', options: { sourceMap: false } },{ loader: 'postcss-loader', options: { sourceMap: false } } ]
      return ExtractTextPlugin.extract({
        use: loaders,
        // 默认使用 vue-style-loader
        fallback: 'vue-style-loader'
      })
    } else {
      return ['vue-style-loader'].concat(loaders)
    }
  }

  // https://vue-loader.vuejs.org/en/configurations/extract-css.html
  // 返回各种 loaders 对象
  return {
    css: generateLoaders(),
    postcss: generateLoaders(),
    less: generateLoaders('less'),
    sass: generateLoaders('sass', { indentedSyntax: true }),
    scss: generateLoaders('sass'),
    stylus: generateLoaders('stylus'),
    styl: generateLoaders('stylus')
  }
  /*
  返回如下内容：
  {
    css: ['vue-style-loader',
      { loader: 'css-loader', options: { sourceMap: true/false } }
    ],
    postcss: ['vue-style-loader',
      { loader: 'css-loader', options: { sourceMap: true/false } }
    ],
    less: ['vue-style-loader',
      { loader: 'css-loader', options: { sourceMap: true/false } },
      { loader: 'less-loader', options: { sourceMap: true/false } }
    ],
    sass: ['vue-style-loader',
      { loader: 'css-loader', options: { sourceMap: true/false } },
      {
        loader: 'sass-loader',
        options: { indentedSyntax: true, sourceMap: true/false }
      }
    ],
    scss: ['vue-style-loader',
      { loader: 'css-loader', options: { sourceMap: true/false } },
      { loader: 'sass-loader', options: { sourceMap: true/false } }
    ],
    stylus: ['vue-style-loader',
      { loader: 'css-loader', options: { sourceMap: true/false } },
      { loader: 'stylus-loader', options: { sourceMap: true/false } }
    ],
    styl: ['vue-style-loader',
      { loader: 'css-loader', options: { sourceMap: true/false } },
      { loader: 'stylus-loader', options: { sourceMap: true/false } }
    ]
  }
  */
}

// Generate loaders for standalone style files (outside of .vue)
// 为独立样式文件生成装载器（.vue之外）
exports.styleLoaders = function (options) {
  const output = []
  const loaders = exports.cssLoaders(options)

  for (const extension in loaders) {
    const loader = loaders[extension]
    output.push({
      test: new RegExp('\\.' + extension + '$'),
      use: loader
    })
  }

  return output
}

/*
styleLoaders({
  sourceMap: config.build.productionSourceMap, // true
  extract: true,
  usePostCSS: true
})
extract 为 false

[
  {
    test: /\.css$/,
    use:[
      {
        loader: '/Users/wanjun/Desktop/demo/vue-doc/node_modules/extract-text-webpack-plugin/dist/loader.js',
        options: { omit: 1, remove: true }
      },
      { loader: 'vue-style-loader' },
      { loader: 'css-loader', options: { sourceMap: true } },
      { loader: 'postcss-loader', options: { sourceMap: true } }
    ]
  },
]
extract 为true
[
  {
    test: /\.css$/,
    use:
      [{
        loader: '/Users/wanjun/Desktop/demo/vue-doc/node_modules/extract-text-webpack-plugin/dist/loader.js',
        options: { omit: 1, remove: true }
      },
      { loader: 'vue-style-loader' },
      { loader: 'css-loader', options: { sourceMap: true } },
      { loader: 'postcss-loader', options: { sourceMap: true } }]
  },
  {
    test: /\.postcss$/,
    use:
      [{
        loader: '/Users/wanjun/Desktop/demo/vue-doc/node_modules/extract-text-webpack-plugin/dist/loader.js',
        options: { omit: 1, remove: true }
      },
      { loader: 'vue-style-loader' },
      { loader: 'css-loader', options: { sourceMap: true } },
      { loader: 'postcss-loader', options: { sourceMap: true } }]
  },
  {
    test: /\.less$/,
    use:
      [{
        loader: '/Users/wanjun/Desktop/demo/vue-doc/node_modules/extract-text-webpack-plugin/dist/loader.js',
        options: { omit: 1, remove: true }
      },
      { loader: 'vue-style-loader' },
      { loader: 'css-loader', options: { sourceMap: true } },
      { loader: 'postcss-loader', options: { sourceMap: true } },
      { loader: 'less-loader', options: { sourceMap: true } }]
  },
  {
    test: /\.sass$/,
    use:
      [{
        loader: '/Users/wanjun/Desktop/demo/vue-doc/node_modules/extract-text-webpack-plugin/dist/loader.js',
        options: { omit: 1, remove: true }
      },
      { loader: 'vue-style-loader' },
      { loader: 'css-loader', options: { sourceMap: true } },
      { loader: 'postcss-loader', options: { sourceMap: true } },
      { loader: 'sass-loader', options: { indentedSyntax: true, sourceMap: true } }]
  },
  {
    test: /\.scss$/,
    use:
      [{
        loader: '/Users/wanjun/Desktop/demo/vue-doc/node_modules/extract-text-webpack-plugin/dist/loader.js',
        options: { omit: 1, remove: true }
      },
      { loader: 'vue-style-loader' },
      { loader: 'css-loader', options: { sourceMap: true } },
      { loader: 'postcss-loader', options: { sourceMap: true } },
      { loader: 'sass-loader', options: { sourceMap: true } }]
  },
  {
    test: /\.stylus$/,
    use:
      [{
        loader: '/Users/wanjun/Desktop/demo/vue-doc/node_modules/extract-text-webpack-plugin/dist/loader.js',
        options: { omit: 1, remove: true }
      },
      { loader: 'vue-style-loader' },
      { loader: 'css-loader', options: { sourceMap: true } },
      { loader: 'postcss-loader', options: { sourceMap: true } },
      { loader: 'stylus-loader', options: { sourceMap: true } }]
  },
  {
    test: /\.styl$/,
    use:
      [{
        loader: '/Users/wanjun/Desktop/demo/vue-doc/node_modules/extract-text-webpack-plugin/dist/loader.js',
        options: { omit: 1, remove: true }
      },
      { loader: 'vue-style-loader' },
      { loader: 'css-loader', options: { sourceMap: true } },
      { loader: 'postcss-loader', options: { sourceMap: true } },
      { loader: 'stylus-loader', options: { sourceMap: true } }]
  }
]
*/

exports.createNotifierCallback = () => {
  // 发送跨平台通知系统, 基本用法：notifier.notify('message')
  const notifier = require('node-notifier')

  return (severity, errors) => {
    // 当前设定是只有出现 error 错误时触发 notifier 发送通知
    // 严重程度可以是 'error' 或 'warning'
    if (severity !== 'error') return

    const error = errors[0]
    const filename = error.file && error.file.split('!').pop()

    notifier.notify({
      title: packageConfig.name,
      message: severity + ': ' + error.name,
      subtitle: filename || '',
      // 通知图标
      icon: path.join(__dirname, 'logo.png')
    })
  }
}
```
