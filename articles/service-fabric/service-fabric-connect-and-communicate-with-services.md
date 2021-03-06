---
title: 在 Azure Service Fabric 中与服务建立连接和通信 | Microsoft 文档
description: 了解如何在 Service Fabric 中解析服务、建立连接以及与之通信。
services: service-fabric
documentationcenter: .net
author: vturecek
manager: timlt
editor: msfussell
ms.assetid: 7d1052ec-2c9f-443d-8b99-b75c97266e6c
ms.service: service-fabric
ms.devlang: dotnet
ms.topic: conceptual
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 11/01/2017
ms.author: vturecek
ms.openlocfilehash: 2b6fd2373a9cd0b376a6c8729d5952c5fc48ddf8
ms.sourcegitcommit: eb75f177fc59d90b1b667afcfe64ac51936e2638
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/16/2018
---
# <a name="connect-and-communicate-with-services-in-service-fabric"></a>在 Service Fabric 中与服务建立连接和通信
在 Service Fabric 中，服务在 Service Fabric 群集（通常分布在多个 VM 间）中的某个位置运行。 它可以从一个位置移动到另一个位置（由服务所有者移动或由 Service Fabric 自动移动）。 服务不以静态方式绑定到特定计算机或地址。

Service Fabric 应用程序通常由许多不同服务组成，其中每个服务执行专门任务。 这些服务可能会相互进行通信以形成一个完整功能，如呈现 Web 应用程序的不同部分。 其中也有连接到服务并与之通信的客户端应用程序。 本文档介绍如何在 Service Fabric 中设置与服务进行的通信以及服务之间的通信。

此 Microsoft Virtual Academy 视频还介绍了服务通信：<center><a target="_blank" href="https://mva.microsoft.com/en-US/training-courses/building-microservices-applications-on-azure-service-fabric-16747?l=iYFCk76yC_6706218965">  
<img src="./media/service-fabric-connect-and-communicate-with-services/CommunicationVid.png" WIDTH="360" HEIGHT="244">  
</a></center>

## <a name="bring-your-own-protocol"></a>自带协议
Service Fabric 可帮助管理服务的生命周期，但是它不会制定有关服务执行的操作的决策。 这包括通信。 服务由 Service Fabric 打开时，服务可以使用所需的任何协议或通信堆栈为传入请求设置终结点。 服务会使用任何寻址方案（如 URI）来侦听通常的 **IP:端口**地址。 多个服务实例或副本可能会共享主机进程，在这种情况下，它们需要使用不同端口，或使用端口共享机制（例如 Windows 中的 http.sys 内核驱动程序）。 在任一情况下，主机进程中的每个服务实例或副本都必须可唯一寻址。

![服务终结点][1]

## <a name="service-discovery-and-resolution"></a>服务发现和解析
在分布式系统中，服务可能随时间推移从一台计算机移动到另一台计算机。 发生这种情况可能是由于各种原因，包括资源平衡、升级、故障转移或扩大。这意味着服务终结点地址会在服务移动到具有不同 IP 地址的节点时发生更改，并且可能在不同端口上打开（如果服务使用动态选择的端口）。

![服务分发][7]

Service Fabric 提供一种服务发现和解析服务，称为“命名服务”。 命名服务维护一个表，它将命名服务实例映射到它们所侦听的终结点地址。 Service Fabric 中的所有命名服务实例都具有表示为 URI 的唯一名称，例如 `"fabric:/MyApplication/MyService"`。 服务的名称在服务的生存期内不会更改，只有终结点地址才能在服务移动时发生更改。 这与 URL 保持不变但 IP 地址可能会更改的网站类似。 与 Web 上的 DNS（将网站 URL 解析为 IP 地址）类似，Service Fabric 具有一个注册机构，它将服务名称映射到其终结点地址。

![服务终结点][2]

解析服务以及连接到服务涉及循环运行的以下步骤：

* **解析**：从命名服务获取服务发布的终结点。
* **连接**：通过服务在该终结点上使用的任何协议连接到服务。
* **重试**：连接尝试可能会由于许多原因而失败（例如，如果服务自上次解析终结点地址以来已移动）。 在这种情况下，前面的解析和连接步骤需要重试，并且此循环会重复执行，直到连接成功。

## <a name="connecting-to-other-services"></a>连接到其他服务
在群集内相互连接的服务通常可以直接访问其他服务的终结点，因为群集中的节点处于相同的本地网络上。 为了更轻松地在服务之间连接，Service Fabric 提供使用命名服务的其他服务。 DNS 服务和反向代理服务。


### <a name="dns-service"></a>DNS 服务
由于许多服务（特别是容器化服务）可以拥有一个现有的 URL 名称，能够使用标准 DNS 协议（而不是命名服务协议）解析这些名称会十分方便，尤其是在应用程序“提升和转移”方案中。 这正是 DNS 服务能够发挥作用的地方。 借助 DNS 服务，用户能够将 DNS 名称映射到服务名称，进而解析终结点 IP 地址。 

