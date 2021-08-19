# Websocket

https://juejin.cn/post/6844903544978407431#heading-13

>  WebSocket最大的意义的就是让浏览器可以实时的双向通信，因为我们知道的所有HTTP协议（1.0，1.1，2.0乃至是3）都只能通过请求响应式应答，无法脱离请求主动发出消息
>
> 主要介绍：
>
> + WebSocket如何建立连接
> + 如何交换数据
> + 数据帧的格式
> + 针对WebSocket的安全攻击
> + WebScket如何抵御攻击

### 什么是WebSocket

> HTML5提供的一种浏览器与服务器进行全双工通信的网络技术，应用层协议（同HTTP），与HTTP3之前的协议一样，它基于TCP并复用了HTTP的握手通道

#### 优点

> + 支持双向通信
>
> + 更好的二进制支持
>
> + 较少的控制开销
>
>   创建连接时，ws客户端和服务端数据交换时协议控制的数据包头部较小。不包含头部时，服务端到客户端的包头只有2-10字节（取决于数据包长度），客户端到服务端，需要加上额外的4字节掩码，而HTTP协议每次通信都需要携带完整的头部
>
> + 支持扩展
>
>   ws协议定义了扩展，用户可以扩展协议，或者实现自定义的子协议

#### 学习什么

+ 如何建立连接
+ 如何交换数据
+ 数据帧格式
+ 如何维持连接

### 入门例子

#### 服务端

> 它监听了8080端口，当有新的连接请求到达时，打印日志，并向客户端发送消息
>
> 当收到来自客户端消息时，同样打印日志

```javascript
var app = require('express')();
var server = require('http').Server(app);
var WebSocket = require('ws');

var wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', function connection(ws) {
    console.log('server: receive connection.');
    
    ws.on('message', function incoming(message) {
        console.log('server: received: %s', message);
    });

    ws.send('world');
});

app.get('/', function (req, res) {
  res.sendfile(__dirname + '/index.html');
});

app.listen(3000);
```

#### 客户端

> 向8080端口发起WebSocket连接，连接建立后，打印日志，同时发送消息给服务端
>
> 接收到来自服务端的消息后，同样打印日志

```html
<script>
  var ws = new WebSocket('ws://localhost:8080');
  ws.onopen = function () {
    console.log('ws onopen');
    ws.send('from client: hello');
  };
  ws.onmessage = function (e) {
    console.log('ws onmessage');
    console.log('from server: ' + e.data);
  };
</script>
```

### 如何建立连接

> WebSocket复用了HTTP的握手通道，客户端通过HTTP请求与WebSocket服务端协商升级协议
>
> 协议升级完成后，后续的数据交换都通过WebSocket协议

#### 客户端申请协议升级

> 客户端发起协议升级请求，采用标准的HTTP报文格式，只支持GET方法
>
> 下面只是一些重点的请求头部，因为是标准的HTTP请求，类似Host、Origin、Cookie等请求头都会发送

```http
GET / HTTP/1.1
Host: localhost:8080
Origin: http://127.0.0.1:3000
// 表示要升级协议
Connection: Upgrade
// 要升级到websocket协议
Upgrade: websocket
// websocket版本，如果服务端不支持，应返回Sec-WebSocket-Version的header，包含服务端支持的版本号
Sec-WebSocket-Version: 13
// 与后面服务端响应首部的Sec-WebSocket-Accept配套，提供基本的防护，防止恶意的连接或无意的连接
Sec-WebSocket-Key: w4v7O6xFTi36lq3RNcgctw==
```

#### 服务端响应协议升级

> 状态101表示1协议切换，到此完成协议升级，后续的数据交互按照新的协议来

```http
HTTP/1.1 101 Switching Protocols
Connection:Upgrade
Upgrade: websocket
Sec-WebSocket-Accept: Oy4NRAQ13jhfONC7bP8dTKb4PTU=
```

#### Sec-WebSocket_Accept的计算

> Sec-WebSocket-Accept根据客户端请求首部的Sec-WebSocket-Key计算出来

计算公式：

+ 将Sec-WebSocket-Key与258EAFA5-E914-47DA-95CA-C5AB0DC85B11拼接
+ 通过SHA1计算出摘要，并转成base64字符串

```javascript
const crypto = require('crypto');
const magic = '258EAFA5-E914-47DA-95CA-C5AB0DC85B11';
const secWebSocketKey = 'w4v7O6xFTi36lq3RNcgctw==';

let secWebSocketAccept = crypto.createHash('sha1')
	.update(secWebSocketKey + magic)
	.digest('base64');

console.log(secWebSocketAccept);
// Oy4NRAQ13jhfONC7bP8dTKb4PTU=
```

### 数据帧格式

> 客户端、服务端数据的交换，离不开数据帧格式的定义，因此我们需要了解一些WebSocket的数据帧格式
>
> WebSocket客户端、服务端通信的最小单位是帧frame，由1个或多个帧组成一条完整message

+ 发送端：将消息切割成多个帧，并发送给服务端
+ 接受端：接受消息帧，并将关联的帧重新组装成完整的message

#### 格式概览

> 从左到右，单位是bit，比如FIN、RSV1/2/3各占据1bit，opcode占据4bit
>
> 内容包括标识位、操作代码opcode、掩码mask、数据、数据长度

