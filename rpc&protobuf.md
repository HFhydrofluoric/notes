# rpc应用和protobuf的学习

## protobuf（protocol buffer）
1. 简介：protobuf是谷歌的一种数据协议，相比较json或者xml来说，protobuf的二进制格式存储空间要小很多，且与语言耦合性小，c、java、python等都有相应的编译工具，互相兼容。在服务器间数据传输十分常用，在分布式系统中尤为适用频繁。
2. 安装：[在github上](https://github.com/google/protobuf/)上找到相对应的语言的版本并下载。解压 tar xzf 包，执行命令进行安装。</br>```./configure和make && make install``` </br>本文以python为例，要安装python的依赖则进入python文件夹进行以下的命令的安装</br>```python setup.py build和python setup.py test```</br>在.bash_profile文件中写入路径.使用```protoc --version```来查看版本号。
3. 使用：先编写test.proto文档,required为必填数据，optional为可选数据。package类似命名空间。syntax是编译器要求加上的版本说明</br>
    
``` 
    syntax = "proto2";
    package test;
    message TestMessage
    {
        required int32 m1 = 1;
        optional string m2 = 2;
    }

``` 
> 然后使用对应的编译器进行编译，编译语句
`protoc --proto_path=「proto文件所在路径」 --  python_out=「生成路径」 「生成文件名称（包含路径）」`    
就可以生成【文件名_pb2.py】文件了，可以直接在python中调用。其中包含两个数据转化的函数，SerializeToString将数据转化  为二进制，ParseFromString方法将二进制转化为对应数据。protobuf有部分rpc的实现，关于编码和解码部分则需要我们手动实现。rpc的proto文件编写例子如下：

```  
  syntax = "proto2";
  package test;
  message TestMessage
  {
        required int32 m1 = 1;
  }
  service Client
  {
        rpc respond(TestMessage) returns();
  }
  
```
**这部分详细可以看官网或者他人博客。**
## rpc（Remote Procedure Call Protocol）
1. 简介：rpc是远程过程调用的意思，指利用网络在一端应用调用另一端应用的协议。我们常见的网络应用都算做rpc应用。rpc是一类应用的统称协议，可以基于不同的网络协议。如http、tcp、udp等。这里我们需要实现基于protobuf的rpc应用，网络使用socket进行编写。
2. 架构：protobuf的rpc的基本架构为![rpc架构](./rpc_base.jpg)
根据维基百科的原文其中1.6为通过stub调用server服务，这部分在编译proto后已经生成，2.5的数据序列化和反序列化已经由protobuf提供了方法，我们需要做的是将数据组合成我们想要的格式。3.4两步完全需要我们自己实现，这样做是为了适应各种网络协议。

```
(1). The client calls the client stub. The call is a local procedure call, with parameters pushed on to the stack in the normal way.
(2). The client stub packs the parameters into a message and makes a system call to send the message. Packing the parameters is called marshalling.
(3). The client's local operating system sends the message from the client machine to the server machine.
(4). The local operating system on the server machine passes the incoming packets to the server stub.
(5). The server stub unpacks the parameters from the message. Unpacking the parameters is called unmarshalling.
(6). Finally, the server stub calls the server procedure. The reply traces the same steps in the reverse direction.

```
3. 分析：在仔细对比了两份他人写的教程后，我总结了python的rpc基本模块为四个</br>

```
(1). 编译后的xxx_pb2.py，这个py文件中包含了service和service_stub的实现.
(2). 继承RpcChannel实现自己的channel，google protobuf的rpc提供的channel的callMethod没有实现内容，需要自己手动实现encode和send data， 另外decode也需要自己实现
(3). 继承asyncore.dispatcher实现通信层，这里使用了异步的方式，包括需要继承asyncore实现writable和handle_write和handle_read等函数，在client部分需要建立socket并连接，在server端需要建立socket并监听端口和listen以及accept函数，在asyncore中是通过select的方式（在windows和linux和macOS中通过select，在linux中还有poll和epoll两种方式） 
(4). 继承(1)中实现的两个生成类并具体实现需要的函数功能
```
4. 总结：目前对于此系统的理解就到这里了，之后再有机会深入研究c++版本的muduo来进行学习。
