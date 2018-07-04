# 参考
[protobuf.js](https://github.com/dcodeIO/protobuf.js)

# test.proto
```
syntax = "proto3";
package awesomepackage;

message AwesomeMessage {
    string awesome_field = 1; // becomes awesomeField
}
```

# html
```
<script src="protobuf.js/dist/protobuf.js"></script>

var AwesomeMessage;
protobuf.load("test.proto", function (err, root) {
  // Obtain a message type
  AwesomeMessage = root.lookupType("awesomepackage.AwesomeMessage");
  var payload = { awesomeField: "AwesomeString" };
  var errMsg = AwesomeMessage.verify(payload);
  if (errMsg)
    throw Error(errMsg);

  // Create a new message
  var message = AwesomeMessage.create(payload); // or use .fromObject if conversion is necessary

  // Encode a message to an Uint8Array (browser) or Buffer (node)
  var buffer = AwesomeMessage.encode(message).finish();
  socket.send(buffer);
}
```

# nodejs
```
const WebSocket = require('ws');
var protobuf = require("protobufjs");

var listen_port = 8888;
var AwesomeMessage;
protobuf.load("test.proto", function(err, root) {
  AwesomeMessage  = root.lookupType("awesomepackage.AwesomeMessage");
  const wss = new WebSocket.Server({
    port: listen_port,
  });

  wss.on('connection', function connection(ws) {
    ws.on('message', function incoming(message) {
      console.log('received: %s', JSON.stringify(message));
      var message = AwesomeMessage.decode(message);
      console.log(JSON.stringify(message));
    });

    ws.send('something');
  });
});

console.log("ws://localhost:" + listen_port);
```
