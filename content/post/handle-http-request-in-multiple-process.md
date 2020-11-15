+++
categories = ["tech"]
date = "2014-12-16T01:28:26+08:00"
draft = false
slug = ""
tags = ["nodejs"]
title = "Handle HTTP request in multiple processes in NodeJS"

+++

NodeJS is single process based which is good at frequent IO operations. But single process can't fully utilize multi-core CPU. Luckily we have `child-process` module in NodeJS to spawn multiple processes. One of the practical examples is handling HTTP request, especially for a large number of concurrent requests. A common way is to use master-worker pattern, a master process working as a proxy to delegate the HTTP requests to child processes with load balance enabled. Cons of master-worker is rapid resource consuming as each process needs to listen on a different port. Another ninja way is to make each child process listen on the same port. Let's have a look at how to achieve that:

<span class="more"></span>

```javascript
// master.js
var cp = require('child_process'),
    cp1 = cp.fork('child-process.js'),
    cp2 = cp.fork('child-process.js'),
    net = require('net');

var server = net.createServer(); // create a TCP server

server.on('listening', function () {
    /**
     * this is a tricky part
     * it's not sending the whole server object
     * but a stringified message object with server._handle in
     * we will have another post to discuss it later
     */
    cp1.send('server', server);
    cp2.send('server', server);
    server.close();
});

server.listen(1337);

// child.js
var http = require('http');

var server = http.createServer(function (req, res) {
   res.writeHead(200, {'Content-Type': 'text/plain'}); 
   res.end('handled by child, pid: ' + process.pid + '\n');
});

process.on('message', function (m, tcpServer) {
   if (m === 'server') {
      tcpServer.on('connection', function (socket) {
          server.emit('connection', socket);
      });
   }
});
```
In `master.js`, we create a TCP server, have it bound to port 1337, and close it immediately once delegate it to child process. In `child.js`, we create a HTTP server, not like usual, it doesn't listen on any port explicitly. Now we have the TCP server sent from master process (regard it to be the same TCP server instance for now, but actually not. We will discuss it in another post). Once there is connection to the TCP server, we manually emit the `connection` event on HTTP server with the TCP socket passed in. So that any HTTP reqeust to port 1337 will be handled by child process now, one process at a time. Let's use curl to check the result:
```bash
$ curl http://localhost:1337
handled by child, pid: 18169 
$ curl http://localhost:1337
handled by child, pid: 18168
```