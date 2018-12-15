# grpc-gin
proto grpc gin plugin , use the proto files  to generate gin web handers

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


you can use the generated web handers like this

```
r := gin.Default()
s := pb.NewGreeterGinServer(&GrpcServer{})
s.RegisterGreeterHander(r)
r.Run(":8080")

```

Now you can use postman to debug you grpc servers





