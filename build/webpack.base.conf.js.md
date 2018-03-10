

``` js
'use strict'
const path = require('path')
const utils = require('./utils')
const config = require('../config')
const vueLoaderConfig = require('./vue-loader.conf')

// 生成绝对路径
function resolve (dir) {
  return path.join(__dirname, '..', dir)
}
// 创建 eslint loader所需的 rule配置 
const createLintingRule = () => ({
  test: /\.(js|vue)$/,
  loader: 'eslint-loader',
  enforce: 'pre',
  include: [resolve('src'), resolve('test')],
  options: {
    // formatter: 格式化程序， 
    // 默认是eslint 主流的格式化程序 require("eslint/lib/formatters/stylish"),
    // eslint-friendly-formatter: 一款社区 formatter
    formatter: require('eslint-friendly-formatter'),
    // 错误会显示在浏览器中
    emitWarning: !config.dev.showEslintErrorsInOverlay // showEslintErrorsInOverlay: false
  }
})

module.exports = {
  // 上下文
  context: path.resolve(__dirname, '../'),
  entry: {
    app: './src/main.js'
  },
  output: {
    // dist/ 目录 path.resolve(__dirname, '../dist')
    path: config.build.assetsRoot,
    filename: '[name].js',
    publicPath: process.env.NODE_ENV === 'production'
      ? config.build.assetsPublicPath  // '/'
      : config.dev.assetsPublicPath
  },
  resolve: {
    extensions: ['.js', '.vue', '.json'],
    alias: {
      'vue$': 'vue/dist/vue.esm.js',
      '@': resolve('src'),
    }
  },
  module: {
    rules: [
      ...(config.dev.useEslint ? [createLintingRule()] : []),
      {
        test: /\.vue$/,
        loader: 'vue-loader',
        // 详解见下文
        options: vueLoaderConfig
      },
      {
        test: /\.js$/,
        loader: 'babel-loader',
        include: [resolve('src'), resolve('test')]
      },
      {
        test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          // static/img/[name].[hash:7].[ext]
          name: utils.assetsPath('img/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          // static/media/[name].[hash:7].[ext]
          name: utils.assetsPath('media/[name].[hash:7].[ext]')
        }
      },
      {
        test: /\.(woff2?|eot|ttf|otf)(\?.*)?$/,
        loader: 'url-loader',
        options: {
          limit: 10000,
          name: utils.assetsPath('fonts/[name].[hash:7].[ext]')
        }
      }
    ]
  },
  // 这些选项可以配置是否 polyfill 或 mock 某些 Node.js 全局变量和模块。这可以使最初为 Node.js 环境编写的代码，在其他环境（如浏览器）中运行。
//   true：提供 polyfill。
// "mock"：提供 mock 实现预期接口，但功能很少或没有。
// "empty"：提供空对象。
// false: 什么都不提供。预期获取此对象的代码，可能会因为获取不到此对象，触发 ReferenceError 而崩溃。尝试使用 require('modulename') 导入模块的代码，可能会触发 Cannot find module "modulename" 错误。

  node: {
    // prevent webpack from injecting useless setImmediate polyfill because Vue
    // source contains it (although only uses it if it's native).
    // 防止webpack注入无用的setImmediate polyfill，因为Vue源代码包含它（尽管只在本机使用时才使用它）。
    setImmediate: false,
    // prevent webpack from injecting mocks to Node native modules
    // that does not make sense for the client
    dgram: 'empty',
    fs: 'empty',
    net: 'empty',
    tls: 'empty',
    child_process: 'empty'
  }
}
```

# vueLoaderConfig 完整配置

``` js
const vueLoaderConfig = {
  loaders: {
    css: ['vue-style-loader',
      { loader: 'css-loader', options: { sourceMap: false } }
    ],
    postcss: ['vue-style-loader',
      { loader: 'css-loader', options: { sourceMap: false } }
    ],
    less: ['vue-style-loader',
      { loader: 'css-loader', options: { sourceMap: false } },
      { loader: 'less-loader', options: { sourceMap: false } }
    ],
    sass: ['vue-style-loader',
      { loader: 'css-loader', options: { sourceMap: false } },
      {
        loader: 'sass-loader',
        options: { indentedSyntax: true, sourceMap: false }
      }
    ],
    scss: ['vue-style-loader',
      { loader: 'css-loader', options: { sourceMap: false } },
      { loader: 'sass-loader', options: { sourceMap: false } }
    ],
    stylus: ['vue-style-loader',
      { loader: 'css-loader', options: { sourceMap: false } },
      { loader: 'stylus-loader', options: { sourceMap: false } }
    ],
    styl: ['vue-style-loader',
      { loader: 'css-loader', options: { sourceMap: false } },
      { loader: 'stylus-loader', options: { sourceMap: false } }
    ]
  },
  cssSourceMap: false,
  cacheBusting: true,
  transformToRequire: {
    video: ['src', 'poster'],
    source: 'src',
    img: 'src',
    image: 'xlink:href'
  }
}
```