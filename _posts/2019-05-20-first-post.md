---
layout: post
title: first-post
---

因为工作中经常需要生产新的项目模板，有天想到vue-cli可以使用命令行初始化一个项目模板，我就想是不是也能自己实现一个命令行工具来为
工作提供便利，就尝试写了一个，感觉还算好用，这里记录一下如何用node和npm生产命令行工具。

<!-- more -->

## 起步

在package.json里需要设置一个 'bin'的属性来设置该命令行工具的命令和执行该命令所要运行的文件
如：

```javascript
// package.json
"bin": {
  "testcli": "./index.js"
}
```

然后执行 

```javascript
npm link // 将一个任意位置的npm包链接到全局执行环境，从而在任意位置使用命令行都可以直接运行该npm包
npm unlink // 取消link
```

index.js开头需要添加一句代码代表该文件要以什么执行程序去执行他
如：

```javascript
#! /usr/bin/env node 
console.log(process.argv);
```


在 index.js中可以通过 process.argv 获取执行命令时传入的参数

```javascript
// 执行
// testcli --test
console.log(process.argv);
// 输出
// [ 'C:\\Program Files\\nodejs\\node.exe',
//   'C:\\Users\\Administrator\\AppData\\Roaming\\npm\\node_modules\\testcli\\app.js',
//   '--test' ]
```

然后就可以根据传入的不同的参数而执行不同的操作了

## 使用commander包来开发命令行工具

虽然通过以上方法能实现命令行工具的开发，然而通过 `process.argv`获取参数还是比较麻烦的，我们可以通过使用tj大神的commander包来简化命令行开发

简单的例子：

```javascript
const program = require('commander')
program
  .command('init [param] [param2]') // 匹配当用户执行 testcli init命令 (后面的参数可省略)
  .option('-t, --type [type1]', '类型') // 当option配合command使用时需要定义在command后面，才能在action的回调里通过cmd.someOption获取到
  .alias('i') // command的别名，这里相当于 init
  .description('初始化') // 对该命令的描述
  .action((param1, param2, cmd /* cmd(最后一个参数是一个Command对象) */) => {
    // testcli init param param2 --type newType
    // 一个action对应一个 command命令
    console.log(param1, param2)
    let type = cmd.type || 'default'
  })
 /*  
 // 如果有多个 command 命令需要写多个command个action的组合
 program
  .command('init [param] [param2]') 
  .option('-t, --type [type1]', '类型') 
  .alias('i') 
  .description('初始化') 
  .action((param1, param2, cmd) => {
    一个action对应一个 command命令
    console.log(param1, param2)
    let type = cmd.type || 'default'
  }) */
program.parse(process.argv) //  解析命令行，一般在最后执行
// 当option单独使用时则需要再次执行parse方法
program
  .option('-l, --log [str]', '打印')
  .parse(process.argv)
// 通过if判断来执行操作，判断需要在执行parse方法之后
if (program.log) {
  console.log(program.log)
  // 如果没有传参数（[str]）则 program.log 的值为 true
  // 如果有传参数（[str]）则 program.log 的值为 传入的参数
}

```

>使用 testcli -h 或者 --help 可以查看帮助信息，command会自动把command和action的信息写入help中

### command
>用来定义命令，后面可以传入参数; 如  init [type1] [type2] (如果为 < type1 > 包围则表示改参数不可省略)

### option
>当option配合command使用时需要定义在command后面，才能在action的回调里通过cmd.someOption获取到
>当option单独使用时则需要再次执行parse方法
>它接受四个参数，
>在第一个参数中，它可输入短名字 -a和长名字–app ,使用 | 或者,分隔，在命令行里使用时，这两个是等价的，区别是后者可以在程序里通过回调获取到；
>第二个为描述, 会在 help 信息里展示出来；
>第三个参数为回调函数，他接收的参数为是调用时传入的参数 如 [type]，可以根据需要把传入的参数处理后再输出到action的回调里
>第四个参数为默认值，当调用时没有传参，就会使用默认值

### alias
>command的别名，可以取一个相对较短的名字

### description
>对command的描述

### action
>执行command命令后的回调

### parse 
>解析命令行

## 通过inquirer使用跟用户交互,这里写几个常用的用法
在使用命令行时，某些情况下需要根据用的的不同选择来执行不同的操作，这时候inquirer包就起到了作用；
他提供了多种的交互行为可以与使用者进行交互，如input、comfirm、password、checkbox等
具体的api可以查阅[文档](https://www.npmjs.com/package/inquirer)

这里就写一个简单的示例：

```javascript
inquirer.prompt([
        {
          type: 'confirm',
          name: 'confirmPath',
          message: `该目录已存在，是否在此目录下载？`,
        },
        {
          type: 'input',
          name: 'confirmurl',
          message: `url is exit?`,
        },
        {
          type: 'checkbox',
          name: 'checkbox',
          choices: ['1','2','3'],
        },
        {
          type: 'Password',
          name: 'Password',
          message: 'password',
        },
        {
          type: 'list',
          name: 'cssPretreatment',
          message: '想用什么css预处理器呢',
          choices: [
              {
                name: 'Sass/Compass',
                value: 'sass'
              },
              {
                name: 'Less',
                value: 'less'
              }
            ]
          }
        ]).then((answers) => {
          console.log(answers)
        })

```

至此，就已经可以开发一个命令行工具了，可以根据不同的需求再加入不同的模块包，如chalk，他可以使命令行输出有颜色的文字。

接下来还可以把该命令行发布到npm上，以后还需要用  只要通过npm install -g 安装下就能使用了

本篇文章到此结束~
