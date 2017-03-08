---
title: RPC原理 概述
date: 2017-03-02 09:40:18
tags: [rpc]
---
远程过程调用（Remote Procedure Call，缩写为RPC）是一种计算机通信协议。该协议运行运行与一台计算机的程序调用另一台计算机的子程序，而程序员无需额外地为这个交互作用编程。如果涉及的软件采用面向对象编程，那么远程过程调用亦可称作远程调用或远程方法调用。——维基百科

### RPC调用分类
同步调用：客户方等待调用执行完成并返回结果
异步调用：客户方调用后不用等待执行结果返回，但依然可以通过回调通知等方式获取返回结果。 若客户方不关心调用返回结果，则变成单向异步调用，单向调用不用返回结果

### RPC结构

![rpc-20170302.jpg](http://upload-images.jianshu.io/upload_images/1419542-29bfe0b7d38005c1.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/960)
RPC 服务方通过 RpcServer 去导出（export）远程接口方法，而客户方通过 RpcClient 去引入（import）远程接口方法。 客户方像调用本地方法一样去调用远程接口方法，RPC 框架提供接口的代理实现，实际的调用将委托给代理 RpcProxy 。 代理封装调用信息并将调用转交给 RpcInvoker 去实际执行。 在客户端的 RpcInvoker 通过连接器 RpcConnector 去维持与服务端的通道 RpcChannel， 并使用 RpcProtocol 执行协议编码（encode）并将编码后的请求消息通过通道发送给服务方。
RPC 服务端接收器 RpcAcceptor 接收客户端的调用请求，同样使用 RpcProtocol 执行协议解码（decode）。 解码后的调用信息传递给 RpcProcessor 去控制处理调用过程，最后再委托调用给 RpcInvoker 去实际执行并返回调用结果。

### RPC组件职责
1. RpcServer：负责导出（export）远程接口
2. RpcClient：负责导入（import）远程接口的代理实现
3. RpcProxy：远程接口的代理实现
4. RpcInvoker：
    1. 客户端实现方式：负责编码调用信息和发送调用请求到服务方并等待调用结果返回
    2. 服务端实现方式：负责调用服务端接口的具体实现并返回调用结果
5. RpcProtocol：负责协议编/解码
6. RpcConnector：负责维持客户方和服务方的连接通道和发送数据到服务方
7. RpcAcceptor：负责接收客户方请求并返回请求结果
8. RpcProcessor：负责在服务方控制调用过程，包括管理调用线程池、超时时间等
9. RpcChannel：数据传输通道

参考：
http://www.cnblogs.com/metoy/p/4321311.html?utm_source=tuicool&utm_medium=referral
http://www.cnblogs.com/LBSer/p/4853234.html