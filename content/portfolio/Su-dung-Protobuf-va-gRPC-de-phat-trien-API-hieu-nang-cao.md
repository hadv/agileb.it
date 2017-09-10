---
title: "Sử dụng Protobuf và gRPC để phát triển API hiệu năng cao"
date: 2017-09-04T18:45:42+07:00
draft: false
image: "img/portfolio/gRPC.png"
---

Trong thời điểm hiện tại thì JSON REST API vẫn đang rất phổ biến và phổ thông nhất bởi tính dễ sử dụng của nó. Tuy nhiên, các hãng công nghệ lớn luôn luôn tìm kiếm những phương pháp mới để nâng cao hiệu năng cho sản phẩm của họ. Trong bài viết này, chúng ta sẽ tìm hiểu và khám phá về một framework RPC mới dựa trên [protobufs](https://developers.google.com/protocol-buffers/) và HTTP/2 của **Google** có tên là [gRPC](http://www.grpc.io).

Một tính năng ưu việt của gRPC là sử dụng HTTP/2, cho phép chúng ta có thể đồng thời phát triển cả HTTP/1.1 REST+JSON API như thông thường và cả giao tiếp hiệu năng cao thông qua gRPC trên cùng một TCP port. Phiên bản Go1.6 thì đã kèm theo thư viện `net/http2`.

![](https://cdn-images-1.medium.com/max/800/1*fUR37-hx7Q2NvjT1PGSXpA.png)

Chúng ra sẽ bắt đầu bằng việc sử dụng gRPC để xây dựng một ứng dụng lấy tên là **EchoService** bằng `Golang`. Ứng dụng này chỉ đơn giản là trả về chính nội dung message mà nó nhận được. 

### Cài đặt

Cài đặt ProtoBuffers 3.0

```
$ brew install --devel protobuf
```

Sau đó chạy `go get -u` để cài đặt một số thư viện `Golang` cần thiết khác
 
```
go get -u github.com/gengo/grpc-gateway/protoc-gen-grpc-gateway
go get -u github.com/gengo/grpc-gateway/protoc-gen-swagger
go get -u github.com/golang/protobuf/protoc-gen-go
```

### Tạo một dịch vụ gRPC

Trước tiên, chúng ta mô tả ứng dụng trong file `service.proto` theo chuẩn *proto3* như sau:

```
syntax = "proto3";
option go_package = "echo";

// Echo Service
//
// Echo Service API consists of a single service which returns a message.
package hadv.grpc.echo;

import "google/api/annotations.proto";

// Message represents a simple message sent to the Echo service.
message Message {
	// Id represents the message identifier.
	string id = 1;
}

// Echo service responds to incoming echo requests.
service EchoService {
	// Echo method receives a simple message and returns it.
	// The message posted as the id parameter will also be returned.
	rpc Echo(Message) returns (Message) {
		option (google.api.http) = {
			post: "/v1/example/echo/{id}"
		};
	}
	// EchoBody method receives a simple message and returns it.
	rpc EchoBody(Message) returns (Message) {
		option (google.api.http) = {
			post: "/v1/example/echo_body"
			body: "*"
		};
	}
}
```

Trước khi cài đặt tính năng cho dịch vụ **EchoService** thì chúng ta cần chạy lệnh sau để tự động sinh ra các stub class cần thiết trong file `service.pb.go`

```
$ protoc -I/usr/local/include -I. \
 -I$GOPATH/src \
 -I$GOPATH/src/github.com/gengo/grpc-gateway/third_party/googleapis \
 --go_out=Mgoogle/api/annotations.proto=github.com/gengo/grpc-gateway/third_party/googleapis/google/api,plugins=grpc:. \
 service.proto
```

Sau đó, chúng ta có thể cài đặt **EchoService** như trong file `echo.go` sau.

```
package main

import (
	"github.com/golang/glog"
	examples "github.com/hadv/grpc"
	"golang.org/x/net/context"
	"google.golang.org/grpc"
	"google.golang.org/grpc/metadata"
)

// Implements of EchoServiceServer

type echoServer struct{}

func newEchoServer() examples.EchoServiceServer {
	return new(echoServer)
}

func (s *echoServer) Echo(ctx context.Context, msg *examples.Message) (*examples.Message, error) {
	glog.Info(msg)
	return msg, nil
}

func (s *echoServer) EchoBody(ctx context.Context, msg *examples.Message) (*examples.Message, error) {
	glog.Info(msg)
	grpc.SendHeader(ctx, metadata.New(map[string]string{
		"foo": "foo1",
		"bar": "bar1",
	}))
	grpc.SetTrailer(ctx, metadata.New(map[string]string{
		"foo": "foo2",
		"bar": "bar2",
	}))
	return msg, nil
}
```

Để tạo entry point và khởi động echo service thì chúng ra tạo ra một file `main.go` như sau.

```
package main

import (
	"flag"
	"net"

	"github.com/golang/glog"
	examples "github.com/hadv/grpc"
	"google.golang.org/grpc"
)

func Run() error {
	listen, err := net.Listen("tcp", ":9090")
	if err != nil {
		return err
	}
	server := grpc.NewServer()
	examples.RegisterEchoServiceServer(server, newEchoServer())
	server.Serve(listen)
	return nil
}

func main() {
	flag.Parse()
	defer glog.Flush()

	if err := Run(); err != nil {
		glog.Fatal(err)
	}
}
```

Để kết nối đến với echo server trên thì chúng ra có thể cài đặt `EchoServiceClient` kết nối đến. Tuy nhiên, ở đây tôi bỏ qua bước này và chuyển sang tạo reverse-proxy cho kết nối thông qua REST JSON API. Để tạo ra reverse-proxy `service.pb.gw.go`, chúng ra chạy lệnh sau.

```
$ protoc -I/usr/local/include -I. \
 -I$GOPATH/src \
 -I$GOPATH/src/github.com/gengo/grpc-gateway/third_party/googleapis \
 --grpc-gateway_out=logtostderr=true:. \
 ./service.proto
```

Để tạo entry-point và khởi động server cho REST JSON API thì chúng ta tạo ra file `main.go` như sau

```
package main

import (
	"flag"
	"net/http"

	"github.com/gengo/grpc-gateway/runtime"
	"github.com/golang/glog"
	echo "github.com/hadv/grpc/echo"
	"golang.org/x/net/context"
	"google.golang.org/grpc"
)

var (
	echoEndpoint = flag.String("echo_endpoint", "localhost:9090", "endpoint of EchoService")
)

func Run(address string, opts ...runtime.ServeMuxOption) error {
	ctx := context.Background()
	ctx, cancel := context.WithCancel(ctx)
	defer cancel()

	mux := runtime.NewServeMux(opts...)
	dialOpts := []grpc.DialOption{grpc.WithInsecure()}
	err := echo.RegisterEchoServiceHandlerFromEndpoint(ctx, mux, *echoEndpoint, dialOpts)
	if err != nil {
		return err
	}

	http.ListenAndServe(address, mux)
	return nil
}

func main() {
	flag.Parse()
	defer glog.Flush()

	if err := Run(":8080"); err != nil {
		glog.Fatal(err)
	}
}
```

Sau khi build và khởi động server thì chúng ta có thể gọi REST API bằng lệnh `curl -X POST "http://localhost:8080/v1/example/echo/grpc"` và nhận được kết quả như sau

```
{"id":"grpc"}
```

Ngoài ra, chúng ra có thể chạy lệnh sau để generate ra file `service.swagger.json` mô tả API theo chuẩn của **Swagger**.

```
$ protoc -I/usr/local/include -I. \
>  -I$GOPATH/src \
>  -I$GOPATH/src/github.com/gengo/grpc-gateway/third_party/googleapis \
>  --swagger_out=logtostderr=true:. \
> ./service.proto
```

Chúng ta có thể xem mô tả API bằng cách mở file `service.swagger.json` trên [Swagger Editor](http://editor.swagger.io/#/) như hình sau.

![alt text](https://s3-ap-southeast-1.amazonaws.com/kipalog.com/qgju5yck5z_Swagger_Editor_Echo_Service.jpg)

Như vậy, cùng với *protobuf* và gRPC chúng ta có thể nhanh chóng tạo ra các API đồng thời cho HTTP/1.1 REST+JSON API và RPC dựa trên protobuf và HTTP/2.

Chi tiết về source code các bạn có thể xem trên [github của tôi](https://github.com/hadv/go-grpc)

