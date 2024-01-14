---
title: WebSocket
author: 荀乐
tags:
  - 协议
categories:
  - 计算机网络
date: 2022-03-28 09:09:01
---
# WebSocket

## 概述

**WebSocket**是一种在单个[TCP](https://baike.baidu.com/item/TCP)连接上进行[全双工](https://baike.baidu.com/item/全双工)通信的协议。**HTTP**协议有一个缺陷，即通信只能由客户端发起，如果服务器有状态变化，只能通过客户端**轮询**每隔一段时间给服务端发起请求，询问是否有新消息。而**WebSocket**协议中，服务器可以主动向客户端推送信息，客户端也可以主动向服务器发送信息。

![img](https://xunle-picture-bed.oss-cn-hangzhou.aliyuncs.com/20221206153228.png)

**WebSocket**协议由两部分组成

-   握手
-   数据传输

---

**握手**

握手是基于http协议的。

来自客户端的握手

```http
GET ws://localhost/char HTTP/1.1
HOST: localhost
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-key: key1
Sec-WebSocket-Extensions: permessage-deflate
Sec-WebSocket-Version: 13
```

来自服务器的握手

```http
HTTP/1.1 101 Switching Protocols
Upgrade: webscocket
Connection: Upgrade
Sec-WebSocket-Accept: acc1
Sec-WebSocket-Extensions: permessage-deflate
```

字段说明

| 头名称                                      | 说明                                                         |
| ------------------------------------------- | ------------------------------------------------------------ |
| Connection: Upgrade                         | 标识该HTTP请求是一个协议升级请求                             |
| Upgrade: websocket                          | 协议升级为WebScoket协议                                      |
| Sec-WbSocket-Versions: 13                   | 客户端支持WebSocket的版本                                    |
| Sec-WbSocket-key: xxx                       | 客户端采用base64编码的24位随机字符序列，服务器接受客户端HTTP协议升级的证明。要求服务端响应一个对应的加密的Sec-WebSocket-Accept头信息作为应答 |
| Sec-WbSocket-Extensions: permessage-deflate | 协议拓展类型                                                 |



---

### 客户端（浏览器）实现

**WebSocket对象**

```js
var ws = new WebSocket(url)
```

>   通过上面的语句创建一个WebSocket对象，需要传入参数`url`
>
>   `url`：ws://ip:prot/resources



**WebSocket事件**

| 事件    | 事件处理程序        | 描述                       |
| ------- | ------------------- | -------------------------- |
| open    | websocket.onopen    | 连接建立时触发             |
| message | websocket.onmessage | 客户端接收服务端数据时触发 |
| error   | websocket.onerror   | 通信发生错误时触发         |
| close   | websocket.onclose   | 连接关闭时触发             |



**WebSocket方法**

| 方法   | 描述             |
| ------ | ---------------- |
| send() | 使用连接发送数据 |



---

### 服务端实现

Tomcat的7.0.5版本开始支持**WebSocket**协议，并且实现了**Java WebSocket**规范

**Java WebSocket**应用有一系列的**WebSocketEndpoint**组成。**EndPoint**是一个Java对象，代表**WebSocket**连接的一端。

两种方式定义**EndPoint**

-   编程式：继承类``javax.websocket.EndPoint``并实现方法
-   注解式：定义一个POJO，并添加`@ServerEndpoint`相关注解

**服务端接受客户端发送的数据**

>   通过为`Session`添加`MessageHandler`消息处理器来接受消息，当采用注解方式时，通过注解`@OnMessage`注解指定接受消息的方法。

**服务端推送数据给客户端**

>   发送消息由`RemoteEndPoint`完成，其实例由`Session`维护。根据使用情况，可以通过`Session.getBasicRemote`获取发送同步消息的实例，调用`sendXxx()`方法发送消息；可以通过`Session.getAsyncRemote`获取发送异步消息的实例

