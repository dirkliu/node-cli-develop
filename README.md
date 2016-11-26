# Node.js 命令行程序开发教程
一种编程语言是否易用，很大程度上，取决于开发命令行程序的能力。
Node.js 作为目前最热门的开发工具之一，怎样使用它开发命令行程序，是 Web 开发者应该掌握的技能。
下面我们从开发一个简单的hello命令开始：
## 1.可执行脚本
首先，使用 JavaScript 语言，写一个可执行脚本 hello 。
```
#!/usr/bin/env node
console.log('hello world');
```
然后，修改 hello 的权限。
```
$ chmod 755 hello
```
现在，hello 就可以执行了。
```
$ ./hello
hello world
```
如果想把 hello 前面的路径去除，可以将 hello 的路径加入环境变量 PATH。但是，另一种更好的做法，是在当前目录下新建 package.json ，写入下面的内容。
```

{
  "name": "hello",
  "bin": {
    "hello": "hello"
  }
}
```
然后执行 npm link 命令。
```
$ npm link
```
现在再执行 hello ，就不用输入路径了。
```
$ hello
hello world
```
## 2.命令行参数的原始写法
命令行参数可以用系统变量 process.argv 获取。
还是以hello脚本为例：
```
#!/usr/bin/env node
console.log('hello ', process.argv[2]);
```
执行时，直接在脚本文件后面，加上参数即可。
```
$ ./hello tom
hello tom
```
上面代码中，实际上执行的是 node ./hello tom ，对应的 process.argv 是 ['node', '/path/to/hello', 'tom'] 。
## 3.新建进程
脚本可以通过 child_process 模块新建子进程，从而执行 Unix 系统命令。
```
#!/usr/bin/env node
var name = process.argv[2];
var exec = require('child_process').exec;

var child = exec('echo hello ' + name, function(err, stdout, stderr) {
  if (err) throw err;
  console.log(stdout);
});
```
用法如下。
```
$ ./hello tom
hello tom
```
## 4.shelljs 模块
shelljs 模块重新包装了 child_process，调用系统命令更加方便。它需要安装后使用。
```
npm install --save shelljs
```
然后，改写脚本。
```
#!/usr/bin/env node
var name = process.argv[2];
var shell = require("shelljs");

shell.exec("echo hello " + name);
```
上面代码是 shelljs 的本地模式，即通过 exec 方法执行 shell 命令。此外还有全局模式，允许直接在脚本中写 shell 命令。
```
require('shelljs/global');

if (!which('git')) {
  echo('Sorry, this script requires git');
  exit(1);
}

mkdir('-p', 'out/Release');
cp('-R', 'stuff/*', 'out/Release');

cd('lib');
ls('*.js').forEach(function(file) {
  sed('-i', 'BUILD_VERSION', 'v0.1.2', file);
  sed('-i', /.*REMOVE_THIS_LINE.*\n/, '', file);
  sed('-i', /.*REPLACE_LINE_WITH_MACRO.*\n/, cat('macro.js'), file);
});
cd('..');

if (exec('git commit -am "Auto-commit"').code !== 0) {
  echo('Error: Git commit failed');
  exit(1);
}
```
## 5.yargs 模块
shelljs 只解决了如何调用 shell 命令，而 yargs 模块能够解决如何处理命令行参数。它也需要安装。
```
$ npm install --save yargs
```
yargs 模块提供 argv 对象，用来读取命令行参数。请看改写后的 hello 。
```
#!/usr/bin/env node
var argv = require('yargs').argv;
console.log('hello ', argv.name);
```
使用时，下面两种用法都可以。
```
$ hello --name=tom
hello tom

$ hello --name tom
hello tom
```
也就是说，process.argv 的原始返回值如下。
```
$ node hello --name=tom
[ 'node',
  '/path/to/myscript.js',
  '--name=tom' ]
```
yargs 可以上面的结果改为一个对象，每个参数项就是一个键值对。
```
var argv = require('yargs').argv;

// $ node hello --name=tom
// argv = {
//   name: tom
// };
```
如果将 argv.name 改成 argv.n，就可以使用一个字母的短参数形式了。
```
$ hello -n tom
hello tom
```
可以使用 alias 方法，指定 name 是 n 的别名。
```
#!/usr/bin/env node
var argv = require('yargs')
  .alias('n', 'name')
  .argv;

console.log('hello ', argv.n);
```
这样一来，短参数和长参数就都可以使用了。
```
$ hello -n tom
hello tom
$ hello --name tom
hello tom
```
argv 对象有一个下划线（_）属性，可以获取非连词线开头的参数。
```
#!/usr/bin/env node
var argv = require('yargs').argv;

console.log('hello ', argv.n);
console.log(argv._);
```
用法如下。
```
$ hello A -n tom B C
hello  tom
[ 'A', 'B', 'C' ]
```
## 6.命令行参数的配置
yargs 模块还提供3个方法，用来配置命令行参数。
 * demand：是否必选
 * default：默认值
 * describe：提示
