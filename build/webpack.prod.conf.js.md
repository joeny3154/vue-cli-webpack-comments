
``` js
'use strict'
const path = require('path')
const utils = require('./utils')
const webpack = require('webpack')
const config = require('../config')
const merge = require('webpack-merge')
const baseWebpackConfig = require('./webpack.base.conf')
const CopyWebpackPlugin = require('copy-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const ExtractTextPlugin = require('extract-text-webpack-plugin')
const OptimizeCSSPlugin = require('optimize-css-assets-webpack-plugin')
// 压缩JavaScript
const UglifyJsPlugin = require('uglifyjs-webpack-plugin')

// node环境变量标识
const env = process.env.NODE_ENV === 'testing'
  ? require('../config/test.env')
  : require('../config/prod.env')

const webpackConfig = merge(baseWebpackConfig, {
  module: {
    rules: utils.styleLoaders({
      // production 下生成 sourceMap
      sourceMap: config.build.productionSourceMap,
      // 是否抽离css
      extract: true,
      // 是否使用 postcss
      usePostCSS: true
    })
  },
  // 开启 Source Maps, 此项中 config.build.devtool === '#source-map'
  devtool: config.build.productionSourceMap ? config.build.devtool : false,
  output: {
    // 此项设置为项目根目录下的dist目录 config.build.assetsRoot: path.resolve(__dirname, '../dist')
    path: config.build.assetsRoot,
    // 将js打包至dist/static/js目录下
    filename: utils.assetsPath('js/[name].[chunkhash].js'),
    // 其他chunk（如异步组件、按需加载的模块） 打包至dist/static/js目录下
    chunkFilename: utils.assetsPath('js/[id].[chunkhash].js')
  },
  plugins: [
    // http://vuejs.github.io/vue-loader/en/workflow/production.html
    // 定义全局常量，使用：if(process.env !== 'production') { ... }
    new webpack.DefinePlugin({
      'process.env': env
    }),
    // js 压缩
    new UglifyJsPlugin({
      uglifyOptions: {
        // 压缩
        compress: {
          warnings: false
        }
      },
      sourceMap: config.build.productionSourceMap,
      // parallel: { Boolean | Number } Enable parallelization. Default number of concurrent runs: os.cpus().length - 1
      // 启用并行化。 并发运行的默认数量：os.cpus().length - 1。
      parallel: true
    }),
    // extract css into its own file
    // 提取 css 成独立的文件
    new ExtractTextPlugin({
      filename: utils.assetsPath('css/[name].[contenthash].css'),
      // set the following option to `true` if you want to extract CSS from
      // codesplit chunks into this main css file as well.
      // This will result in *all* of your app's CSS being loaded upfront.
      // 如果你想从代码片段中提取CSS到这个主要的CSS文件，那么把下面的选项设置为`true`。 这将导致你的应用程序的所有CSS被预先加载。
      allChunks: false,
    }),
    // Compress extracted CSS. We are using this plugin so that possible
    // duplicated CSS from different components can be deduped.
    // 压缩 提取的CSS。 我们使用这个插件，以便可以从不同组件中重复使用CSS。

    // 1. 压缩提取的CSS，体积过大。
    // 2. 解决`extract-text-webpack-plugin`处理CSS时，可能出现重复条目问题。同一个入口模块及其子模块`import`同样的**样式文件**并不会出现bundle中存在重复的模块内容，存在重复条目指的是不同的样式文件中可能存在同样的条目，比如两个css文件中都有`.clearfix { ... }`,使用`extract-text-webpack-plugin` 来bundles CSS后，bundle 就会出现重复的`.clearfix { ... }`条目
    new OptimizeCSSPlugin({
      cssProcessorOptions: config.build.productionSourceMap
        ? { safe: true, map: { inline: false } }
        : { safe: true }
    }),
    // generate dist index.html with correct asset hash for caching.
    // you can customize output by editing /index.html
    // see https://github.com/ampedandwired/html-webpack-plugin
    new HtmlWebpackPlugin({
      // 生成的文件的名称
      filename: process.env.NODE_ENV === 'testing'
        ? 'index.html'
        // path.resolve(__dirname, '../dist/index.html'),
        : config.build.index,
      // // 使用的模板的名称
      template: 'index.html',
      // 注入js css
      inject: true,
      // 配置html的压缩行为
      minify: {
        // 移除注释
        removeComments: true,
        // 去除空格和换行
        collapseWhitespace: true,
        // 移除属性中的引号
        removeAttributeQuotes: true
        // more options:
        // https://github.com/kangax/html-minifier#options-quick-reference
      },
      // necessary to consistently work with multiple chunks via CommonsChunkPlugin
      // 控制chunks的顺序，dependency 表示按照依赖关系进行排序，也可以是一个函数，自己定义排序规则
      chunksSortMode: 'dependency'
    }),
    // keep module.id stable when vender modules does not change
    // 根据模块的相对路径生成一个四位数的`hash`作为模块id(`module.id`),当vender模块不变时，可以保持module.id稳定, 类似：3Di9、Bau1
    // webpackJsonp([6], { "3Di9":(function (module, __webpack_exports__, __webpack_require__) {...}), ["Bau1"])

    // https://doc.webpack-china.org/guides/caching/
    new webpack.HashedModuleIdsPlugin(),
    // 作用域提升
    // webpack2处理过的每一个模块都会使用一个函数进行包裹
    // 这样会带来一个问题：降低浏览器中JS执行效率，这主要是闭包函数降低了JS引擎解析速度。
    // webpack3中，通过下面这个插件就能够将一些有联系的模块，
    // 放到一个闭包函数里面去，通过减少闭包函数数量从而加快JS的执行速度。
    // 解析来自：https://segmentfault.com/a/1190000012472099#articleHeader6
    new webpack.optimize.ModuleConcatenationPlugin(),
    // split vendor js into its own file
    // 将第三方js分割成独立的文件
    
    new webpack.optimize.CommonsChunkPlugin({
      name: 'vendor',
      minChunks (module) {
        // any required modules inside node_modules are extracted to vendor
        return (
          module.resource &&
          /\.js$/.test(module.resource) &&
          module.resource.indexOf(
            path.join(__dirname, '../node_modules')
          ) === 0
        )
      }
    }),
    // extract webpack runtime and module manifest to its own file in order to
    // prevent vendor hash from being updated whenever app bundle is updated
    // 将webpack运行时代码和模块清单提取到独立的文件中，防止每次更新应用程序包时更新 vendor 哈希值
    new webpack.optimize.CommonsChunkPlugin({
      name: 'manifest',
      minChunks: Infinity
    }),
    // This instance extracts shared chunks from code splitted chunks and bundles them
    // in a separate chunk, similar to the vendor chunk
    // see: https://webpack.js.org/plugins/commons-chunk-plugin/#extra-async-commons-chunk
    // 这个实例从代码 拆分的 chunk 中提取共享块，并将它们捆绑在一个单独的块中，类似于供应商块
    new webpack.optimize.CommonsChunkPlugin({
      name: 'app',
      async: 'vendor-async',
      children: true,
      minChunks: 3
    }),

    // copy custom static assets
    // 拷贝静态资源
    new CopyWebpackPlugin([
      {
        from: path.resolve(__dirname, '../static'),
        to: config.build.assetsSubDirectory,
        ignore: ['.*']
      }
    ])
  ]
})
// gzip 压缩
if (config.build.productionGzip) {
  const CompressionWebpackPlugin = require('compression-webpack-plugin')

  webpackConfig.plugins.push(
    new CompressionWebpackPlugin({
      // 目标资源的名称
      // [path]会被替换成原资源路径
      // [query]会被替换成原查询字符串
      asset: '[path].gz[query]',
      // gzip算法
      // 这个选项可以配置成zlib模块中的各个算法
      // 也可以是(buffer, cb) => cb(buffer)
      algorithm: 'gzip',
      // 处理所有匹配此正则表达式的资源
      test: new RegExp(
        '\\.(' +
        config.build.productionGzipExtensions.join('|') +
        ')$'
      ),
      // 只处理比这个值大的资源
      threshold: 10240,
      // 只有压缩率比这个值小的资源才会被处理
      minRatio: 0.8
    })
  )
}
// 可视化分析包的尺寸
if (config.build.bundleAnalyzerReport) {
  const BundleAnalyzerPlugin = require('webpack-bundle-analyzer').BundleAnalyzerPlugin
  webpackConfig.plugins.push(new BundleAnalyzerPlugin())
}

module.exports = webpackConfig

```