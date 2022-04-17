---
title: 2019年，你需要Thrift这样一个RPC框架来拯救一下传统HTTP接口了
date: 2019-03-15 10:50:00
tags: [Thrift,RPC]
---

​	目前市面上类似Django的drf框架基于json的http接口解决方案大行其道，人们也热衷于在接口不多、系统与系统交互较少的情况下使用它，http接口的优点就是简单、直接、开发方便，门槛低，利用现成的http协议进行传输。



​	但是事情往往有两面，如果是一个大型的网站，内部子系统较多、接口非常多的情况下，RPC框架的好处就显示出来了，首先就是长链接，不必每次通信都要像http 一样去3次握手4次挥手，减少了网络开销；其次就是RPC框架一般都有注册中心，有丰富的监控管理；发布、下线接口、动态扩展等，对调用方来说是无感知、统一化的操作。第三个来说就是安全性。最后就是最近流行的服务化架构、服务化治理，RPC框架是一个强力的支撑。



​	论复杂度，RPC框架肯定是高于简单的HTTP接口的。但毋庸置疑，HTTP接口由于受限于HTTP协议，需要带HTTP请求头，导致传输起来效率或者说安全性不如RPC，目前市面上流行的rpc框架有dubbo/hessian Thrift，阿里开源的dubbo固然还不错，但是本人更倾向于facebook开源的Thrift框架，这款框架在github上好评如潮，这一次我们使用的就是基于Thrift的thriftpy2框架。



​	Thrift是一种接口描述语言和二进制通讯协议，它被用来定义和创建跨语言的服务，这是维基百科的描述。简单来说就是你可以按照Thrift定义语法编写.thrift,然后用Thrift命令行生成各种语言的代码，比如OC、Java、C++、JS，调用这些代码就可以完成客户端与服务器的通信了，不需要自己去写网络请求、数据解析等接口。



​	其实在本人的实际教学工作中主要考虑到这两个优点：



​	RPC。通过简单定义Thrift描述语言文件，使用Thrift -gen命令可以生成多种语言的代码，这些代码包含了网络通信,数据编解码的功能。这就免去了前后台编写这部分繁琐的代码，同时也统一了前后台的实现逻辑。



​	Thrift的二进制数据的编码比json更加紧凑、减少了无用的数据的传输。

​    

​	安装:

```pyth
pip3 install thriftpy2
```



​	首先定义 thrift 通讯文件，无论是server端还是clinet端都是基于这个文件进行通信 pingpong.thrift 

```pyt
service PingPong {
    string ping(),
    string check_login(
        1: string username,
        2: string password
    ),
}
```



​	可以看到我们定义了两个方法，一个有参一个无参，第一个方法用来检测接口是否通信成功，也就是传统的ping命令，第二个方法顾名思义，用户登录



​	然后建立一个thrift_server.py 建立服务端的代码

```pyt
import thriftpy2
pingpong_thrift = thriftpy2.load("pingpong.thrift", module_name="pingpong_thrift")

from thriftpy2.rpc import make_server

class Dispatcher(object):
    
    def ping(self):
        return "pong"

    def check_login(self,username,password):
        print(username,password)
        return '123'

server = make_server(pingpong_thrift.PingPong, Dispatcher(), '127.0.0.1', 6000)
server.serve()
```



​	服务端首先读取通信文件，然后建立起一个服务，监听6000端口，等待客户端请求，实际上服务端的方法也是主要业务逻辑编写的地方。

​	随后建立一个thrift_client.py文件，编写客户端代码



```pyt
import thriftpy2
pingpong_thrift = thriftpy2.load("pingpong.thrift", module_name="pingpong_thrift")

from thriftpy2.rpc import make_client

client = make_client(pingpong_thrift.PingPong, '127.0.0.1', 6000)

print(client.ping())

print(client.check_login('admin','123456'))
```



​	我们看到客户端同样读取通信文件，严格按照通信文件的方法调用方式进行传参调用，获取返回值



​	运行服务器端的服务

```pyt
python3 thrift_server.py
```



​	可以看到服务端和客户端就可以通信了



​	可以说非常简单，这里着重提到的一点是Thrift的数据编解码，我们知道传统http接口通常以json为数据介质，json中一个对象类似于这样的：{"key":"content"},但实际上这个对象只有“content”才是我们真正想要的数据，而“key”这个字符串并不是我们实际需要的，只是为了做一个标记，方便我们查找“content”。而Thrift则可以省去“key”这个多余的字符串。



​	定义thrift的结构里的属性名称实际上在thrift数据二进制编解码是被忽略的（thrift的json编解码未验证），这个名称的作用只是作为生成的OC代码类的属性名称。这也解释了为什么Thrift的二进制编码会比平时使用的json更省流量。同时也说明了只要我们在.thrift文件中定义struct的时候保证struct的属性的顺序不变，即使通信双方使用了各自使用不同的属性名称也不会有问题。



​	随着请求并发量的提高，简单的HTTP肯定达不到预期的效果，Thrift或许才是你寻找的答案。