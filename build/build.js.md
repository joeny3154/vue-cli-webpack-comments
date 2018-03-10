编译入口

``` js
'use strict'
require('./check-versions')()
// 设置当前环境为生产环境
process.env.NODE_ENV = 'production'
// loading...进度条
const ora = require('ora')
// 删除文件 'rm -rf'
const rm = require('rimraf')
const path = require('path')
// stdout颜色设置, stdout意思是标准输出
const chalk = require('chalk')
const webpack = require('webpack')
const config = require('../config')
const webpackConfig = require('./webpack.prod.conf')

const spinner = ora('building for production...') // 正在编译...
spinner.start()
// 清空文件夹
rm(path.join(config.build.assetsRoot, config.build.assetsSubDirectory), err => {
  if (err) throw err
  webpack(webpackConfig, (err, stats) => {
    // 编译错误不在 err 对象内，而是需要使用 stats.hasErrors() 单独处理， err 对象只会包含 webpack 相关的问题，比如配置错误等
    // 完备的错误处理中需要考虑以下三种类型的错误： 1. 致命的 wepback 错误（配置出错等）2. 编译错误（缺失的 module，语法错误等）3. 编译警告
    // 查看完备的webpack 错误处理可查看：https://doc.webpack-china.org/api/node#stats-tojson-options-
    spinner.stop()
    if (err) throw err
    // 编译完成，输出编译文件
    // stats 能让你准确地控制显示哪些包的信息
    // stats.toString：以格式化的字符串形式返回描述编译信息（类似 CLI 的输出）
    // stats.toString 所有配置选项可查看这里：https://doc.webpack-china.org/configuration/stats
    process.stdout.write(stats.toString({
      colors: true, // 在控制台展示颜色
      modules: false, // 增加内置的模块信息
      children: false, // 增加子级的信息
      chunks: false, // 使构建过程更静默无输出
      chunkModules: false // 将内置模块信息增加到包信息
    }) + '\n\n')
    // 出现编译错误：编译错误不在 err 对象内，而是需要使用 stats.hasErrors() 单独处理， err 对象只会包含 webpack 相关的问题，比如配置错误等
    if (stats.hasErrors()) {
      console.log(chalk.red('  Build failed with errors.\n'))
      // process.exit()方法以结束状态码code指示Node.js同步终止进程。
      // 'failure'状态码: 1
      process.exit(1)
    }
    // 完成
    console.log(chalk.cyan('  Build complete.\n'))
    console.log(chalk.yellow(
      '  Tip: built files are meant to be served over an HTTP server.\n' +
      '  Opening index.html over file:// won\'t work.\n'
    ))
  })
})

```