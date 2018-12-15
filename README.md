# grpc-gin
proto grpc gin plugin , use the proto files  to generate gin web interface

## install
```
 cp -r  ginweb webapi  $GOPATH/src/github.com/golang/protobuf/protoc-gen-go/

```

add below code 

```
import _ "github.com/golang/protobuf/protoc-gen-go/webapi"
import _ "github.com/golang/protobuf/protoc-gen-go/ginweb"
```

to 
$GOPATH/src/github.com/golang/protobuf/protoc-gen-go/link_grpc.go

```
cd  $GOPATH/src/github.com/golang/protobuf/protoc-gen-go/
go install
```


example 

```
protoc -I helloworld/ helloworld/helloworld.proto --go_out=plugins=grpc+ginweb:helloworld

protoc -I helloworld/ helloworld/helloworld.proto --go_out=plugins=grpc+webapi:helloworld
```

the generated file  will be added below code

```
// GreeterServer is the server API for Greeter service.
type GreeterServer interface {
	// Sends a greeting
	SayHello(context.Context, *HelloRequest) (*HelloReply, error)
	Echo(context.Context, *EchoReq) (*EchoResp, error)
}

func RegisterGreeterServer(s *grpc.Server, srv GreeterServer) {
	s.RegisterService(&_Greeter_serviceDesc, srv)
}

func _Greeter_SayHello_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
	in := new(HelloRequest)
	if err := dec(in); err != nil {
		return nil, err
	}
	if interceptor == nil {
		return srv.(GreeterServer).SayHello(ctx, in)
	}
	info := &grpc.UnaryServerInfo{
		Server:     srv,
		FullMethod: "/helloworld.Greeter/SayHello",
	}
	handler := func(ctx context.Context, req interface{}) (interface{}, error) {
		return srv.(GreeterServer).SayHello(ctx, req.(*HelloRequest))
	}
	return interceptor(ctx, in, info, handler)
}

func _Greeter_Echo_Handler(srv interface{}, ctx context.Context, dec func(interface{}) error, interceptor grpc.UnaryServerInterceptor) (interface{}, error) {
	in := new(EchoReq)
	if err := dec(in); err != nil {
		return nil, err
	}
	if interceptor == nil {
		return srv.(GreeterServer).Echo(ctx, in)
	}
	info := &grpc.UnaryServerInfo{
		Server:     srv,
		FullMethod: "/helloworld.Greeter/Echo",
	}
	handler := func(ctx context.Context, req interface{}) (interface{}, error) {
		return srv.(GreeterServer).Echo(ctx, req.(*EchoReq))
	}
	return interceptor(ctx, in, info, handler)
}

var _Greeter_serviceDesc = grpc.ServiceDesc{
	ServiceName: "helloworld.Greeter",
	HandlerType: (*GreeterServer)(nil),
	Methods: []grpc.MethodDesc{
		{
			MethodName: "SayHello",
			Handler:    _Greeter_SayHello_Handler,
		},
		{
			MethodName: "Echo",
			Handler:    _Greeter_Echo_Handler,
		},
	},
	Streams:  []grpc.StreamDesc{},
	Metadata: "helloworld/helloworld.proto",
}

type GreeterGinServer struct {
	GServer GreeterServer
}

func NewGreeterGinServer(s GreeterServer) *GreeterGinServer {
	web := &GreeterGinServer{
		GServer: s,
	}
	return web
}
func (s *GreeterGinServer) SayHello(c *gin.Context) {
	var in HelloRequest
	if err := jsonpb.Unmarshal(c.Request.Body, &in); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		return
	}
	if resp, err := s.GServer.SayHello(c, &in); err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
	} else {
		c.JSON(http.StatusOK, resp)
	}
}
func (s *GreeterGinServer) Echo(c *gin.Context) {
	var in EchoReq
	if err := jsonpb.Unmarshal(c.Request.Body, &in); err != nil {
		c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
		return
	}
	if resp, err := s.GServer.Echo(c, &in); err != nil {
		c.JSON(http.StatusInternalServerError, gin.H{"error": err.Error()})
	} else {
		c.JSON(http.StatusOK, resp)
	}
}
func (s *GreeterGinServer) RegisterGreeterHander(e *gin.Engine) {
	greeter := e.Group("/helloworld.Greeter")
	{
		greeter.POST("/SayHello", s.SayHello)
		greeter.POST("/Echo", s.Echo)
	}
}

```


you can use the generated web interface like this

```
r := gin.Default()
s := pb.NewGreeterGinServer(&GrpcServer{})
s.RegisterGreeterHander(r)
r.Run(":8080")

```





