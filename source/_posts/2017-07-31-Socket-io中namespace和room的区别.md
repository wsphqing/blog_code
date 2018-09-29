---
title: Socket.io中namespace和room的区别
date: 2017-07-31 16:58:22
categories: 计算机
tags: [NodeJs]
---

## socket，room，namespace三者关系
`socket.io`支持命名空间和房间，默认命名空间为`/`，没有默认的房间
注意：命名空间是`Socket.IO`协议的实现细节，与底层传输的实际URL无关，默认为/socket.io/...

`socket`一定是属于某个`namespace`，`room`一定属于`namespace`，`socket`可以在某个房间或者不在任何房间

## 命名空间(namespace)

服务器端可以使用`of`来创建命名空间:
```js
var hua = io.of('/huaqing');
hua.on('connection',function(socket){
  console.log('someone connected ~');
});
hua.emit('hi','everyone~');
```

客户端可以使用:
```js
var socket = io('/huaqing');
```
链接到这个命名空间

通常使用默认的命名空间，所以可以使用省去命名空间名字:
```js
var socket = io('/');
var socket = io();
```
<!-- more -->

## 房间(room)

在每个命名空间中，还可以定义频道，可以加入和离开的任意频道

创建和加入房间是同一个函数`join`:
```js
io.on('connection', function(socket){
  socket.join('some room');
});
```
离开房间:
```js
socket.leave('room');
```

给房间中的所有客户端发送消息:
```js
io.to('room').emit(message);
io.socket.in('room').emit(message);
```

广播消息:
```js
io.on('connection', function(socket){
  socket.broadcast.emit('user connected');
});
```

在`Socket.IO`，每个`socket` 是通过一个随机、不可猜测的标识 `Socket# id` 来识别的。为了方便，每个`socket`通过这个`id`标识来自动加入房间。
这样可以方便地将消息广播到其他`socket`:
```js
io.on('connection', function(scoket){
  socket.on('say to someone', function(id, msg){
    socket.broadcast.to(id).emit('my message', msg);
  });
});
```
