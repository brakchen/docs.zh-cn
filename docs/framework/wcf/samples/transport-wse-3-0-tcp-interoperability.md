---
title: 传输：WSE 3.0 TCP 互操作性
ms.date: 03/30/2017
ms.assetid: 5f7c3708-acad-4eb3-acb9-d232c77d1486
ms.openlocfilehash: f799f3b6968f31472acc7752846bab34351648db
ms.sourcegitcommit: 7980a91f90ae5eca859db7e6bfa03e23e76a1a50
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/14/2020
ms.locfileid: "81278894"
---
# <a name="transport-wse-30-tcp-interoperability"></a>传输：WSE 3.0 TCP 互操作性
WSE 3.0 TCP 互操作性传输示例演示如何实现 TCP 双工会话作为自定义 Windows 通信基础 （WCF） 传输。 还演示如何通过网络，使用通道层的扩展性与已经过部署的现有系统进行交互。 以下步骤演示如何构建此自定义 WCF 传输：  
  
1. 从 TCP 套接字开始，创建 <xref:System.ServiceModel.Channels.IDuplexSessionChannel> 的客户端和服务器实现以使用 DIME 组帧来描述消息边界。  
  
2. 创建一个连接到 WSE TCP 服务并通过客户端 <xref:System.ServiceModel.Channels.IDuplexSessionChannel> 发送组帧消息的通道工厂。  
  
3. 创建一个通道侦听器以接受传入的 TCP 连接并生成相应的通道。  
  
4. 请确保将特定于网络的任何异常正常化为 <xref:System.ServiceModel.CommunicationException> 的相应派生类。  
  
5. 添加一个用来向通道堆栈中添加自定义传输的绑定元素。 有关详细信息，请参阅 [添加绑定元素]。  
  
