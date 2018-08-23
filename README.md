# gRPC-Web

gRPC-Web is a Javascript client library that enables web apps to interact
gRPC servers directly, without needing an HTTP intermediary. You can find
out much more about gRPC on the [official gRPC website](https://grpc.io).

> The current gRPC-Web release is a beta release. We expect to announce
> general availability (GA) by October 2018.

## Background

gRPC-Web has been used for some time by Google and Alphabet projects that
use the [Closure compiler](https://github.com/google/closure-compiler)
and its TypeScript generator (which has not yet been open-sourced).

gRPC-Web clients connect to gRPC servers via a special gateway proxy. Our
provided version uses [Envoy](https://www.envoyproxy.io/), which offers
built-in support for gRPC-Web and will become the default gateway when
gRPC-Web goes GA.

In the future, we expect gRPC-Web to be supported in a variety of
language-specific Web frameworks, such as Python, Java, and Node.js. See
the [roadmap](https://github.com/grpc/grpc-web/blob/master/ROADMAP.md)
for more information.

## Quick start

You can try out gRPC-Web by running a simple echo example in your browser.

First, clone this repository:

```shell
$ git clone https://github.com/grpc/grpc-web
```

From the repo root directory:

```shell
$ docker-compose up echo-server envoy commonjs-client
```

Open a browser tab and inspect http://localhost:8081/echotest.html.

## How it works

Let's take a look at how gRPC-Web works with a simple example. You can find out
how to build, run, and explore the example yourself in
[Build and Run the Echo Example](net/grpc/gateway/examples/echo).

### 1. Define your service

The first step when creating any gRPC service is to define it. Like all gRPC
services, gRPC-Web uses [protocol buffers](https://developers.google.com/protocol-buffers/)
to define its RPC service methods and their message request and response types.

```grpc
service EchoService {
  rpc Echo(EchoRequest) returns (EchoResponse);

  rpc ServerStreamingEcho(ServerStreamingEchoRequest)
      returns (stream ServerStreamingEchoResponse);
}
```

### 2. Build the server

Next, you need to create a gRPC server that implements the service interface and a
gateway that allows the client to connect to the server. Our example builds a
simple C++ gRPC backend server and the Envoy proxy. You can find out more in
the [Echo Example](net/grpc/gateway/examples/echo).

### 3. Write your JS client

Once the server and gateway are up and running, you can start making gRPC calls
from the browser!

Create your client

```js
var echoService = new proto.grpc.gateway.testing.EchoServiceClient('http://localhost:8080');
```

Make a unary RPC call:

```js
var unaryRequest = new proto.grpc.gateway.testing.EchoRequest();
unaryRequest.setMessage(msg);

const metadata = {};

echoService.echo(unaryRequest, metadata,
  function(err, response) {
    console.log(response.getMessage());
  });
```

Server-side streaming is also supported:

```js
var stream = echoService.serverStreamingEcho(streamRequest, {});

stream.on('data', function(response) {
  console.log(response.getMessage());
});
```

## Proxy interoperability

Multiple proxies currently support the gRPC-Web protocol.

> The default proxy is currently [Envoy](https://envoyproxy.io), which supports
> gRPC-Web out of the box.

### Envoy

To run the echo example with Envoy:

```shell
$ docker-compose up echo-server envoy commonjs-client
```

### nginx

An alternative is to use a specially configured [nginx](https://nginx.org) proxy: 

```shell
$ docker-compose up echo-server nginx commonjs-client
```

### gRPC-Web Go Proxy

Finally, you can also try the
[gRPC-Web Go Proxy](https://github.com/improbable-eng/grpc-web/tree/master/go/grpcwebproxy):

```shell
$ docker-compose up echo-server grpcwebproxy binary-client
```

## Client configuration options

You can generate a client stub from your Protobuf definitions using a `protoc`
command like this:

```shell
$ protoc -I=$DIR echo.proto \
  --plugin=protoc-gen-grpc-web=/path-to/protoc-gen-grpc-web \
  --js_out=import_style=commonjs:$OUT_DIR \
  --grpc-web_out=import_style=commonjs,mode=grpcwebtext,out=echo_grpc_pb.js:$OUT_DIR
```

### Import style

gRPC-Web supports two different import styles that you can specify using the `--js_out` flag:

Style | Import syntax | Flag
:-----|:--------------|:----
[Closure](https://developers.google.com/closure/library/) (**default**) | `goog.require()` | `import_style=closure`
[CommonJS](https://requirejs.org/docs/commonjs.html) | `require()` | `import_style=commonjs`

> **Note**: ES6-style `import` is not yet supported.

### Wire format mode

gRPC-Web supports two different wire formats that you can specify using the `--grpc-web_out` flag:

Content type | Encoding | Supported calls | Flag
:------------|:---------|:----------------|:----
`application/grpc-web-text` (**default**) | [base64](https://en.wikipedia.org/wiki/Base64) | Unary, streaming | `mode=grpcwebtext`
`application/grpc-web+proto` | Binary Protobuf | Unary | `mode=grpcweb`

For more information about the gRPC-Web wire format, please see the
[protocol spec](https://github.com/grpc/grpc/blob/master/doc/PROTOCOL-WEB.md#protocol-differences-vs-grpc-over-http2).