```
#!/usr/bin/env node
var argv = require('yargs')
  .demand(['n'])
  .default({n: 'tom'})
  .describe({n: 'your name'})
  .argv;

console.log('hello ', argv.n);
```
上面代码指定 n 参数不可省略，默认值为 tom，并给出一行提示。
options 方法允许将所有这些配置写进一个对象。
```
#!/usr/bin/env node
var argv = require('yargs')
  .option('n', {
    alias : 'name',
    demand: true,
    default: 'tom',
    describe: 'your name',
    type: 'string'
  })
  .argv;

console.log('hello ', argv.n);
```
有时，某些参数不需要值，只起到一个开关作用，这时可以用 boolean 方法指定这些参数返回布尔值。
```
#!/usr/bin/env node
var argv = require('yargs')
  .boolean(['n'])
  .argv;

console.log('hello ', argv.n);
```
上面代码中，参数 n 总是返回一个布尔值，用法如下。
```
$ hello
hello  false
$ hello -n
hello  true
$ hello -n tom
hello  true
```
boolean 方法也可以作为属性，写入 option 对象。
```
#!/usr/bin/env node
var argv = require('yargs')
  .option('n', {
    boolean: true
  })
  .argv;

console.log('hello ', argv.n);
```
## 7.帮助信息
yargs 模块提供以下方法，生成帮助信息。
 * usage：用法格式
 * example：提供例子
 * help：显示帮助信息
 * epilog：出现在帮助信息的结尾
```
#!/usr/bin/env node
var argv = require('yargs')
  .option('f', {
    alias : 'name',
    demand: true,
    default: 'tom',
    describe: 'your name',
    type: 'string'
  })
  .usage('Usage: hello [options]')
  .example('hello -n tom', 'say hello to Tom')
  .help('h')
  .alias('h', 'help')
  .epilog('copyright 2015')
  .argv;

console.log('hello ', argv.n);
```
执行结果如下。
```
$ hello -h

Usage: hello [options]

Options:
  -f, --name  your name [string] [required] [default: "tom"]
  -h, --help  Show help [boolean]

Examples:
  hello -n tom  say hello to Tom

copyright 2015
```
## 8.子命令
yargs 模块还允许通过 command 方法，设置 Git 风格的子命令。
```
#!/usr/bin/env node
var argv = require('yargs')
  .command("morning", "good morning", function (yargs) {
    console.log("Good Morning");
  })
  .command("evening", "good evening", function (yargs) {
    console.log("Good Evening");
  })
  .argv;

console.log('hello ', argv.n);
```
用法如下。
```
$ hello morning -n tom
Good Morning
hello tom
```
可以将这个功能与 shellojs 模块结合起来。
```
#!/usr/bin/env node
require('shelljs/global');
var argv = require('yargs')
  .command("morning", "good morning", function (yargs) {
    echo("Good Morning");
  })
  .command("evening", "good evening", function (yargs) {
    echo("Good Evening");
  })
  .argv;

console.log('hello ', argv.n);
```
每个子命令往往有自己的参数，这时就需要在回调函数中单独指定。回调函数中，要先用 reset 方法重置 yargs 对象。
```
#!/usr/bin/env node
require('shelljs/global');
var argv = require('yargs')
  .command("morning", "good morning", function (yargs) {
    echo("Good Morning");
    var argv = yargs.reset()
      .option("m", {
        alias: "message",
        description: "provide any sentence"
      })
      .help("h")
      .alias("h", "help")
      .argv;

    echo(argv.m);
  })
  .argv;
```
用法如下。
```
$ hello morning -m "Are you hungry?"
Good Morning
Are you hungry?
```
## 9.其他事项
###（1）返回值
```
根据 Unix 传统，程序执行成功返回 0，否则返回 1 。
```
### （2）重定向
Unix 允许程序之间使用管道重定向数据。
```
$ ps aux | grep 'node'
```
脚本可以通过监听标准输入的data 事件，获取重定向的数据。
```
process.stdin.resume();
process.stdin.setEncoding('utf8');
process.stdin.on('data', function(data) {
  process.stdout.write(data);
});
```
下面是用法。
```
$ echo 'foo' | ./hello
hello foo
```
### （3）系统信号
操作系统可以向执行中的进程发送信号，process 对象能够监听信号事件。
```
process.on('SIGINT', function () {
  console.log('Got a SIGINT');
  process.exit(0);
});
```
发送信号的方法如下。
```
$ kill -s SIGINT [process_id]
```
