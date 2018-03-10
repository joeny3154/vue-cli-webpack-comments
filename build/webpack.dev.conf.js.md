``` js
'use strict'
const utils = require('./utils')
const webpack = require('webpack')
const config = require('../config')
const merge = require('webpack-merge')
const baseWebpackConfig = require('./webpack.base.conf')
const HtmlWebpackPlugin = require('html-webpack-plugin')
// 能够更好在终端看到webapck运行的警告和错误
const FriendlyErrorsPlugin = require('friendly-errors-webpack-plugin')
// 自动检索下一个可用端口
const portfinder = require('portfinder')

const HOST = process.env.HOST
const PORT = process.env.PORT && Number(process.env.PORT)

const devWebpackConfig = merge(baseWebpackConfig, {
  module: {
    // 自动生成了 css, postcss, less 等规则，与自己一个个手写一样，默认包括了 css 和 postcss 规则
    rules: utils.styleLoaders({ sourceMap: config.dev.cssSourceMap, usePostCSS: true })
  },
  // cheap-module-eval-source-map is faster for development
  // 设置Source Maps 
  // dev: eval-source-map, 行数能够正确映射，因为会映射到原始代码中
  // 生产：#source-map, 生成完整的 SourceMap，输出为独立文件，在 bundle 中添加了SourceMap引用注释
  devtool: config.dev.devtool,

  // these devServer options should be customized in /config/index.js
  // 这些devServer选项应该在/config/index.js中定制
  devServer: {
    // devServer.contentBase: 默认情况下，将使用当前工作目录作为提供内容的目录，devServer.publicPath 将用于确定应该从哪里提供 bundle，并且此选项优先
    // 你可以修改为其他目录,eg: contentBase: path.join(__dirname, "public");
    // contentBase: path.join(__dirname, "dist"),
    /*
      有 none, error, warning 或者 info（默认值）等选项，控制台始终都会显示 bundle 的错误和警告，此选项只影响 bundle 的错误和警告之前的消息，比如
      1. 在重新加载之前，2. 在一个错误之前，3.或者模块热替换(Hot Module Replacement)启用时
      所有这些都显示显得比较繁琐，设置clientLogLevel: "none"将阻止所有这些消息显示，warning只显示警告信息
    */
    clientLogLevel: 'warning',
    // 当使用HTML5 History API，任意的 404 响应可以提供为 index.html 页面。通过传入以下启用
    historyApiFallback: true,
    // 启用 webpack 的模块热替换特性
    hot: true,
    // 一切服务都启用gzip 压缩：
    compress: true,
    // 只用在命令行工具(CLI)， 指定使用一个 host。默认是 localhost
    host: HOST || config.dev.host,//  host: 'localhost'
    port: PORT || config.dev.port,// port: 8080
    // 自动打开浏览器，也可以使用命令行工具(CLI)：webpack-dev-server --open 或 webpack-dev-server --open 'Google Chrome'
    open: config.dev.autoOpenBrowser,
    /* 
      当出现编译器错误或警告时，在浏览器中全屏显示出来。 默认情况下禁用。 如果您只想显示编译器错误：overlay: true;
      如果你想显示警告以及错误：overlay: {warnings: true, errors: true}
    */ 
    overlay: config.dev.errorOverlay
      ? { warnings: false, errors: true }
      : false,
    /* 
      此路径下的打包文件可在浏览器中访问, 此处被设置为'/',假设服务器运行在 http://localhost:8080 并且 output.filename 被设置为 bundle.js。默认 publicPath 是 "/"，所以你的包(bundle)可以通过 `http://localhost:8080/bundle.js` 访问。
      假如设置为`publicPath: "/assets/"`, 你的包现在可以通过 http://localhost:8080/assets/bundle.js 访问
    */
    publicPath: config.dev.assetsPublicPath,
    /*
      如果你有单独的后端开发服务器 API，并且希望在 同域名 下发送 API 请求 ，那么代理某些 URL 会很有用。
    */
    proxy: config.dev.proxyTable,
    // 启用 quiet 后，除了初始启动信息之外的任何内容都不会被打印到控制台。这也意味着来自 webpack 的错误或警告在控制台不可见。
    // 使用FriendlyErrorsPlugin 是必要的
    quiet: true, // necessary for FriendlyErrorsPlugin
    /* 
      与监视文件相关的控制选项
      webpack 可以监听文件变化，当它们修改后会重新编译, Watch 模式默认关闭。
      webpack-dev-server 和 webpack-dev-middleware 里 Watch 模式默认开启。
      如果启用 Watch 模式。这意味着在初始构建之后，webpack 将继续监听任何已解析文件的更改。
      watchOptions是一组用来定制 Watch 模式的选项
    */
    watchOptions: {
      // 是否开启轮询，此处为 false，如果指定毫秒数，则在指定毫秒时间进行轮询
      poll: config.dev.poll,
    }
  },
  plugins: [
    // 配置的全局常量
    new webpack.DefinePlugin({
      'process.env': require('../config/dev.env')
    }),
    new webpack.HotModuleReplacementPlugin(),
    // 当开启 HMR 的时候使用该插件会显示模块的相对路径，建议用于开发环境。
    new webpack.NamedModulesPlugin(), // HMR shows correct file names in console on update.
    // 在输出阶段时，遇到编译错误跳过
    // 在编译出现错误时，使用 NoEmitOnErrorsPlugin 来跳过输出阶段。这样可以确保输出资源不会包含错误。对于所有资源，统计资料(stat)的 emitted 标识都是 false
    // 此插件用于取代（现已弃用）webpack 1 的 NoErrorsPlugin 插件。
    // 如果你在使用 CLI(命令行界面command-line interface)，启用此插件后，webpack 进程遇到错误代码将不会退出。
    new webpack.NoEmitOnErrorsPlugin(), 
    // https://github.com/ampedandwired/html-webpack-plugin
    /*
      为你生成一个HTML5文件，其中包括使用script标签的body中的所有webpack包
      如果你有多多个webpack入口点，他们都会在生成的HTML文件中的script标签内
      如果你有任何CSS assets 在webpack的输出中（例如，利用ExtractTextPlugin提取CSS），那么这些将被包含在HTML head中的<link>标签内。
      这对于在文件名中包含每次会随着变异会发生变化的哈希的webpack bundle尤其有用
    */
    new HtmlWebpackPlugin({
      filename: 'index.html',
      // 使用自己的提供的html模板
      template: 'index.html',
      inject: true
    }),
  ]
})
// 自动检索下一个可用端口
module.exports = new Promise((resolve, reject) => {
  // 获取当前设定的端口
  portfinder.basePort = process.env.PORT || config.dev.port
  // 如果当前设定的端口被占用自动检索下一个可用端口
  portfinder.getPort((err, port) => {
    if (err) {
      reject(err)
    } else {
      // publish the new Port, necessary for e2e tests
      // 发布新的端口, 需要 e2e 测试
      process.env.PORT = port
      // add port to devServer config
      // 设置 devServer 端口
      devWebpackConfig.devServer.port = port

      // Add FriendlyErrorsPlugin
      // 能够更好在终端看到webapck运行的警告和错误
      devWebpackConfig.plugins.push(new FriendlyErrorsPlugin({
        // 编译成功提醒
        compilationSuccessInfo: {
          messages: [`Your application is running here: http://${devWebpackConfig.devServer.host}:${port}`],
        },
        // 此处为true
        onErrors: config.dev.notifyOnErrors
        ? utils.createNotifierCallback()
        : undefined
      }))

      resolve(devWebpackConfig)
    }
  })
})
```


``` js
const reateNotifierCallback = () => {
  const notifier = require('node-notifier')

  return (severity, errors) => {
    if (severity !== 'error') return

    const error = errors[0]
    const filename = error.file && error.file.split('!').pop()

    notifier.notify({
      title: packageConfig.name,
      message: severity + ': ' + error.name,
      subtitle: filename || '',
      icon: path.join(__dirname, 'logo.png')
    })
  }
}
```