# Express + WebSocket

## Express服务代码
```js
var express = require('express');
var fs = require('fs');
var path = require('path');
var bodyParser = require('body-parser');

var app = express();


app.use(bodyParser.json());
app.use(bodyParser.urlencoded({
  extended: false
}));

// 访问静态资源
app.use(express.static(path.resolve(__dirname, 'dist')));

app.get("/test", function(req, res) {
  res.send("hello");
})

// 访问单页
app.get('*', function (req, res) {
  var html = fs.readFileSync(path.resolve(__dirname, 'dist/index.html'), 'utf-8');
  // res.header("Access-Control-Allow-Origin", "*");
  res.send(html);
});

// 监听
app.listen(3000, function () {
  console.log('success listen...http://localhost:3000');
});
```

## html代码
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title> 
</head>
<body>
    
</body>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/socket.io/2.1.1/socket.io.js"></script>
    <script>
        const socket = new WebSocket('ws://10.1.2.150:8888');
        socket.addEventListener('message', function (event) {
            console.log('Message from server', event.data);
        });
        console.log("33333");
    </script>
</html>
```

## WebSocket服务代码
```js
const WebSocket = require('ws');

var listen_port = 8888;
const wss = new WebSocket.Server({
  port: listen_port,
});

wss.on('connection', function connection(ws) {
  ws.on('message', function incoming(message) {
    console.log('received: %s', message);
  });

  ws.send('something');
});
console.log("ws://localhost:" + listen_port);
```
