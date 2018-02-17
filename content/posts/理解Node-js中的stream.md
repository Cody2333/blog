---
title: 理解nodeJs中的stream
tags: [
    "javascript",
    "nodejs"
]
date: 2018-01-22
categories: [
    "javascript",
    "nodejs"
]
---

## 简介
主要对stream这个概念做一个形象的描述和理解，同时介绍一下比较常用的API。主要参考了Node.js的官方文档。

## stream的种类

*   [Readable](https://nodejs.org/api/stream.html#stream_class_stream_readable) - streams from which data can be read (for example [`fs.createReadStream()`](https://nodejs.org/api/fs.html#fs_fs_createreadstream_path_options)).
*   [Writable](https://nodejs.org/api/stream.html#stream_class_stream_writable) - streams to which data can be written (for example [`fs.createWriteStream()`](https://nodejs.org/api/fs.html#fs_fs_createwritestream_path_options)).
*   [Duplex](https://nodejs.org/api/stream.html#stream_class_stream_duplex) - streams that are both Readable and Writable (for example [`net.Socket`](https://nodejs.org/api/net.html#net_class_net_socket)).
*   [Transform](https://nodejs.org/api/stream.html#stream_class_stream_transform) - Duplex streams that can modify or transform the data as it is written and read (for example [`zlib.createDeflate()`](https://nodejs.org/api/zlib.html#zlib_zlib_createdeflate_options)).

## stream解决的问题

设计 stream API 的一个关键目标是将数据缓冲限制到可接受的级别，从而使得不同传输速率的源可以进行数据的传输，同时不会占用过量的内存。比如，文件的读取。系统从硬盘中读取文件的速度和我们程序处理文件内容的速度是不相匹配的，而且读取的文件可能是很大的。如果不用流来读取文件，那么我们首先就需要把整个文件读取到内存中，然后程序从内存中读取文件内容来进行后续的业务处理。这会极大的消耗系统的内存，并且降低处理的效率（要先读取整个文件，再处理数据）。

stream这个概念是很形象的，就像是水流，可以通过管道，从一处流向另一处。比如从文件输入，最终由程序接收，进行后续的处理。而 [`stream.pipe()`](https://nodejs.org/api/stream.html#stream_readable_pipe_destination_options)就是流中最关键的一个管道方法。

```
// 此处代码实现了从file.txt读取数据，然后压缩数据，然后写入file.txt.gz的过程
const r = fs.createReadStream('file.txt');
const z = zlib.createGzip();
const w = fs.createWriteStream('file.txt.gz');
r.pipe(z).pipe(w);
```
## 缓存机制

Writeable 和 Readable stream 都将数据存储在一个自身内部的buffer中。分别`writable.writableBuffer` or `readable.readableBuffer` 可以得到buffer的数据。有一个参数`highWaterMark` 用来限制这个buffer的最大容量。从而使得流之间的数据传输可以被限制在一定的内存占用下，并且拥有较高的效率。

## Writable Streams

##### 几个典型的可写流：

*   [HTTP requests, on the client](https://nodejs.org/api/http.html#http_class_http_clientrequest)
*   [HTTP responses, on the server](https://nodejs.org/api/http.html#http_class_http_serverresponse)
*   [fs write streams](https://nodejs.org/api/fs.html#fs_class_fs_writestream)
*   [zlib streams](https://nodejs.org/api/zlib.html)
*   [crypto streams](https://nodejs.org/api/crypto.html)
*   [TCP sockets](https://nodejs.org/api/net.html#net_class_net_socket)
*   [child process stdin](https://nodejs.org/api/child_process.html#child_process_subprocess_stdin)
*   [`process.stdout`](https://nodejs.org/api/process.html#process_process_stdout), [`process.stderr`](https://nodejs.org/api/process.html#process_process_stderr)

```
// 示例
const myStream = getWritableStreamSomehow();
myStream.write('some data');
myStream.write('some more data');
myStream.end('done writing data'); // 关闭流，不可再写入。
```

##### 触发的事件：
- close
- **drain**
If a call to [`stream.write(chunk)`](https://nodejs.org/api/stream.html#stream_writable_write_chunk_encoding_callback) returns `false`, the `'drain'` event will be emitted when it is appropriate to resume writing data to the stream.
不能无限制的写入，当writeable stream的内部缓存数据超过`highWaterMark` 的阈值，stream.write会返回false，这是应该停止写入，等到触发drain事件再继续写入。
- **error**
注意，error事件触发的时候，stream不会自动被关闭，需要手动处理关闭。
- finish
- **pipe** 
对应于`readable.pipe()`,有一个readable stream和该stream连通的时候触发
- unpipe
对应于`readable.unpipe()`,有一个readable stream和该stream管道断开的时候触发

## Readable Streams

##### 几个典型的可读流：

*   [HTTP responses, on the client](https://nodejs.org/api/http.html#http_class_http_incomingmessage)
*   [HTTP requests, on the server](https://nodejs.org/api/http.html#http_class_http_incomingmessage)
*   [fs read streams](https://nodejs.org/api/fs.html#fs_class_fs_readstream)
*   [zlib streams](https://nodejs.org/api/zlib.html)
*   [crypto streams](https://nodejs.org/api/crypto.html)
*   [TCP sockets](https://nodejs.org/api/net.html#net_class_net_socket)
*   [child process stdout and stderr](https://nodejs.org/api/child_process.html#child_process_subprocess_stdout)
*   [`process.stdin`](https://nodejs.org/api/process.html#process_process_stdin)

##### 两种模式: flowing and paused

readable stream 创建时都为paused模式，但是可以通过以下几个方法变为flowing：
*   Adding a [`'data'`](https://nodejs.org/api/stream.html#stream_event_data) event handler.
*   Calling the [`stream.resume()`](https://nodejs.org/api/stream.html#stream_readable_resume) method.
*   Calling the [`stream.pipe()`](https://nodejs.org/api/stream.html#stream_readable_pipe_destination_options) method to send the data to a [Writable](https://nodejs.org/api/stream.html#stream_class_stream_writable).

最常用的其实就是stream.pipe()了。

##### 触发的事件：
- close
- data
- end
- readable

```
// 示例
const readable = getReadableStreamSomehow();
readable.on('data', (chunk) => {
  console.log(`Received ${chunk.length} bytes of data.`);
});
readable.on('end', () => {
  console.log('There will be no more data.');
});
```
##### readable.setEncoding(encoding)

调用readable.setEncoding('utf8')可以使得chunk的类型由buffer变为string。
```
const readable = getReadableStreamSomehow();
readable.setEncoding('utf8');
readable.on('data', (chunk) => {
  assert.equal(typeof chunk, 'string');
  console.log('got %d characters of string data', chunk.length);
});
```

## HTTP通信中的应用
看代码和注释应该就能懂了
```
const http = require('http');

const server = http.createServer((req, res) => {
  // req is an http.IncomingMessage, which is a Readable Stream
  // res is an http.ServerResponse, which is a Writable Stream

  let body = '';
  // Get the data as utf8 strings.
  // If an encoding is not set, Buffer objects will be received.
  req.setEncoding('utf8');

  // Readable streams emit 'data' events once a listener is added
  req.on('data', (chunk) => {
    body += chunk;
  });

  // the end event indicates that the entire body has been received
  req.on('end', () => {
    try {
      const data = JSON.parse(body);
      // write back something interesting to the user:
      res.write(typeof data);
      res.end();
    } catch (er) {
      // uh oh! bad json!
      res.statusCode = 400;
      return res.end(`error: ${er.message}`);
    }
  });
});

server.listen(1337);

// $ curl localhost:1337 -d "{}"
// object
// $ curl localhost:1337 -d "\"foo\""
// string
// $ curl localhost:1337 -d "not json"
// error: Unexpected token o in JSON at position 1
```
