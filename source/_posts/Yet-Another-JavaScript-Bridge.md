---
title: Yet Another JavaScript Bridge
date: 2017-04-06 23:47:44
tags:
---

# 什么是JSBridge

在Hybrid App中，关键的部分是Native和Web的交互。JSBridge就是用来建立通信机制的。通过JSBridge，Native可以调用JS的本地方法，Web调用会被Native捕获的方法，最终实现通信。

下面介绍我们封装的一个JSBridge。来看一下它的API及其用法

----

# API及其用法

## 一、YAJB初始化

初始化一个YAJB实例

**用法**

```
npm install yajb-js
```

```
var YAJB = require('yajb-js');
var yajb = new YAJB()
```

**API**

初始化实际上做了三件事情。首先是初始化一个事件队列，然后判断是在Android还是iOS平台，最后保存WebView初始化传入的data。

```
var YAJB = function(){
    // init evnetQueue
	this.eventQueue = []
	this.counter = 1
	......
	// get global options
	if (window.javaInterface) {
        options = JSON.parse(window.javaInterface.toString())
        this.isAndroid = true
    }else if (window.YAJB_INJECT){
        options = window.YAJB_INJECT
        this.isiOS = true
    }
    ......
	// store data
	this.platform = options.platform
	this.data = options.data
	// store instance
	window.YAJB_INSTANCE = this 
}
```

## 二、本地调用的方法

### YAJB.send()

**用法**

yajb.send()中可以传入的参数有事件名和data，另外还可以定义一个函数。该函数将会在Native触发该事件之后被调用，可接收Native返回的值。

```
yajb.send("${eventName}", data).then(function(val){
	console.log(val)	
}) 
```

**API**

yajb.send()做的事情是，通过`YAJB._send`将事件名、事件id和data发送给Native，并将`event + “Resolved”`的事件加入事件队列

（YAJB默认当Native触发该事件后返回的事件的事件名为`event + "Resolved"`）

这里利用了Promise，主要是为了用户可以通过Promise resolve function自定义执行成功的callback。当Native触发`event + “Resolved”`的事件时，就会传入参数并执行callback，`Promise resolve()`在此时也被调用。

```
YAJB.prototype.send = function(event, data) {
	var that = this;
	return new Promise(function(resolve, reject){
		that.eventQueue.push({
			event: event + "Resolved",
			id : that.counter,
			callback: function(value){
				console.log("resolve")
				resolve(value)
			}
		})
		that._send({event:event,id:that.counter,data:JSON.stringify(data)});
		that.counter++
	})
}
```

### YAJB._send()

上面提到了`YAJB._send`的作用是是将事件名，事件id和data发送给Native。那它具体是怎样实现的呢？

**API**

```
YAJB.prototype._send = function(option) {
	if (this.isAndroid) {
		window.location = "hybrid://" + option.event + ':' + option.id + '/'+ option.data
	}else if (this.isiOS) {
		// window.postMessage
	}
}
```

可以看到，YAJB用的是URL scheme。我们约定的URL协议是`hybrid://${eventName}:${eventid}/${data}`。当我们更改URL时，Native将会捕获该动作，并解析URL。如果该URL符合约定的协议，则不进行跳转。以此实现向Native发送消息。


### YAJB.register()

**用法**

本地先注册一个事件，等待Native触发。

YAJB.register()需要传入事件名和Native触发该事件之后需要执行的回调函数。该回调函数第一个参数为Native返回的data，第二个参数为返回event Resolved事件给Native的函数

```
yajb.register("emit", function(data,fn){
    console.log(data)
    var result = data + "haha"
    fn(result)
})
```

**API**

这里register做的事情很简单，只是把事件推到事件队列中。

```
YAJB.prototype.register = function(event,fn){
	this.eventQueue.push({
		event: event,
		callback: fn
	})
}
```

接下来会具体介绍，Native触发已注册事件时所调用的`_trigger`方法

## 三、提供给Native调用的方法

### YAJB._emit()

**用法**

上文介绍`YAJB.send()`的时候说到，Web通过send方法向Native发送该事件及其data，Native接收并解析了之后，返回事件名为`${event}Resolved`的事件。

Native通过调用本地的_emit()，返回`${event}Resolved`事件。

**API**

返回的option有`{${event}Resolved,eventid,data}`

```
YAJB.prototype._emit = function(option){
	var opt = JSON.parse(option)
    this.checkQueue(opt)
}
```

接下来到事件队列中通过id找到`${event}Resolve`事件，执行回调函数

```
YAJB.prototype.checkQueue = function(option){
	var event = this.eventQueue.find(function(item){
		return item.id == option.id
	})
	event.callback(option.data)
}
```

### YAJB._trigger()

**用法**

当Native调用_trigger的同时，传入事件名，事件id和data。根据事件名从事件队列中找出之前注册过的事件，传入Native返回的data并执行callback。如果在callback中还传入了第二个参数(如：yajb.register用法示例的`fn`)，则会给Native发送`${event}Resolved`的响应事件。

**API**

```
YAJB.prototype._trigger = function(option){
	var opt = JSON.parse(option)
	var event = this.eventQueue.find(function(item){
		return item.event === opt.event
	})
	var that = this
	event.id = opt.id
	event.callback(opt.data,function(result){
		var op = {event: opt.event + 'Resolved',data:result,id:opt.id};
		that._send(op);
	})
}
```




