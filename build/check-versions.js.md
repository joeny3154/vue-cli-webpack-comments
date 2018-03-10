
``` js
'use strict'
// 在终端为不同字体显示不同的风格
const chalk = require('chalk')
// 解析npm包的version
const semver = require('semver')
// 引入package.json文件
const packageConfig = require('../package.json')
// node版本的uninx shell命令
const shell = require('shelljs')

// 执行命令的函数
function exec (cmd) {
  return require('child_process').execSync(cmd).toString().trim()
}

const versionRequirements = [
  {
    name: 'node',
    // node的版本
    // process.version就是node的版本
    // semver.clean('v8.8.0') => 8.8.0
    currentVersion: semver.clean(process.version),
    // package.json中定义的node版本的范围 
    versionRequirement: packageConfig.engines.node
  }
]

// 会在环境变量$PATH设置的目录里查找符合条件的文件
// 相当于 which npm 返回 npm /usr/local/bin/npm
if (shell.which('npm')) {
  // 如果npm命令存在的话
  versionRequirements.push({
    name: 'npm',
    // 检查npm的版本 => 5.4.2
    currentVersion: exec('npm --version'),
    // package.json中定义的npm版本
    versionRequirement: packageConfig.engines.npm
  })
}

module.exports = function () {
  const warnings = []

  for (let i = 0; i < versionRequirements.length; i++) {
    const mod = versionRequirements[i]

    // semver.satisfies()进行版本之间的比较
    if (!semver.satisfies(mod.currentVersion, mod.versionRequirement)) {
      // 如果现有的npm或者node的版本比定义的版本低，则生成一段警告
      warnings.push(mod.name + ': ' +
        chalk.red(mod.currentVersion) + ' should be ' +
        chalk.green(mod.versionRequirement)
      )
    }
  }

  if (warnings.length) {
    console.log('')
    console.log(chalk.yellow('To use this template, you must update following to modules:'))
    console.log()

    for (let i = 0; i < warnings.length; i++) {
      const warning = warnings[i]
      console.log('  ' + warning)
    }

    console.log()
    // 退出程序
    process.exit(1)
  }
}
```

# semver 解析npm包的version

``` js
const semver = require('semver')
// 验证
semver.valid('1.2.3') // '1.2.3'
semver.valid('a.b.c') // null
// 清洁
semver.clean('  =v1.2.3   ') // '1.2.3'
semver.clean('v8.8.0') // '8.8.0'
// 满足
semver.satisfies('1.2.3', '1.x || >=2.5.0 || 5.0.0 - 7.2.3') // true
// 大于
semver.gt('1.2.3', '9.8.7') // false
// 小于
semver.lt('1.2.3', '9.8.7') // true

```