```
  0                   1                   2                   3
  0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
 +-+-+-+-+-------+-+-------------+-------------------------------+
 |F|R|R|R| opcode|M| Payload len |    Extended payload length    |
 |I|S|S|S|  (4)  |A|     (7)     |             (16/64)           |
 |N|V|V|V|       |S|             |   (if payload len==126/127)   |
 | |1|2|3|       |K|             |                               |
 +-+-+-+-+-------+-+-------------+ - - - - - - - - - - - - - - - +
 |     Extended payload length continued, if payload len == 127  |
 + - - - - - - - - - - - - - - - +-------------------------------+
 |                               |Masking-key, if MASK set to 1  |
 +-------------------------------+-------------------------------+
 | Masking-key (continued)       |          Payload Data         |
 +-------------------------------- - - - - - - - - - - - - - - - +
 :                     Payload Data continued ...                :
 + - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - +
 |                     Payload Data continued ...                |
 +---------------------------------------------------------------+
```

#### 数据帧格式详解

> + FIN：1bit，可以参考TCP，如果FIN位是1，表示这是message的最后一个fragment，如果是0表示还有其他分片
>
> + RSV1/2/3：1bit，一般全为0，当客户端、服务端协商采用WebSocket扩展时，三个标志位可以非0，且值的含义由扩展定义
>
>   如果出现非0值，且没有采用WebSocket扩展，连接出错
>
> + Opcode：4bit，操作代码，决定了应该如何解析后续的数据载何（Data Payload）
>
> + Mask：1bit，表示是否要对数据载荷进行掩码操作，从客户端向服务端发送数据时，需要对开数据进行掩码擦欧总；从服务端向客户端发送数据时，不需要进行掩码操作
>
>   如果服务端收到了没有过掩码操作的数据，需要断开连接
>
> + Payload length：数据载荷长度，7bit或7+16bit或1+64bit，不包括Masking-key的长度
>
> + Masking-key：0或4bit，从客户端传输到服务端的数据帧，数据载荷都要掩码操作，Mask为1，会携带Masking-key，为0则没有
>
> + Payload data：x+y bit，扩展数据x bit+应用数据y bit

#### 掩码算法

- original-octet-i：为原始数据的第i字节。
- transformed-octet-i：为转换后的数据的第i字节。
- j：为`i mod 4`的结果。
- masking-key-octet-j：为mask key第j字节。

```
j = i MOD 4 transformed-octet-i = original-octet-i XOR masking-key-octet-j
```

### 数据传递

> 一旦建立连接，后续的操作都是基于数据帧的传递
>
> WebSocket根据opcode区分操作的类型，比如0x8表示断开连接，0x0-0x2表示数据交互

#### 数据分片

> WebSocket的每条message都会被切分成多个frame，当接收方收到一个frame时，会根据FIN值判断是否为最后一个frame
>
> FIN1表示最后一个frame，可以处理完整的消息，FIN0表示需要继续监听接受剩余的数据帧
>
> opcode在数据交换场景下，表示数据类型
>
> + 0x08表示文本
> + 0x02表示二进制
> + 0x00表示延续帧，完整消息对应的数据帧还未接受完

#### 数据分片例子

```
Client: FIN=1, opcode=0x1, msg="hello"
Server: (process complete message immediately) Hi.
Client: FIN=0, opcode=0x1, msg="and a"
Server: (listening, new message containing text started)
Client: FIN=0, opcode=0x0, msg="happy new"
Server: (listening, payload concatenated to previous message)
Client: FIN=1, opcode=0x0, msg="year!"
Server: (process complete message) Happy new year to you too!
```

### 连接保持+心跳

> WebSocket为了保持客户端、服务端的实时双向痛惜你，需要确保客户端和服务端的TCP通道保持连接没有断开
>
> 但是，如果长时间没有数据往来，保持着会浪费包括的连接资源

> 采用心跳保持客户端与服务端的长时间连接
>
> + 发送方给接收方：ping
>
> + 接收方给发送方：pong
>
> ping和pong对应WebSocket的两个控制帧，opcode对应0x9和0xA

```javascript
// WebSocket服务端向客户单发送ping
ws.ping('', false, true);
```

### Sec-WebSocket-Key/Accept的作用

> 提供基础的防护，避免恶意连接和意外连接

+ 避免服务端收到非法的websocket连接，http客户端不小心连接到websocket服务
+ 确保服务端立即ws连接，因为ws在握手阶段仍然采用http协议，因此ws连接可能会被http服务器处理返回，这个头部可以避免，但不能预防无聊的http服务器故意处理这个头部但不实现ws协议
+ 发起ajax请求时，设置header时，ws有关的头部会被禁止携带，防止不需要的请求升级
+ 防止反向代理返回错误的数据，比如反向代理前后收到两次ws连接的升级请求，把第一次请求的返回cache，第二次直接返回（会出现脏数据）
+ 这个头部的目的不是确保数据的安全性，因为转换计算公式都是公开的，主要是为了预防一些常见的意外情况

### 数据掩码的作用

> 数据掩码的作用是为了增强协议的安全性，但并不是为了保护数据本身，因为算法本身是公开的
>
> 安全并不是为了防止数据泄密，而是为了防止早期版本协议中存在的代理缓存污染攻击



































