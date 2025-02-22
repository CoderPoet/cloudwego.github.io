---
title: "Client"
linkTitle: "Client"
weight: 3
description: >
---

The cwgo tool supports generating the calling code of HTTP Client or RPC Client through IDL, which is convenient for users to develop.

## Basic commands

Use `cwgo client -h` to view the help command for generating client code.

```sh
cwgo client -h
NAME:
    cwgo client - generate RPC or HTTP client

                  Examples:
                    # Generate RPR client code
                    cwgo client --type RPC --idl {{path/to/IDL_file.thrift}} --service {{svc_name}}

                    # Generate HTTP client code
                    cwgo client --type HTTP --idl {{path/to/IDL_file.thrift}} --service {{svc_name}}


USAGE:
    cwgo client [command options] [arguments...]

OPTIONS:
    --service value Specify the service name.
    --type value Specify the generate type. (RPC or HTTP) (default: "RPC")
    --module value, --mod value Specify the Go module name to generate go.mod.
    --idl value Specify the IDL file path. (.thrift or .proto)
    --out_dir value, -o value Specify the output path. (default: biz/http)
    --registry value Specify the registry, default is None
    --proto_search_path value, -I value [ --proto_search_path value, -I value ] Add an IDL search path for includes. (Valid only if idl is protobuf)
    --pass value [ --pass value ] pass param to hz or kitex
    --help, -h show help (default: false)
```

## Specification

```console
--service specify service name
--type specifies the generation type, supports parameters RPC, HTTP
--module specifies the generated module name
--idl specify IDL file path
--out_dir specify the output path
--registry specifies the service registration component, currently only useful for RPC type, supports parameters ZK, NACOS, ETCD, POLARIS
--proto_search_path Add IDL search path, only valid for pb
--pass value parameter passed to hz and kitex
```

## RPC Client

### Write IDL

```go
  // hello. thrift
namespace go hello.example

struct HelloReq {
     1: string Name
}

struct HelloResp {
     1: string RespBody;
}

service HelloService {
     HelloResp HelloMethod(1: HelloReq request);
     HelloResp HelloMethod1(1: HelloReq request);
     HelloResp HelloMethod2(1: HelloReq request);
}
```

### Commands

```sh
cwgo client --type RPC --idl hello.thrift --service hellotest
```

### Generate Code

```console
├── hello. thrift # IDL file
├── kitex_gen # Generate code related to IDL content
│ └── hello
│ └── example
│ ├── hello.go # The product of thriftgo, the go code containing the content defined by hello.thrift
│ ├── helloservice
│ │ ├── client.go # provides NewClient API
│ │ ├── helloservice.go # Provides some definitions shared by client.go and server.go
│ │ ├── invoker.go
│ │ └── server.go # provides NewServer API
│ ├── k-consts.go
│ └── k-hello.go # code generated by kitex outside of thriftgo's product
└── rpc
     └── hellotest
         ├── hellotest_client.go # client wrapper code
         ├── hellotest_default.go # client default implementation code
         └── hellotest_init.go # client initialization code
```

## HTTP Client

### Write IDL

To write a simple IDL to generate HTTP Client, you need to add `api.$method` and `api.base_domain` to fill `uri` and `host`.

```thrift
  // hello. thrift
namespace go hello.example

struct HelloReq {
     1: string Name (api. query="name");
}

struct HelloResp {
     1: string RespBody;
}


service HelloService {
     HelloResp HelloMethod1(1: HelloReq request) (api.get="/hello1");
     HelloResp HelloMethod2(1: HelloReq request) (api.get="/hello2");
     HelloResp HelloMethod3(1: HelloReq request) (api.get="/hello3");
}(
      api.base_domain="http://127.0.0.1:8888";
  )
```

### Commands

Execute the following basic commands to generate the client

```sh
cwgo client --type HTTP --idl hello.thrift --service hellotest
```

### Generate Code

A default client implementation is provided in `hello_service.go`, and users can use it directly. If there is a need for custom configuration, you can use the `options` provided in `hertz_client.go` to customize the complex configuration of the Client.

```console
.
├── biz
│ └── http
│ └── hello_service
│ ├── hello_service.go # client initialization and calling code
│ └── hertz_client.go # client specific implementation code
├── hello. thrift # IDL file
└── hertz_gen #IDL content-related generated code
     └── hello
         └── example
             └── hello.go
```

client default implementation code

```go
var defaultClient, _ = NewHelloServiceClient("http://127.0.0.1:8888")

func HelloMethod1(context context.Context, req *example.HelloReq, reqOpt ...config.RequestOption) (resp *example.HelloResp, rawResponse *protocol.Response, err error) {
    return defaultClient.HelloMethod1(context, req, reqOpt...)
}

func HelloMethod2(context context.Context, req *example.HelloReq, reqOpt ...config.RequestOption) (resp *example.HelloResp, rawResponse *protocol.Response, err error) {
    return defaultClient.HelloMethod2(context, req, reqOpt...)
}

func HelloMethod3(context context.Context, req *example.HelloReq, reqOpt ...config.RequestOption) (resp *example.HelloResp, rawResponse *protocol.Response, err error) {
    return defaultClient.HelloMethod3(context, req, reqOpt...)
}
```
