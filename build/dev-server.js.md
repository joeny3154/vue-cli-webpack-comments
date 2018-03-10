```js
require('./check-versions')()

var config = require('../config')
if (!process.env.NODE_ENV) {
  process.env.NODE_ENV = JSON.parse(config.dev.env.NODE_ENV)
}
// ? 一个可以强制打开浏览器并跳转到指定 url 的插件
var opn = require('opn')
var path = require('path')
var express = require('express')
var webpack = require('webpack')
var proxyMiddleware = require('http-proxy-middleware')
var webpackConfig = process.env.NODE_ENV === 'testing'
  ? require('./webpack.prod.conf')
  : require('./webpack.dev.conf')

// default port where dev server listens for incoming traffic
var port = process.env.PORT || config.dev.port
// automatically open browser, if not set will be false
var autoOpenBrowser = !!config.dev.autoOpenBrowser
// Define HTTP proxies to your custom API backend
// https://github.com/chimurai/http-proxy-middleware
var proxyTable = config.dev.proxyTable // 默认配置：{}

var app = express()
var compiler = webpack(webpackConfig)

var devMiddleware = require('webpack-dev-middleware')(compiler, {
  publicPath: webpackConfig.output.publicPath,
  quiet: true
})

var hotMiddleware = require('webpack-hot-middleware')(compiler, {
  log: () => {}
})
// force page reload when html-webpack-plugin template changes
// 当html-webpack-plugin模板改变时强制页面重新加载
compiler.plugin('compilation', function (compilation) {
  compilation.plugin('html-webpack-plugin-after-emit', function (data, cb) {
    hotMiddleware.publish({ action: 'reload' })
    cb()
  })
})

// proxy api requests
// 代理 api 请求
Object.keys(proxyTable).forEach(function (context) {
  var options = proxyTable[context]
  if (typeof options === 'string') {
    options = { target: options }
  }
  app.use(proxyMiddleware(options.filter || context, options))
})

// handle fallback for HTML5 history API
// 处理HTML5 history API的回退
// 使用 connect-history-api-fallback 匹配资源，如果不匹配就可以重定向到指定地址
app.use(require('connect-history-api-fallback')())

// serve webpack bundle output
// 提供测试环境下webpack包输出
app.use(devMiddleware)

// enable hot-reload and state-preserving
// compilation error display
// 启用热重载和保持状态
// 编译错误显示
app.use(hotMiddleware)

// serve pure static assets
// 提供纯静态资源
// 输出：/static
var staticPath = path.posix.join(config.dev.assetsPublicPath, config.dev.assetsSubDirectory)

/*
  常见：app.use('/static', express.static(__dirname + '/public'));
  express.static() 是一个中间件, 把所有的静态文件路径都添加前缀"/static", 
  比如：
  GET /static/javascripts/jquery.js
  GET /static/style.css
  GET /static/favicon.ico
  你可以使用“挂载”功能。 如果req.url 不包含这个前缀, 挂载过的中间件不会执行。 当function被执行的时候,这个参数不会被传递。 这个只会影响这个函数，后面的中间件里得到的 req.url里将会包含"/static"
*/
app.use(staticPath, express.static('./static'))

var uri = 'http://localhost:' + port

var _resolve
var readyPromise = new Promise(resolve => {
  _resolve = resolve
})

console.log('> Starting dev server...')
devMiddleware.waitUntilValid(() => {
  console.log('> Listening at ' + uri + '\n')
  // when env is testing, don't need open it
  if (autoOpenBrowser && process.env.NODE_ENV !== 'testing') {
    opn(uri)
  }
  _resolve()
})

var server = app.listen(port)

module.exports = {
  ready: readyPromise,
  close: () => {
    server.close()
  }
}

```