## ESR配置要点

配置组件时，有两个DT_REQ,分别是 获取数据 和 再次返回处理消息

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240820082911428.png" alt="image-20240820082911428" style="zoom:50%;" />

配置服务接口SI时，作为数据发送方的配置成Asynchronous异步，且只有Req

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240820085840704.png" alt="image-20240820085840704" style="zoom:33%;" />

作为接收方的SI 同理配置成Asynchronous异步，且只有REQ和Fault

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240820090025246.png" alt="image-20240820090025246" style="zoom:33%;" />

OM 操作映射时 内部只有REQ的映射

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240820090454585.png" alt="image-20240820090454585" style="zoom:33%;" />





## ID配置要点

配置CC 通道时，作为发送信息方 服务质量为Exactly Once

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240820090758950.png" alt="image-20240820090758950" style="zoom:33%;" />