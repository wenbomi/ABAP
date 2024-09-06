

1、返回到应用程序。异常：com.sap.engine.interfaces.messaging.api.exception.MessageExpiredException: Message b9a0967c-f179-1ede-9ebd-9af3c41a4544(OUTBOUND) expired.

处理：导致该问题的渠道比较多 还不清楚

MP: exception caught with cause com.sap.aii.adapter.rest.ejb.sender.InterfaceExtensionFailedException: Could not parse message content to add interface element

解决：排查通道时发现 这个按钮选择了 去除按钮 解决

![img](https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240718170748472-492440213.png)



2、MP: exception caught with cause com.sap.aii.adapter.rest.ejb.common.exception.HttpCallException: HTTP POST call to http://172.31.2.7/ExternalWebApi/QMStandard/SynQualityStandard not successful. Unsupported Media Type

解决：问题原因内容类型不匹配

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240718171826295-636333900.png" alt="image-20240105170445221" style="zoom:50%;" />

解决过程: 修改 Java mapping文件中的 转换代码

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240718171826830-936414644.png" alt="image-20240105170513556" style="zoom:50%;" />

重新导入Java mapping文件

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240718171827689-648064445.png" alt="image-20240105170533068" style="zoom:50%;" />

完成后格式变更为：

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240718171828471-964374708.png" alt="image-20240105170545467" style="zoom:50%;" />

3、MP: exception caught with cause java.io.IOException: iaik.security.ssl.SSLCertificateException: **Peer certificate rejected by ChainVerifier**

翻译过来给人的感觉是缺少相关SSl证书

其实是 访问对方系统地址使用的https 导致的 将地址改为http即可