如下图所示，Service Fabric 群集中运行的 DNS 服务会将 DNS 名称映射到服务名称，服务名称随后会被命名服务解析，以返回要连接的终结点地址。 在创建时提供服务的 DNS 名称。 

![服务终结点][9]

有关如何使用 DNS 服务的更多详细信息，请参阅 [Azure Service Fabric 中的 DNS 服务](service-fabric-dnsservice.md)一文。

### <a name="reverse-proxy-service"></a>反向代理服务
反向代理处理群集中的服务，群集公开包括 HTTPS 在内的 HTTP 终结点。 反向代理具有特定的 URI 格式，能够极大地简化调用其他服务及其方法的操作，它能够处理一个服务使用命名服务与其他服务进行通信所需完成的解析、连接、重试步骤。 换而言之，它会在调用其他服务时对用户隐藏命名服务，并让这一操作和调用 URL 一样简单。

![服务终结点][10]

有关如何使用反向代理服务的更多详细信息，请参阅 [Azure Service Fabric 中的反向代理](service-fabric-reverseproxy.md)一文。

## <a name="connections-from-external-clients"></a>来自外部客户端的连接
在群集内相互连接的服务通常可以直接访问其他服务的终结点，因为群集中的节点处于相同的本地网络上。 但是在某些环境中，群集可能位于通过一组有限端口对外部传入流量进行路由的负载均衡器之后。 在这些情况下，服务仍可以使用命名服务相互通信和解析地址，但必须执行额外步骤才能允许外部客户端连接到服务。

## <a name="service-fabric-in-azure"></a>Azure 中的 Service Fabric
Azure 中的 Service Fabric 群集位于 Azure 负载均衡器之后。 发送到群集的所有外部流量都必须穿过该负载均衡器。 该负载均衡器会自动在给定端口上将入站流量转发到打开了相同端口的随机*节点*。 Azure 负载均衡器只了解*节点*上打开的端口，它不了解各个*服务*打开的端口。

![Azure 负载均衡器和 Service Fabric 拓扑][3]

例如，若要在端口 **80** 上接受外部流量，必须配置以下内容：

1. 编写侦听端口 80 的服务。 在服务的 ServiceManifest.xml 中配置端口 80，并在服务中打开一个侦听器，例如自承载的 Web 服务器。

    ```xml
    <Resources>
        <Endpoints>
            <Endpoint Name="WebEndpoint" Protocol="http" Port="80" />
        </Endpoints>
    </Resources>
    ```
    ```csharp
        class HttpCommunicationListener : ICommunicationListener
        {
            ...

            public Task<string> OpenAsync(CancellationToken cancellationToken)
            {
                EndpointResourceDescription endpoint =
                    serviceContext.CodePackageActivationContext.GetEndpoint("WebEndpoint");

                string uriPrefix = $"{endpoint.Protocol}://+:{endpoint.Port}/myapp/";

                this.httpListener = new HttpListener();
                this.httpListener.Prefixes.Add(uriPrefix);
                this.httpListener.Start();

                string publishUri = uriPrefix.Replace("+", FabricRuntime.GetNodeContext().IPAddressOrFQDN);
                return Task.FromResult(publishUri);
            }

            ...
        }

        class WebService : StatelessService
        {
            ...

            protected override IEnumerable<ServiceInstanceListener> CreateServiceInstanceListeners()
            {
                return new[] { new ServiceInstanceListener(context => new HttpCommunicationListener(context))};
            }

            ...
        }
    ```
    ```java
        class HttpCommunicationlistener implements CommunicationListener {
            ...

            @Override
            public CompletableFuture<String> openAsync(CancellationToken arg0) {
                EndpointResourceDescription endpoint =
                    this.serviceContext.getCodePackageActivationContext().getEndpoint("WebEndpoint");
                try {
                    HttpServer server = com.sun.net.httpserver.HttpServer.create(new InetSocketAddress(endpoint.getPort()), 0);
                    server.start();

                    String publishUri = String.format("http://%s:%d/",
                        this.serviceContext.getNodeContext().getIpAddressOrFQDN(), endpoint.getPort());
                    return CompletableFuture.completedFuture(publishUri);
                } catch (IOException e) {
                    throw new RuntimeException(e);
                }
            }

            ...
        }

        class WebService extends StatelessService {
            ...

            @Override
            protected List<ServiceInstanceListener> createServiceInstanceListeners() {
                <ServiceInstanceListener> listeners = new ArrayList<ServiceInstanceListener>();
                listeners.add(new ServiceInstanceListener((context) -> new HttpCommunicationlistener(context)));
                return listeners;       
            }

            ...
        }
    ```
