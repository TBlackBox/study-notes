# 简介

go使用grpc,然而grpc一般和protobuf 一起使用，这里就用protobuf



# 使用案列

1. 下载并安装protoc编译软件

2. 安装相关的包

   - 安装 golang 的proto工具包

     ```
     go get -u github.com/golang/protobuf/proto
     ```

   - 安装 goalng 的proto编译支持

     ```
     go get -u github.com/golang/protobuf/protoc-gen-go
     ```

   - 安装 GRPC 包

     ```
     go get -u google.golang.org/grpc
     ```

     

## 编译器调用

 [文档说明](https://developers.google.com/protocol-buffers/docs/reference/go-generated#package)

1. .proto文件生成.go代码需要一个插件，需在go1.16 或以上运行

```shell
go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
```

安装的`protoc-gen-go`将在`$GOPATH`下

2. .go文件的输出取决go_out的配置,后面可以跟放go文件的的位置,需要把文件夹创建好。

3. 输出文件的位置，取决于编译器的几个标志：

- 如果指定了path = import标志，则将输出文件放置在以Go软件包的导入路径命名的目录中。

```
例如，输入文件protos / buzz.proto的go导入路径为example.com/project/protos/fizz，则输出文件为example.com/project/protos/fizz/buzz.pb.go。如果未指定路径标志，则这是默认的输出模式
```

- 如果`module=$PREFIX`指定了标志，则将输出文件放置在以Go软件包的导入路径命名的目录中，但将从输出文件名中删除指定的目录前缀。

```
例如，具有protos/buzz.proto Go导入路径example.com/project/protos/fizz且 example.com/project指定为module前缀的输入文件将导致输出文件位于protos/fizz/buzz.pb.go。在模块路径之外生成任何Go软件包都会导致错误。此模式对于将生成的文件直接输出到Go模块很有用。
```

如果`paths=source_relative`指定了标志，则将输出文件放置在与输入文件相同的相对目录中。

```
例如，输入文件将`protos/buzz.proto` 导致输出文件位于`protos/buzz.pb.go`。
```


`protoc-gen-go`通过`go_opt`调用时传递标志来提供特定于的标志`protoc`。`go_opt`可以传递多个标志。例如，在运行时：

```shell
protoc --proto_path=src --go_out=out --go_opt=paths=source_relative foo.proto bar/baz.proto
```

编译器将读取输入文件`foo.proto`，并`bar/baz.proto` 从内部`src`目录，并写入输出文件`foo.pb.go`，并`bar/baz.pb.go` 到`out`目录。**必要时，编译器会自动创建嵌套的输出子目录，但不会自行创建输出目录**。

4. 案列

   ```shell
   protoc --proto_path=grpc --go_out=grpc/out --go_opt=paths=source_relative test.proto
   ```

**简单参数说明：**

- `test.proto` 文件在grpc下
- 输出文件到`grpx/src`下

- `--proto_path`可以用于指定.proto文件的路径。

- `-I`也可以指定protoc的搜索import的proto的文件夹。在`MacOS`操作系统中protobuf把一些扩展的proto放在了`/usr/local/include`对应的文件夹中，一些第三方的Go库放在了gopath对应的包下，所以这里都把它们加上了。



## 特别注意

为了生成Go代码，必须为每个`.proto`文件提供Go包的导入路径（包括`.proto`所生成文件所依赖的传递路径）。有两种方法可以指定Go导入路径：

1. 在`.proto`文件中声明(这种方法推荐)，示列：

   ```
   option go_package = "example.com/project/protos/fizz";
   ```

2. 调用编译器时通过传递一个或多个`M${PROTO_FILE}=${GO_IMPORT_PATH}`标志的方式在命令行上指定Go导入路径。用法示例：

   ```shell
   protoc --proto_path=src \
     --go_opt=Mprotos/buzz.proto=example.com/project/protos/fizz \
     --go_opt=Mprotos/bar.proto=example.com/project/protos/foo \
     protos/buzz.proto protos/bar.proto
   ```




## 生成.go代码的一些说明

```
message Foo {}
```

1. 协议缓冲区编译器生成一个名为的结构`Foo`。一个 `*Foo`实现 [`proto.Message`](https://pkg.go.dev/google.golang.org/protobuf/proto?tab=doc#Message) 接口。

2. 该`proto.Message`接口定义了一个`ProtoReflect`方法。此方法返回 [`protoreflect.Message`](https://pkg.go.dev/google.golang.org/protobuf/reflect/protoreflect?tab=doc#Message) ，提供消息的基于反射的视图。

3. 该`optimize_for`选项不会影响Go代码生成器的输出。

4. 嵌套类型上传两个结构体

   ```protobuf
   message Foo {
     message Bar {
     }
   }
   ```

   编译器会生成两个结构：`Foo`和 `Foo_Bar`。

5. 字段

   - 第一个字母大写用于出口。如果第一个字符是下划线，则将其删除并以大写字母X开头。

   - 如果内部下划线后跟小写字母，则下划线将被删除，并且以下字母大写。

     因此，原始字段`foo_bar_baz`变为 `FooBarBaz`Go，`_my_field_name_2`变为 `XMyFieldName_2`。



参考搭建grpc的例子：

https://grpc.io/docs/languages/go/quickstart/