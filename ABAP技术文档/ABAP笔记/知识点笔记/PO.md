介绍：SAP PO是基于SAP Net weaver平台的中间件产品，为企业提供一套支持SAP系统间、SAP系统与Non-Sap系统间以及Non-SAP系统间进行数据交换和流程整合的平台，支持同步和异步的数据交互方式，帮助企业及其IT组织实现大部分集成需求。

在大型企业中实施SAP ERP时，发现并非所有部分都可以纳入SAP ERP。许多业务部门可能有自己的专有工具，这些工具非常复杂，可能无法替换。它们与 SAP 系统并行运行。它们被称为遗留系统。然后，有必要在SAP系统和这种预先存在的非SAP系统之间进行集成。这就是SAP PI发挥作用的地方。

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240617163054869.png" alt="image-20240617163054869" style="zoom:50%;" />

当 SAP PI 从双堆栈移动到单堆栈时，这两个适配器就成为 Java 堆栈的一部分。修改后的适配器引擎称为高级适配器引擎，这两个适配器分别称为IDOC_AAE适配器和HTTP_AAE适配器。



<img src="../../../../Library/Application%20Support/typora-user-images/image-20240617163542831.png" alt="image-20240617163542831" style="zoom:50%;" />

主页包含指向以下 4 个工作区的超链接
1、企业服务存储库 （ESR）

2、集成目录 （ID）

3、系统环境 （SL）

4、配置和监控 （CM）

每个超链接将打开一个应用程序。这四个都是 Java 应用程序。ESR 和 ID 是摆动应用。它们是从基于 JNLP 的浏览器启动的。因此，这是第一次下载整个库文件时需要更多时间。但从第二次开始，启动所需的时间就更少了。SL 和 CM 是纯 Web 应用程序，在浏览器上运行。



ESR

在这里，我们设计和创建用于制作集成方案的对象。PI 中的数据流将如下所示

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240617163854154.png" alt="image-20240617163854154" style="zoom:50%;" />

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240617164207304.png" alt="image-20240617164207304" style="zoom:50%;" />

PI使用集成存储库为发送方和接收方系统设计消息结构，并使用相应的消息结构开发接口消息，这些消息结构充当与外界的交互点。数据类型和消息类型用于简化和模块化复杂接口的设计。

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240617164251845.png" alt="image-20240617164251845" style="zoom:50%;" />

当两个结构不同时，操作映射允许将源结构转换为目标结构。但是，如果源结构和目标结构相同，则可以省略操作映射。与服务接口类似，消息映射用于简化和模块化复杂操作映射的设计。消息映射可以通过 4 种方式实现





**Integration Directory 集成目录**

Communication channel determines the inbound and outbound processing of messages. The messages are converted from native format to soap-xml specific message format and vice-versa through the adapter. Generally there are two types of communication channel in a scenario
通信通道决定了消息的入站和出站处理。消息通过适配器从本机格式转换为特定于 soap-xml 的消息格式，反之亦然。 通常，一个场景中有两种类型的通信通道