2. 在 Azure 中创建 Service Fabric 群集，并将端口 **80** 指定为将承载服务的节点类型的自定义终结点端口。 如果具有多种节点类型，则可以对服务设置*放置约束*，以确保它只在打开了自定义终结点端口的节点类型上运行。

    ![在节点类型上打开端口][4]
3. 创建了群集之后，在群集的资源组中配置 Azure 负载均衡器以在端口 80 上转发流量。 通过 Azure 门户创建群集时，会为每个已配置的自定义终结点端口自动设置此项。

    ![在 Azure 负载均衡器中转发流量][5]
4. Azure 负载均衡器使用探测来确定是否要将流量发送到特定节点。 探测会定期检查每个节点上的终结点以确定节点是否正在进行响应。 如果探测未能在配置的次数之后收到响应，则负载均衡器会停止将流量发送到该节点。 通过 Azure 门户创建群集时，会为每个已配置的自定义终结点端口自动设置探测。

    ![在 Azure 负载均衡器中转发流量][8]

请务必记住，Azure 负载均衡器和探测只了解*节点*，而不了解在节点上运行的*服务*。 Azure 负载均衡器始终将流量发送到响应探测的节点，因此必须格外小心以确保服务在能够响应探测的节点上可用。

## <a name="reliable-services-built-in-communication-api-options"></a>Reliable Services：内置通信 API 选项
Reliable Services 框架附带几个预建的通信选项。 可以根据所选的编程模型、通信框架和编写服务时使用的编程语言来决定哪个选项最适合自己。

* **无特定协议：** 如果没有选择特定的通信框架，但想要快速启动并运行某个程序，则理想之选是[服务远程处理](service-fabric-reliable-services-communication-remoting.md)，此项允许对 Reliable Services 和 Reliable Actors 进行强类型远程过程调用。 这是服务通信入门最简单、最快捷的方法。 服务远程处理可处理服务地址的解析、连接、重试和错误处理。 这适用于 C# 和 Java 应用程序。
* **HTTP**：对于与语言无关的通信，HTTP 为行业标准选择提供了可在许多不同语言（Service Fabric 全都支持）中使用的工具和 HTTP 服务器。 服务可以使用提供的任何 HTTP 堆栈，包括适用于 C# 应用程序的 [ASP.NET Web API](service-fabric-reliable-services-communication-webapi.md)。 以 C# 编写的客户端可以利用 `ICommunicationClient` 和 `ServicePartitionClient` 类（而 Java 则可以使用 `CommunicationClient` 和 `FabricServicePartitionClient` 类）来[进行服务解析、HTTP 连接和重试循环](service-fabric-reliable-services-communication.md)。
* **WCF**：如果具备将 WCF 用作通信框架的现有代码，则可将 `WcfCommunicationListener` 用于服务器端，并将 `WcfCommunicationClient` 和 `ServicePartitionClient` 类用于客户端。 但是，这仅适用于基于 Windows 的群集上的 C# 应用程序。 有关更多详细信息，请参阅这篇有关[通信堆栈的基于 WCF 的实现](service-fabric-reliable-services-communication-wcf.md)的文章。

## <a name="using-custom-protocols-and-other-communication-frameworks"></a>使用自定义协议和其他通信框架
服务可以使用任何协议或框架进行通信，无论它是 TCP 套接字上的自定义二进制协议，还是通过 [Azure 事件中心](https://azure.microsoft.com/services/event-hubs/)或 [Azure IoT 中心](https://azure.microsoft.com/services/iot-hub/)实现的流式处理事件。 Service Fabric 提供了通信 API，可以将通信堆栈插入其中，同时将用于发现和连接的所有工作与你分离。 有关更多详细信息，请参阅这篇有关 [Reliable Service 通信模型](service-fabric-reliable-services-communication.md)的文章。

## <a name="next-steps"></a>后续步骤
了解有关 [Reliable Services 通信模型](service-fabric-reliable-services-communication.md)中可用的概念和 API 的详细信息，并快速开始使用[服务远程处理](service-fabric-reliable-services-communication-remoting.md)或深入了解如何使用[具有 OWIN 自承载的 Web API](service-fabric-reliable-services-communication-webapi.md) 编写通信侦听器。

[1]: ./media/service-fabric-connect-and-communicate-with-services/serviceendpoints.png
[2]: ./media/service-fabric-connect-and-communicate-with-services/namingservice.png
[3]: ./media/service-fabric-connect-and-communicate-with-services/loadbalancertopology.png
[4]: ./media/service-fabric-connect-and-communicate-with-services/nodeport.png
[5]: ./media/service-fabric-connect-and-communicate-with-services/loadbalancerport.png
[7]: ./media/service-fabric-connect-and-communicate-with-services/distributedservices.png
[8]: ./media/service-fabric-connect-and-communicate-with-services/loadbalancerprobe.png
[9]: ./media/service-fabric-connect-and-communicate-with-services/dns.png
[10]: ./media/service-fabric-reverseproxy/internal-communication.png
