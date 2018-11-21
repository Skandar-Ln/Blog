---
title: server-side-events(SSE)开发指南（Node）
date: 2018-10-18 16:12:41
tags:
---
**SSE**是介于websocket、长短轮训之外的一种服务端推送的方式。他的好处有

- 使用简单，无需借助第三方库（如 socket.io）
- 基于HTTP协议（WebSocket 是一个独立协议），无需对其做额外处理。还能享受HTTP2带来的优势
- 默认支持断线重连
- 支持自定义发送的消息类型

[详细对比](https://codeburst.io/polling-vs-sse-vs-websocket-how-to-choose-the-right-one-1859e4e13bd9)，这里我选择尝试将一个原本基于轮询的web app转到sse上来。虽然这套技术看上去使用很简单，但可能由于普及程度不高和资料较少的原因，在开发过程中会遇到很多的坑和要面临的新东西。这里帮大家总结一下，后端使用了koa.js（express应该会更简单）。

## 后端

对于一个SSE相应我们需要返回如下一些HTTP头

```
Content-Type: text/event-stream
Cache-Control: no-cache, no-transform
Connection: keep-alive
X-Accel-Buffering: no
```

在其他的教程中提供的http头可能没有这里的全，区别主要在于：
- Cache-Control中需要包含no-transform，没有这个的话在开发中，如果你用了create-react-app等工具来转发你的请求，那么你的数据流很可能被压缩，造成你怎么也收不到响应。这里当时排查了蛮久的（[issue](https://github.com/facebook/create-react-app/issues/1633)）
- no-transform是开发环境中的遇到的问题，但是在生产环境仍然还存在问题，比如我的网站使用nginx做反向代理的，默认会对应用的响应做缓冲(buffering)，以至于我应用返回的消息没有立马发出去。所以我们需要给http头加上一条X-Accel-Buffering: no（[issue](https://serverfault.com/questions/801628/for-server-sent-events-sse-what-nginx-proxy-configuration-is-appropriate)）

当设置好header后，我们就可以写入数据了。一般来说我们只需要监听数据的更新然后使用`res.write`即可写入数据：

```javascript
const onEvent = function(data) {
    res.write(`event: message\n`);
    res.write(`data: ${JSON.stringify(data)}\n\n`);
};

emitter.on('message', onEvent);
```
我们用`\n`来分隔每一行数据，用`\n\n`来分隔每一个事件。每一个事件中包含事件的type和事件的data，分别用两行来描述。比如上面是返回来一个message事件（若不指定事件类型，则默认message）。下图中我们还返回来一个withdraw事件，对应的数据行应该是`event: withdraw`。

![events](https://user-gold-cdn.xitu.io/2018/10/18/16686190a0c5a717?w=374&h=100&f=png&s=15304)

### koa.js返回

对于koa情况比较复杂，官方不推荐我们直接操作res对象，而是给context(ctx)对象的body赋值。[官方例子](https://github.com/koajs/examples/tree/master/stream-server-side-events)

其实我们只需要给ctx.body赋一个可写流，关于node流的概念可以看[taobao的这篇文章](http://taobaofed.org/blog/2017/08/31/nodejs-stream/)。如官方示例的：

```javascript
/**
 * Create a transform stream that converts a stream
 * to valid `data: <value>\n\n' events for SSE.
 */

var Transform = require('stream').Transform;
var inherits = require('util').inherits;

module.exports = SSE;

inherits(SSE, Transform);

function SSE(options) {
  if (!(this instanceof SSE)) return new SSE(options);

  options = options || {};
  Transform.call(this, options);
}

SSE.prototype._transform = function(data, enc, cb) {
  this.push(data.toString('utf8'));
  cb();
};
```

注意官方实例中有个坑就是默认给每行数据前面加上了`data:`前缀，这里删除了。在使用`const body = ctx.body = SSE()`后就可以对body对象使用`body.write`了。详见官方实例，实例中的db.js中的可读流是可选项。


## 客户端

客户端（浏览器）的使用就非常简单了。大部分的浏览器支持SSE，而且我们有针对老浏览器的兼容方案，如[Yaffle ](https://github.com/Yaffle/EventSource)。

![](https://user-gold-cdn.xitu.io/2018/10/18/166862ae92cf5fa2?w=1590&h=774&f=png&s=96398)

使用上真的是特别的简单，而且几乎没有什么坑

```javascript
 const evtSource = new EventSource('/events');

 evtSource.addEventListener('event', function(evt) {
      const data = JSON.parse(evt.data);
      // Use data here
 }, false);
```

上面的`event`可以替换为你的其他自定义事件。注意这里的连接中断后会自动重连，也许你需要监听onerror事件来做一些额外的处理。导致中断的原因可能有时间间隔到期、网络错误等。你可以通过定时向客户端返回内容来维护连接不中断：

```javascript
// Heartbeat
const nln = function() {
    res.write('\n');
};
const hbt = setInterval(nln, 15000);

// Clear heartbeat and listener
req.on('close', function() {
    clearInterval(hbt);
    emitter.removeListener('event', onEvent);
});
```

将轮询替换为sse后还是很清爽的。注意和websocket不同sse是单向数据流，我们在发送消息的时候需要使用其它的接口，可以通过node的[events](http://nodejs.cn/api/events.html)来监听触发推送。

## 参考资料

- https://github.com/facebook/create-react-app/issues/1633
- https://codeburst.io/polling-vs-sse-vs-websocket-how-to-choose-the-right-one-1859e4e13bd9
- http://www.ruanyifeng.com/blog/2017/05/server-sent_events.html
- https://serverfault.com/questions/801628/for-server-sent-events-sse-what-nginx-proxy-configuration-is-appropriate
- http://taobaofed.org/blog/2017/08/31/nodejs-stream/