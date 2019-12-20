---
title: HTTP学习
date: 2019-02-26 12:42:46
tags: HTTP URL MIME
---

**超文本传输协议HTTP**是用于传输如HTML的超媒体文档的应用层协议

# 基础

## 资源URI：

HTTP请求的内容统称资源，每个资源都有一个URI来进行表示

### URL

统一资源定为符，也被称为Web地址，是URI的最常见的形式

#### 语法

example

```js
const URL = 'http://www.example.com:80/path/to/myfile.html?key1=value1&key2=value2#SomewhereInTheDocument'
```

其中

**http://** => 方案或协议：告诉浏览器使用何种协议。常见的协议：

```js
const Protocols = {
    'data': 'Data URIs',
    'file': '指定主机上文件的名称',
    'ftp': '文本传输协议',
    'http/https': '超文本传输协议/安全的超文本传输协议',
    'mailto': '电子邮件地址',
    'ssh': '安全shell',
    'urn'： '统一资源名称',
    'view-source': '资源的源代码',
    'ws/wss': '（加密的）WebSocket连接'
}
```

**www.example.com** => 主机：既是一个域名，也代表管理该域名的机构。指示了需要向网络上的哪一台主机发起请求

**:80** => 端口：表示用于访问web服务器上资源的技术“门”

HTTP协议标准端口 HTTP：80 HTTPs：443

**/path/to/myfile.html** => 路径：资源的路径，主要是由没有任何物理实体的web服务器抽象处理而成

**key1=value1&key2=value2** => 查询参数

\#**SomewhereInTheDocument** => 资源某一部分的一个锚点

#### Data URL

##### 语法

前缀（data:）、指示数据类型的MIME类型、如果非文本可选的base64标记、数据本身

```js
// 格式：data:[<mediatype>][;base64],<data>
const DataURL1 = 'data:text/plain;base64,SGVsbG8sIFdvcmxkIQ%3D%3D'
const DataURL2 = 'data:text/html,%3Ch1%3EHello%2C%20World!%3C%2Fh1%3E'
const DataURL3 = 'data:text/html,<script>alert('hi');</script>'
```

注意点：

- 格式中的逗号
- HTML代码格式化
- 长度限制
- 缺乏错误处理，如MIME类型错误或base64编码错误
- 不支持查询字符串

### URN

通过特定命名空间中的唯一名称来标识资源，是URI的另一种形式

## MIME类型

多用途Internet邮件扩展（MIME）类型：是一种标准化的方式来表示文档的性质和格式。

浏览器使用MIME类型（而不是文件扩展名）来确定如何处理文档

### 语法

#### 通用结构

type/subtype

对大小写不敏感，传统写法是小写

#### 独立类型

表明了对文件的分类

