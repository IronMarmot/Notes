# gRPC

## 介绍
gRPC，Google开发的套远程调用框架，远程调用框架顾名思义就是可以像调用本地服务一样调用远程服务，gRPC的使用类似于之前使用过的thrift，使用时都需要通过定义文件来进行服务定义，然后服务端和客户端分别引用该文件来生成对应的嵌入式代码。
gRPC使用的定义文件是.proto文件，即Protobuf文件，该形式的json文件不同于普通的键值对序列化方式，因而其效率较高。
普通键值对json：序列化时不关注先后顺序，通过键值来确定值；
Proto文件的序列化方式，是通过数据类型以及字段对应的序号来序列号，因此，其序列化后的结果，占用内存小，且效率高。

## .net core下使用

### 包引用
服务端
grpc. aspnet
客户端
grpc. net
grpc. client
grpc. clientFactory
grpc. tools

### proto文件定义
首先，先来定义proto文件，我们这里来实现一个创建订单的服务，具体的内容如下

### 服务端创建
我这里使用的是vs2019，新建项目是，选择grpc远程调用模板即可，创建后的项目包含一个Protos和Services文件夹，Protos文件下包含一个默认的greet.proto文件，Services文件夹下包含一个greetService类。
protos文件夹是存放服务定义的，services文件夹是存放对应的服务实现的。
默认情况下，项目生成后，proto文件生成的cs文件在项目根目录下的obj文件内，会生成2个.cs文件，该部分代码不会显示在项目结构中，但编译时会编译到项目中。

这里需要提醒一下的时，如何让proto文件能自动生成.cs文件，刚开始的时候折腾的够呛，因此觉得应该在靠前的地方给出提示。
其实，要让proto文件能自动生成.cs文件，只需要把proto文件添加到项目的引用中就可。因此，在刚开始时，可以通过手动添加的方式来加入自定义的proto文件，但是并不建议该种方式，
因为dotnet提供了grpc操作的工具，
首先，我们通过 dotnet tool install dotnet-grpc -g来安装grpc工具到全局
随后，可以使用dotnet grpc add-file xx.proto命令来将xx.proto文件添加到对应的工程，这里需要注意执行时要切换到对应的工程目录下。
一下是几个常用命令
dotnet grpc add-file ..
dotnet grpc add-url ..
dotnet grpc remove ..
dotnet grpc refresh

好了，我们接着来说服务定义。
服务定义需要引用proto中定义的service名称.名称Base(比如，service定义为Order，则该定义需要继承Order.OrderBase，如果不存在，则需要引入命名空间)
之后，即可重写其中的方法。

服务定义好之后，我们需要将服务发布出去，这里包括两步，一是在ConfigureService里通过services.AddGrpc()来注册grpc，一是通过终结点endpoints将rpc服务发布出去，如endPoints.MapGrpcService<xxx>();
到这里，服务端就定义完成了。

### 客户端创建
服务端定义好，并将服务发布后，我们就该来创建相应的客户端了。我们这里通过创建一个空的aspnetcore程序来演示。
项目创建完成后，我们通过dotnet grpc add-file 来将服务端的proto文件引入到客户端工程，同时，为了完成调用，我们需要在startup的configureService方法中来注册GrpcClient。
如services.AddGrpcClient<Order.OrderClient>(options => { options.Address = new Uri("https://localhost:5001"); });
同时，为了实现调用服务端的目的，我们在终结点内来实现方法调用，具体如下
endpoints.MapGet("/", async context =>
{
    Order.OrderClient orderClient = context.RequestServices.GetService<Order.OrderClient>();
    try
    {
        var r = orderClient.CreateOrder(new RequestParmas() { CustomerId = 1, ProductName = "2", Price = 3.00 });
    }
    catch (Exception ex)
    {

        throw;
    }
    await context.Response.WriteAsync("Hello World!");
});

grpc基于http/2加密，因此需要用到证书
客户端在调用时，需要配置服务端的地址，可通过http  https  http/2来定义，可通过http的方式来进行非加密的http/2调用，需要通过语句
appcontext. setswitch来进行设置
内网间rpc调用可通过这种方式

另外，服务端可使用自定义证书，同时，通过httpclientfactory. validatecertified方法来绕过证书验证
不设置的话，网页会提示非安全链接，并且客户端会抛异常