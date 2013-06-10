# Summary

A WebSocket-based JSON-RPC 2.0 implementation for Node.js.

# Dependencies

This project uses the
[JSON-RPC serializer](https://github.com/soggie/jsonrpc-serializer) library.
This library (JSON-RPC serializer) implements the JSON-RPC 2.0 specification.

This project uses the same interface provided by
[faye-websocket](https://github.com/faye/faye-websocket-node) and it's very
convenient for you to use this (faye-websocket-node) library. It's possible to
use other transports, but you'd need to implement the interface required by this
project (see below).

# How to use

## RpcNode

To use deimos2, create one RpcNode object for each connection. You **must** pass
a connection object as the first argument to RpcNode constructor:

    var RpcNode = require('deimos2').RpcNode;
    var WebSocket = require('faye-websocket');
    var http = require('http');

    var server = http.createServer();

    server.on('upgrade', function(request, socket, head) {
      if (!WebSocket.isWebSocket(request))
        return;

      var rpcNode = new RpcNode(new WebSocket(request, socket, head));
    });

### RpcNode connection interface

The connection object must have a behaviour similar to
[faye-websocket](https://github.com/faye/faye-websocket-node). It should have
the following events:

 * _message_: When a new message is received.
   * The callback must be a function that receives an _event_ argument. The
     _event_ should have the following attributes:
     * _data_: The message's data. It's either a String (for text frames) or a
       Buffer (for binary frames).
 * _close_: When the transport is closed.

The connection should have the following methods:

 * _send_: Used to send messages.
 * _close_: Used to close the connection.
 * _ping_: Used to send a ping. It should receive a function as argument. This
   function should be called when a matching pong is received.

### RpcNode methods attribute

RpcNode has a _methods_ attribute. This attribute is used when it receives a
remote call.

If present, this attribute **must** be a JavaScript object with the exported
methods. RpcNode will correctly dispatch remote calls to the appropriate
methods.

Methods starting with underscore are considered private and not exported.
Methods starting with "rpc." are reserved by JSON-RPC 2.0 and cannot be used.

The constructor takes a second argument (optional) to initialize
_RpcNode.methods_ attribute.

### RpcNode method interface

When the RpcNode receives a remote call, it'll dispatch it to the correct method
from the methods attribute. Each method **must** be a function that receives the
following arguments (in this order):

 * _node_: This argument will be the calling RpcNode object. It can be used to
   call methods from the client. (Hey! It's a duplex channel).
 * _session_: It can be used to respond to the call. (it's async!).

The _session_ argument have the following members:

 * _params_: The arguments received from the remote call.
 * _response_: A function that responds to the remote call. It receives the
   result as argument.
 * _error_: A function that responds an error. It receives the following
   arguments (in this order):
   * _errorCode_: An integer. Its semantics are defined by the JSON-RPC 2.0
     specification.
   * _errorMessage_: A string. Its semantics are defined by the JSON-RPC 2.0
     specification.
   * _data_: Custom data.
 * _invalidParams_: A function that sends an _invalid_ _params_ error message.
   It receives an optional _customData_ argument.