| 类型        | 描述                                      | 示例                                                         |
| ----------- | ----------------------------------------- | ------------------------------------------------------------ |
| text        | 普通文本，理论上是人类可读                | text/plain, text/html, text/css                              |
| image       | 某种图像，不包括视频，但包括动态图，如gif | imag/gif, image/png, img/jpeg, img/bmp, image/webp           |
| audio       | 某种音频                                  | audio/midi, audio/mpeg, audio/webm, audio/ogg, audio/wav, audio/* |
| video       | 某种视频                                  | video/webm, video/ogg                                        |
| application | 某种二进制数据                            | application/json, application/javascript, application/octet-stream |

##### application/actet-stream 

应用程序文件的默认值，表示未知的应用程序文件，浏览器不会自动执行或询问执行，会如同设置了HTTP头Content-Disposition值为attachment的文件一样来对待这类文件

##### text/plain

文本文件默认值，表示未知的文本文件，但浏览器认为是可以直接展示的

注意：假设要用link链接下载一个CSS文件，但是提供的类型是text/plain，那么浏览器并不会认为这是一个有效的css文件，必须使用text/css

##### text/css

在网页中要被解析为CSS的任何CSS文件必须指定为 text/css。通常，服务器不识别以.css为后缀的文件的MIME类型，而是将其以MIME为text/plain 或 application/octet-stream 来发送给浏览器，而大多数浏览器不识别其为CSS文件，直接忽略掉。特别要注意为CSS文件提供正确的MIME类型

##### text/html

所有的HTML内容都应使用

##### javascript types

application/javascript, application/ecmascript

##### 图片类型

被广泛支持的，web安全的，可随时在页面中使用的：image/gif, image/jpeg, image/png, image/svg+xml

另外，如很多浏览器支持icon类型的图标作为favicons或类似的图标，并且浏览器在MIME类型中的image/x-icon支持ICO图像

#### Multipart类型

表示细分领域的文件类型的种类，对应不同的MIME类型，是复合文件的一种表现方式。

##### multipart/form-data

用于HTML表单从浏览器发送信息给服务器。由边界线（一个由__开始的字符串）划分出的不同部分组成。每一部分有自己的实体以及HTTP请求头，Content-Disposition和Content-Type用于文件上传领域

##### multipart/byteranges

用于把部分的响应报文发送回浏览器，当发送状态码206 Partial Content 时，这个MIME类型用于指出这个文件由若干部分组成，每一个都有其请求范围。每一个不同的部分都有Content-Type来说明文件的实际类型，以及Content-Range说明范围

HTTP对不能处理的复合文件使用特殊的方式：将信息直接传送给浏览器，可能会建立一个“另存为”窗口，但是却不知道如何去显示内联文件

### 设置正确的类型的重要性

很多web服务器使用默认的application/actet-stream来发送未知类型。出于一些安全原因，对于这些资源浏览器不允许设置一些自定义默认操作，导致用户必须存储到本地使用。常见的导致服务器配置错误的文件类型：

- RAR编码文件。服务器将发送application/x-rar-compressed作为MIME类型，用户不会将其定义为有用的默认操作
- 音频或视频文件。只有正确设置了MIME类型的文件才能被识别播放
- 专有文件类型。使用application/octet-stream作为特殊处理是不被允许的，对于一般的MIME类型浏览器不允许定义默认行为。

### MIME嗅探

在确实MIME类型或客户端认为文件设置了错误的MIME类型时，浏览器可能会通过查看资源来进行MIME嗅探。每一个浏览器在不同的情况下会执行不同的操作。会有安全问题，有的MIME类型表示可执行内容，而有些是不可执行内容。浏览器可以通过请求头Content-Type来设置X-Content-Type-Options以阻止MIME嗅探。

### 其他传送文件类型的方法

MIME 类型不是传达文件类型信息的唯一方式

- 名称后缀，特别是windows系统
- 魔术数字：不同类型的文件的语法通过查看结构来允许文件类型推断

# 概要

## 基于HTTP的组件系统

### 客户端：user-agent

要展现一个网页，浏览器首先发送一个请求来获取页面的HTML文档，再解析文档中的资源信息发送其他请求，获取可执行脚本或CSS样式来进行页面布局渲染，以及一些其它页面资源（如图片和视频等）。然后，浏览器将这些资源整合到一起，展现出一个完整的文档，也就是网页。浏览器执行的脚本可以在之后的阶段获取更多资源，并相应地更新网页。

### 服务端

可以是共享负载（负载均衡）的一组服务器组成的计算机集群，也可以是一种复杂的软件，通过向其他计算机（如缓存，数据库服务器，电子商务服务器 ...）发起请求来获取部分或全部资源。

### 代理proxies

- 缓存（公开/私有，如浏览器的缓存）
- 过滤（如反病毒扫描，家长控制）
- 负载均衡（让多个服务器服务不同的请求）
- 认证（对不同资源进行权限管理）
- 日志记录（允许存储历史信息）

## 基本性质

- 简单易读
- 可扩展：通过Headers非常容易扩展
- 无状态但有会话：在同一个连接中，两个执行成功的请求之间是没有关心的，但是，头部cookies，让每个请求都能共享相同的上下文，达成相同的状态
- 连接：只需要可靠，或不丢失信息（至少返回错误），因此依赖于TCP进行消息传递，但是连接并不是必须的

## 可控的特性

- 缓存：服务端能告诉代理和客户端哪些文档需要被缓存，缓存多久，而客户端也能够命令中间的缓存代理来忽略存储的文档。
- 开放同源限制：可以通过修改Headers来开放限制
- 认证：Authenticate相似的头部或用HTTP Cookies来设置指定的会话
- 代理和隧道：服务器和/或客户端是处于内网的，对外网隐藏真实 IP 地址。因此 HTTP 请求就要通过代理越过这个网络屏障。
- 会话：使用Cookies用服务端的状态来发起请求

## 客户端和服务端进行信息交互时的过程：

1. 打开一个TCP连接
2. 发送一个HTTP报文
3. 读取服务端返回的报文信息
4. 关闭连接或者为后续请求重用连接

## HTTP报文

类型：请求、响应

格式：

- 请求：Method + Path + HTTP Version + Headers +body
- 响应：HTTP Version + Status Code + Status Message +Header +body
