# 二进制模块


## Running shell command

http://krasimirtsonev.com/blog/article/Nodejs-managing-child-processes-starting-stopping-exec-spawn

There are two methods for running child processes under Node.js - exec and spawn. Here is an example of using the first one:

```
var exec = require('child_process').exec;
exec('node -v', function(error, stdout, stderr) {
    console.log('stdout: ' + stdout);
    console.log('stderr: ' + stderr);
    if (error !== null) {
        console.log('exec error: ' + error);
    }
});
```

We are passing our command node -v as a first argument and expect to get response in the provided callback. stdout is what we normally need. That's the output of the command. error should be null if everything is ok. Let's see what will happen if we run the following failing script:


```
// failing.js
console.log('Hello!');
throw new Error('Ops!')
console.log('End!');

// script.js
var exec = require('child_process').exec;
exec('node ./failing.js', function(error, stdout, stderr) {
    console.log('stdout: ', stdout);
    console.log('stderr: ', stderr);
    if (error !== null) {
        console.log('exec error: ', error);
    }
});

```

The result is:

```
exec error:  { [Error: Command failed: failing.js:2
throw new Error('Ops!')
      ^
Error: Ops!
    at Object.<anonymous> (failing.js:2:7)
    at Module._compile (module.js:456:26)
    at Object.Module._extensions..js (module.js:474:10)
    at Module.load (module.js:356:32)
    at Function.Module._load (module.js:312:12)
    at Function.Module.runMain (module.js:497:10)
    at startup (node.js:119:16)
    at node.js:902:3
] killed: false, code: 8, signal: null }
```

stderr contains the text of the error and error is an instance of Error class. So we may use error.code or error.signal to get more information about the problem.

 
So, the better approach here will be to attach listeners to the stdout and stderr. There is also a close event available. Let's change our script.js file to the following code, that illustrates the idea:

```
var exec = require('child_process').exec;
var child = exec('node ./commands/server.js');
child.stdout.on('data', function(data) {
    console.log('stdout: ' + data);
});
child.stderr.on('data', function(data) {
    console.log('stdout: ' + data);
});
child.on('close', function(code) {
    console.log('closing code: ' + code);
});

```

There is an alternative of exec called spawn. Unlike exec it launches a new process and should be used if we expect long-time communication with the running command. The API is a little bit different, but both methods work similar.

```
child_process.exec(command, [options], callback)
child_process.spawn(command, [args], [options])
```

spawn doesn't accept a callback and if we need to pass arguments along with the command name we have to do this in an additional [args] array.

## Killing/Stopping the command

```
var psTree = require('ps-tree');

var kill = function (pid, signal, callback) {
   signal   = signal || 'SIGKILL';
   callback = callback || function () {};
   var killTree = true;
   if(killTree) {
       psTree(pid, function (err, children) {
           [pid].concat(
               children.map(function (p) {
                   return p.PID;
               })
           ).forEach(function (tpid) {
               try { process.kill(tpid, signal) }
               catch (ex) { }
           });
           callback();
       });
   } else {
       try { process.kill(pid, signal) }
       catch (ex) { }
       callback();
   }
};

// ... somewhere in the code of Yez!
kill(child.pid);
```

And as it normally happens, few hours later I found a workaround. It's dummy, but it works. There is a Windows command line instrument called taskkill. So, what I have to do is to detect that my Node.js module is operating on a Windows machine and send the PID to that small utility.

```
var isWin = /^win/.test(process.platform);
if(!isWin) {
   kill(processing.pid);
} else {
   var cp = require('child_process');
   cp.exec('taskkill /PID ' + processing.pid + ' /T /F', function (error, stdout, stderr) {
       // console.log('stdout: ' + stdout);
       // console.log('stderr: ' + stderr);
       // if(error !== null) {
       //      console.log('exec error: ' + error);
       // }
   });             
}
```

The /T (terminates all the sub processes) and /F (forcefully terminating) arguments are also critical and without them we can't achieve the desired results.

## 使用shelljs

https://github.com/shelljs/shelljs

> Portable Unix shell commands for Node.js http://shelljs.org


```
require('shelljs/global');

if (!which('git')) {
  echo('Sorry, this script requires git');
  exit(1);
}

// Copy files to release dir
rm('-rf', 'out/Release');
cp('-R', 'stuff/', 'out/Release');

// Replace macros in each .js file
cd('lib');
ls('*.js').forEach(function(file) {
  sed('-i', 'BUILD_VERSION', 'v0.1.2', file);
  sed('-i', /^.*REMOVE_THIS_LINE.*$/, '', file);
  sed('-i', /.*REPLACE_LINE_WITH_MACRO.*\n/, cat('macro.js'), file);
});
cd('..');

// Run external tool synchronously
if (exec('git commit -am "Auto-commit"').code !== 0) {
  echo('Error: Git commit failed');
  exit(1);
}
```

其实大部分shell都支持，还是非常用的模块
