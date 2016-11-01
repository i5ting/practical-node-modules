## npm scripts

无意间看到
[forge](https://github.com/digitalbazaar/forge) A native implementation of TLS (and various other cryptographic tools) in JavaScript.

发现它里面的`npm run`命令

我之前只是使用`package.json`里的`scripts`里的

- start
- test

我其实还挺好奇，为啥npm支持的命令这么少

### 可重写的命令

```
Usage: npm <command>

where <command> is one of:
    add-user, adduser, apihelp, author, bin, bugs, c, cache,
    completion, config, ddp, dedupe, deprecate, docs, edit,
    explore, faq, find, find-dupes, get, help, help-search,
    home, i, info, init, install, isntall, issues, la, link,
    list, ll, ln, login, ls, outdated, owner, pack, prefix,
    prune, publish, r, rb, rebuild, remove, repo, restart, rm,
    root, run-script, s, se, search, set, show, shrinkwrap,
    star, stars, start, stop, submodule, t, tag, test, tst, un,
    uninstall, unlink, unpublish, unstar, up, update, v,
    version, view, whoami
```

上面我们说的`start`和`test`命令就是这样玩的。

### 自定义的命令

从上面的命令列表里我们可以看到，没有`index`命令，那么我们怎么自定义呢？

先定义`package.json`里的命令

```
  "scripts": {
		"index":"node index.js"
  },
```

然后我创建index.js，并在里面加一句打印

	console.log('node index.js');
	
测试

	npm run index
	
结果如下

```
➜  npm-run-test git:(master) ✗ npm run index

> npm-run-test@1.0.0 index /Users/sang/workspace/github/npm-run-test
> node index.js

node index.js
```

自定义命令的规律是

1. 不在npm可重写里的
1. 通过`npm run + cmd`即可知晓 
1. cmd可以是`shell脚本`也可以是`gulp命令`，也可以是`nodejs写的脚本`

它给力你无限自由

### 组合拳	

先定义`package.json`里的命令

```
  "scripts": {
    "test": "npm run index",
		"index":"node index.js"
  },
```

这里执行

	npm test
	
输出结果如下

```
➜  npm-run-test git:(master) ✗ npm test

> npm-run-test@1.0.0 test /Users/sang/workspace/github/npm-run-test
> npm run index


> npm-run-test@1.0.0 index /Users/sang/workspace/github/npm-run-test
> node index.js

node index.js
```

### 实例：在iOS项目使用npm scripts


首先看一下package.json

```
cat package.json 
{
  "name": "xbmhz-ios",
  "version": "1.0.0",
  "description": "iOS项目",
  "main": "index.js",
  "scripts": {
    "pod": "pod install --verbose --no-repo-update",
    "test": "echo \"Error: no test specified\" && exit 1",
    "gym": "rvm use 2.0 && gym --workspace 'hz.xcworkspace' --scheme 'hz' --use_legacy_build_api true",
    "gymc": "rvm use 2.0 && gym --workspace 'hz.xcworkspace' --scheme 'hz' --clean"
  },
  "repository": {
    "type": "git",
    "url": "git+https://github.com/mengxiaoban/xbmhz-ios.git"
  },
  "author": "",
  "license": "ISC",
  "bugs": {
    "url": "https://github.com/mengxiaoban/xbmhz-ios/issues"
  },
  "homepage": "https://github.com/mengxiaoban/xbmhz-ios#readme",
  "devDependencies": {
    "cz-conventional-changelog": "^1.1.5",
    "ghooks": "^1.0.3"
  },
  "config": {
    "commitizen": {
      "path": "./node_modules/cz-conventional-changelog"
    },
    "ghooks": {
      "post-receive": "node bin/replace.js",
      "post-update": "node bin/replace.js",
      "post-merge": "node bin/replace.js"
    }
  }
}
```

### 安装依赖

```
$ npm install
```

### npm scripts

```
 "scripts": {
    "pod": "pod install --verbose --no-repo-update",
    "test": "echo \"Error: no test specified\" && exit 1",
    "gym": "rvm use 2.0 && gym --workspace 'hz.xcworkspace' --scheme 'hz' --use_legacy_build_api true",
    "gymc": "rvm use 2.0 && gym --workspace 'hz.xcworkspace' --scheme 'hz' --clean"
  },
```

说明

- pod是cocoapod，是iOS里用到的包管理工具，类似于node里的npm
- gym是一个ruby写的便于iOS应用打包的工具

执行

```
npm run pod
npm run gym
npm run gymc
```

是不是很方便？

### hook

项目里有一处hz.db是可以配置的，为了开发方便，把它放到桌面上，便于随时db调试，但有一个问题，每个人的用户名都不一样，你无法保证录制是一直的，那怎么办呢？


解决办法很简单，就是通过ghooks，在每次更新或者merge的时候进行替换，写一个bin/replace.js脚本，在hook里执行即可

```
$ cat bin/replace.js 
 #!/usr/bin/env node

var fs = require('fs')

function home (){
  return process.env.HOME || process.env.HOMEPATH || process.env.USERPROFILE;
}

var source = "./hz/hz/Xbm.pch";
var data= fs.readFileSync(source ,"utf-8");  

var s = "@\"" + home() + "/Desktop/hz.db\""
console.log(s)

var new_str = data.replace(/@\"\/Users\/(.*?)\/Desktop\/hz\.db\"/g, s);

fs.writeFileSync(source, new_str, 'utf8');
```

### 总结

npm命令分2种

- npm自己的命令，可重写
- 自定义的命令，需要使用`npm run`执行

2种可以组合，其他的就请大家自己发挥吧

## npm hook


- prepublish: Run BEFORE the package is published. (Also run on local npm install without any arguments.)
- publish, postpublish: Run AFTER the package is published.
- preinstall: Run BEFORE the package is installed
- install, postinstall: Run AFTER the package is installed.
- preuninstall, uninstall: Run BEFORE the package is uninstalled.
- postuninstall: Run AFTER the package is uninstalled.
- preversion, version: Run BEFORE bump the package version.
- postversion: Run AFTER bump the package version.
- pretest, test, posttest: Run by the npm test command.
- prestop, stop, poststop: Run by the npm stop command.
- prestart, start, poststart: Run by the npm start command.
- prerestart, restart, postrestart: Run by the npm restart command. Note: npm restart will run the stop and start scripts if no restart script is provided.

Additionally, arbitrary scripts can be executed by running npm run-script <pkg> <stage>. Pre and post commands with matching names will be run for those as well (e.g. premyscript, myscript, postmyscript).