## <a name="creating-iduplexsessionchannel"></a>创建 IDuplexSessionChannel  
 编写 WSE 3.0 TCP 互操作性传输的第一步是在 <xref:System.ServiceModel.Channels.IDuplexSessionChannel> 的顶部创建 <xref:System.Net.Sockets.Socket> 的实现。 `WseTcpDuplexSessionChannel` 派生自 <xref:System.ServiceModel.Channels.ChannelBase>。 消息的发送逻辑主要由以下两个部分组成：(1) 将消息编码为字节；(2) 对这些字节进行组帧并通过网络发送它们。  
  
 `ArraySegment<byte> encodedBytes = EncodeMessage(message);`  
  
 `WriteData(encodedBytes);`  
  
 另外，设置了一个锁，以保证在调用 Send() 时可以按顺序保留 IDuplexSessionChannel，以便对基础套接字的调用能够正确地同步。  
  
 `WseTcpDuplexSessionChannel` 使用 <xref:System.ServiceModel.Channels.MessageEncoder> 在 <xref:System.ServiceModel.Channels.Message> 和 byte[] 之间相互转换。 由于 `WseTcpDuplexSessionChannel` 为传输，因此它还负责应用通道所配置的远程地址。 `EncodeMessage` 封装此转换的逻辑。  
  
 `this.RemoteAddress.ApplyTo(message);`  
  
 `return encoder.WriteMessage(message, maxBufferSize, bufferManager);`  
  
 一旦将 <xref:System.ServiceModel.Channels.Message> 编码为字节，就必须通过线路传输它。 这要求系统定义消息边界。 WSE 3.0 使用[DIME](https://docs.microsoft.com/archive/msdn-magazine/2002/december/sending-files-attachments-and-soap-messages-via-dime)版本作为其帧协议。 `WriteData` 封装框架逻辑以便将 byte[] 包装到一组 DIME 记录中。  
  
 接收消息的逻辑类似。 主要的复杂性是处理这样一个事实，即套接字读取可以返回的字节数比请求的要少。 若要接收消息，`WseTcpDuplexSessionChannel` 读取网络中的字节，对 DIME 组帧进行解码，然后使用 <xref:System.ServiceModel.Channels.MessageEncoder> 将 byte[] 转换为 <xref:System.ServiceModel.Channels.Message>。  
  
 基 `WseTcpDuplexSessionChannel` 假设它接收连接的套接字。 这个基类处理套接字的关闭。 可通过三个位置与套接字关闭进行交互：  
  
- OnAbort - 以非正常方式关闭套接字（硬关闭）。  
  
- On[Begin]Close - 正常关闭套接字（软关闭）。  
  
- 会话。关闭输出会话 -- 关闭出站数据流（半关闭）。  
  
## <a name="channel-factory"></a>通道工厂  
 编写 TCP 传输的下一步是为客户端通道创建 <xref:System.ServiceModel.Channels.IChannelFactory> 的实现。  
  
- `WseTcpChannelFactory`派生自<xref:System.ServiceModel.Channels.ChannelFactoryBase>\<i双工会话通道>。 它是一个用来重写 `OnCreateChannel` 以生成客户端通道的工厂。  
  
 `protected override IDuplexSessionChannel OnCreateChannel(EndpointAddress remoteAddress, Uri via)`  
  
 `{`  
  
 `return new ClientWseTcpDuplexSessionChannel(encoderFactory, bufferManager, remoteAddress, via, this);`  
  
 `}`  
  
- `ClientWseTcpDuplexSessionChannel`将逻辑添加到基中`WseTcpDuplexSessionChannel`，以便及时`channel.Open`连接到 TCP 服务器。 首先，主机名解析为 IP 地址，如下面的代码所示。  
  
 `hostEntry = Dns.GetHostEntry(Via.Host);`  
  
- 然后，主机名连接到循环中的第一个可用 IP 地址，如下面的代码所示。  
  
 `IPAddress address = hostEntry.AddressList[i];`  
  
 `socket = new Socket(address.AddressFamily, SocketType.Stream, ProtocolType.Tcp);`  
  
 `socket.Connect(new IPEndPoint(address, port));`  
  
- 包装了所有特定于域的异常（如 `SocketException` 中的 <xref:System.ServiceModel.CommunicationException>），作为通道协定的一部分。  
  
## <a name="channel-listener"></a>通道侦听器  
 编写 TCP 传输的下一步是创建用来接受服务器通道的 <xref:System.ServiceModel.Channels.IChannelListener> 实现。  
  
- `WseTcpChannelListener`派生自<xref:System.ServiceModel.Channels.ChannelListenerBase>\<iDuplexSessionChannel>并覆盖 On[开始]打开和打开[开始]关闭以控制其侦听套接字的生存期。 在 OnOpen 中，需要创建一个用来侦听 IP_ANY 的套接字。 在更高级的实现中，可以再创建一个同时侦听 IPv6 的套接字。 这些高级实现还允许在主机名中指定 IP 地址。  
  
 `IPEndPoint localEndpoint = new IPEndPoint(IPAddress.Any, uri.Port);`  
  
 `this.listenSocket = new Socket(localEndpoint.AddressFamily, SocketType.Stream, ProtocolType.Tcp);`  
  
 `this.listenSocket.Bind(localEndpoint);`  
  
 `this.listenSocket.Listen(10);`  
  
 在接受了新套接字之后，将用这个套接字来初始化服务器通道。 所有的输入和输出都已经在该基类中实现，因此该通道负责初始化此套接字。  
  
## <a name="adding-a-binding-element"></a>添加绑定元素  
 现在已经生成了工厂和通道，必须通过绑定将它们向 ServiceModel 运行库公开。 绑定是指绑定元素的集合，该集合表示与服务地址相关联的通信堆栈。 该堆栈中的每个元素都由一个绑定元素来表示。  
  
 在下面的示例中，绑定元素为 `WseTcpTransportBindingElement`，它派生自 <xref:System.ServiceModel.Channels.TransportBindingElement>。 它支持 <xref:System.ServiceModel.Channels.IDuplexSessionChannel> 并重写下列方法以生成与绑定相关联的工厂。  
  
 `public IChannelFactory<TChannel> BuildChannelFactory<TChannel>(BindingContext context)`  
  
 `{`  
  
 `return (IChannelFactory<TChannel>)(object)new WseTcpChannelFactory(this, context);`  
  
 `}`  
  
 `public IChannelListener<TChannel> BuildChannelListener<TChannel>(BindingContext context)`  
  
 `{`  
  
 `return (IChannelListener<TChannel>)(object)new WseTcpChannelListener(this, context);`  
  
 `}`  
  
 它还包含用于克隆 `BindingElement` 并返回我们自己的方案 (wse.tcp) 的成员。  
  
## <a name="the-wse-tcp-test-console"></a>WSE TCP 测试控制台  
 TestCode.cs 中提供了用于使用此示例传输的测试代码。 下面的说明演示如何设置 WSE `TcpSyncStockService` 示例。  
  
 该测试代码创建了一个自定义绑定，该绑定将 MTOM 用作编码并将 `WseTcpTransport` 用作传输。 该测试代码还设置了与 WSE 3.0 一致的 AddressingVersion，如下面的代码所示。  
  
 `CustomBinding binding = new CustomBinding();`  
  
 `MtomMessageEncodingBindingElement mtomBindingElement = new MtomMessageEncodingBindingElement();`  
  
 `mtomBindingElement.MessageVersion = MessageVersion.Soap11WSAddressingAugust2004;`  
  
 `binding.Elements.Add(mtomBindingElement);`  
  
 `binding.Elements.Add(new WseTcpTransportBindingElement());`  
  
 它由两个测试组成，第一个测试使用从 WSE 3.0 WSDL 生成的代码来设置类型化客户端， 第二个测试通过将消息直接发送到通道 API 的顶部，将 WCF 用作客户端和服务器。  
  
 运行此示例时，应生成下面的输出。  
  
 Client：  
  
```console  
Calling soap://stockservice.contoso.com/wse/samples/2003/06/TcpSyncStockService  
  
Symbol: FABRIKAM  
        Name: Fabrikam, Inc.  
        Last Price: 120  
  
Symbol: CONTOSO  
        Name: Contoso Corp.  
        Last Price: 50.07  
Press enter.  
  
Received Action: http://SayHello  
Received Body: to you.  
Hello to you.  
Press enter.  
  
Received Action: http://NotHello  
Received Body: to me.  
Press enter.  
```  
  
 服务器:  
  
```console  
Listening for messages at soap://stockservice.contoso.com/wse/samples/2003/06/TcpSyncStockService  
  
Press any key to exit when done...  
  
Request received.  
Symbols:  
        FABRIKAM  
        CONTOSO  
```  
  
## <a name="set-up-build-and-run-the-sample"></a>设置、生成和运行示例  
  
1. 要运行此示例，必须安装 Microsoft .NET 的[Web 服务增强功能 （WSE） 3.0](https://www.microsoft.com/download/details.aspx?id=14089)和 WSE`TcpSyncStockService`示例。
  
> [!NOTE]
> 由于 Windows Server 2008 不支持 WSE 3.0，因此无法在`TcpSyncStockService`该操作系统上安装或运行该示例。  
  
1. 安装了 `TcpSyncStockService` 示例后，请执行下列操作：  
  
    1. 打开`TcpSyncStockService`视觉工作室。 （TcpSyncStock服务示例与 WSE 3.0 一起安装。 它不是此示例代码的一部分。  
  
    2. 将 StockService 项目设置为启动项目。  
  
    3. 在 StockService 项目中打开 StockService.cs，并注释掉 `StockService` 类的 [Policy] 属性。 这会禁用该示例中的安全性。 虽然 WCF 可以与 WSE 3.0 安全终结点互操作，但禁用安全性可使此示例专注于自定义 TCP 传输。  
  
    4. 按 F5 启动 `TcpSyncStockService`。 该服务将在一个新的控制台窗口中启动。  
  
    5. 在 Visual Studio 中打开这个 TCP 传输示例。  
  
    6. 更新 TestCode.cs 中的“hostname”变量，使其与运行 `TcpSyncStockService` 的计算机的名称相匹配。  
  
    7. 按 F5 启动 TCP 传输示例。  
  
    8. TCP 传输测试客户端将在一个新控制台中启动。 客户端从服务请求股票报价，然后将结果显示在其控制台窗口中。  
