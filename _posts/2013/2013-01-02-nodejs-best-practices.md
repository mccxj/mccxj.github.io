---
layout: post
comments: true
title: "Node.js Best practices"
description: "来自slideshare的ppt"
categories: ["内容推荐", "node", "实践"]
---

**完整的ppt来源于[Node.js Best practices][1],作者[Felix Geisendörfer][2],需翻越**

### Callbacks ###

下面是关于解析json文件的示例代码:
{% highlight javascript %}
var fs = require('fs);
function readJSON(path, cb){
  fs.readFile(path, 'utf8', function(err, data){
    cb(JSON.parse(data));
  }
}
{% endhighlight %}

显然，上面的代码没有处理异常情况，因此再加个异常处理逻辑上去:
{% highlight javascript %}
var fs = require('fs);
function readJSON(path, cb){
  fs.readFile(path, 'utf8', function(err, data){
    if(err) return cb(err);
    cb(JSON.parse(data));
  }
}
{% endhighlight %}

还没有结束，我们还没考虑到文件内容不是json这种情况，会导致parse出现异常，因此再处理一下：
{% highlight javascript %}
var fs = require('fs);
function readJSON(path, cb){
  fs.readFile(path, 'utf8', function(err, data){
    if(err) return cb(err);
    try{
      cb(JSON.parse(data));
    }catch(err){
      cb(err);
    }
  }
}
{% endhighlight %}

对于cb来说，仍然无法区分正常结果和异常内容，这个问题通常可以通过增加一个err参数来处理，如:
{% highlight javascript %}
var fs = require('fs);
function readJSON(path, cb){
  fs.readFile(path, 'utf8', function(err, data){
    if(err) return cb(err);
    try{
      var json = JSON.parse(data);
    }catch(err){
      return cb(err);
    }
    cb(null, json);
  }
}
{% endhighlight %}

**这个示例告诉我们，对于callback，需要时刻准备应付正常结果和异常情况的处理。**

再来看看另外一个常见错误:
{% highlight javascript %}
function readJSONFiles(files, cb){
  var results = {};
  var remaining = files.length;
  
  files.forEach(function(file){
    readJSON(file, function(err, json){
      if(err) return cb(err);
      
      results[file] = json;
      if(!--remaining) cb(null, results);
    }
  });
}
{% endhighlight %}

这里隐含了一个常见的场景:**批量处理时，任意一个失败，及时退出**。有时候可以用标识符，这里采用另外一种手法：**重置回调方法**。
{% highlight javascript %}
function readJSONFiles(files, cb){
  var results = {};
  var remaining = files.length;
  
  files.forEach(function(file){
    readJSON(file, function(err, json){
      if(err){
        cb(err);
        cb = function(){};
        return;
      }
      
      results[file] = json;
      if(!--remaining) cb(null, results);
    }
  });
}
{% endhighlight %}

### Nested Callbacks ###

先看看一个恐怖的例子:
{% highlight javascript %}
db.query('SELECT A ...', function(){
  db.query('SELECT B ...', function(){
    db.query('SELECT C ...', function(){
      db.query('SELECT D ...', function(){
      });
    });
  });
});
{% endhighlight %}

活生生就是一个怪物:)，多层嵌套的回调不是很好的风格，我们需要一些流程控制的东西来辅助，例如Control Flow Libs:
{% highlight javascript %}
var async = require('async');

async.series({
  queryA: function(next){
    db.query('SELECT A ...', next);
  },
  queryB: function(next){
    db.query('SELECT B ...', next);
  },
  queryA: function(next){
    db.query('SELECT C ...', next);
  }
  // ...
}, function(err, results){
  //...
});
{% endhighlight %}


像上面的代码，最明显的地方就是异常处理被完全隔离出来。**如果要把代码分布到很多小方法里边的话，Node.js的确不是很容易做到**。

### Exceptions ###

通常throw new Error(msg)可以让你的程序进行异常退出，并在控制台上输入错误信息和堆栈信息。

但有时候我们要考虑的是，**一些未知的bug**，例如下面一个有bug的示例:
{% highlight javascript %}
function MyClass(){}

MyClass.prototype.myMethod = function(){
  setTimeout(function(){
    this.myOtherMethod();
  }, 10);
}

MyClass.prototype.myOtherMethod = function(){};

(new MyClass).myMethod();
{% endhighlight %}

我们可以采用Global Catch的方式:
{% highlight javascript %}
process.on('uncaughtException', function(err){
  console.err('uncaught exception: ' + err.stack); 
});
{% endhighlight %}

更好的处理方式是:**进程挂了，认栽了，到更高层面上去处理**。
{% highlight javascript %}
process.on('uncaughtException', function(err){
  // You could use node-airbake for this
  sendErrorToLog(function(){
    // Once the error was logged, kill the process
    console.err(err.stack);
    process.exit(1);
  }
});
{% endhighlight %}

### Deployment ###

比较初级的方式是采用node直接运行或在后台运行。老手可能会采用一个脚本来搞:
{% highlight bash %}
#! /bin/bash

while :
do
  node server.js
  echo "Server crashed!"
  sleep 1
done
{% endhighlight %}

专家级采用的方式可能是(借助成熟的集成工具):
{% highlight bash %}
#!upstart

description "myapp"
author "felix"

start on(local-filesystems and net-device-up IFACE=eth0)
stop on shutdown

respawn # restart when job dies
respawn limit 5 60 # give up restart after 5 respawns in 60 seconds

script
  exec sudo -u www-data /path/to/server.js >> /var/log/myapp.log 2>&1
end script
{% endhighlight %}

当然，还有些创新风格的(基于托管平台的):
{% highlight bash %}
$git push joyent master
$git push nodejitsu master
$git push heroku master
{% endhighlight %}

**没有托管平台的话，借助成熟的工具应该是最好的选择，性价比高。**

 [1]: http://www.slideshare.net/the_undefined/nodejs-best-practices-10428790
 [2]: http://www.slideshare.net/the_undefined

