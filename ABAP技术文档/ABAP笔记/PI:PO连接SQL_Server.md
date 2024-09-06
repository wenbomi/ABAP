# PO/PI 连接SQL-Server

## 查看PI上是否安装相关组件

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802090352047-434044198.png" alt="image-20240722084750507" style="zoom:33%;" />

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802090352691-1779274847.png" alt="image-20240722084830509" style="zoom:50%;" />

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802090353256-1038534883.png" alt="image-20240722084856853" style="zoom:50%;" />

筛选组件名称 com.sap.aii.adapter.lib

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802090353802-1067380661.png" alt="image-20240722084949438" style="zoom:50%;" />

## 下载相关的JDBC驱动程序

解析依赖关系

注意 SAP NetWeaver 版本、所需的 JVM 和受支持的 JDBC 驱动程序之间的依赖关系。始终使用与 SAP NetWeaver 发行版的 JVM 版本兼容的 JDBC 驱动程序版本。

查看PO上的Java版本

SAP NetWeaver Administrator->配置->基础架构->系统信息->00(instance IDXXX)

得到Java版本  1.8+

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802090354500-964014304.png" alt="image-20240722090318508" style="zoom:33%;" />

下载JDBC驱动支持

链接：https://learn.microsoft.com/en-us/sql/connect/jdbc/microsoft-jdbc-driver-for-sql-server-support-matrix?view=sql-server-ver15

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092244574-315686921.png" alt="image-20240725152305270" style="zoom:33%;" />

找到支持1.8版本的驱动下载，这里随机选择版本12.6 下载ZIP

必须要下载英文版的

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092245045-1232478419.png" alt="image-20240725170224048" style="zoom:33%;" />

解压压缩包，找到路径 ：sqljdbc_12.6/chs/jars 

除了mssql-jdbc-12.6.3.jre8.jar 留下外，其余均删除掉

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092245634-302614854.png" alt="image-20240725152722909" style="zoom:50%;" />

## 下载XI 第三方组件（需要S账号）

下载 java-utility Java 支持工具 （据说只是查看PO版本用的 若已知版本 7.50 不必下载）

链接：https://wiki.scn.sap.com/wiki/display/ASJAVA/SAP+NW+Java+Support+Tool

对于SAP PI / PO系统来说，它是一个非常有用的版本查看器

若要打开该工具 要求Java版本为1.8 更高的会报错

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092246038-1176874391.png" alt="image-20240725160137895" style="zoom:33%;" />



下载SCA组件

下载链接：https://me.sap.com/softwarecenterviewer/73554900100200002120/MAINT

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092246407-1529639439.png" alt="image-20240725161719294" style="zoom:50%;" />

SDA文件的创建

下载**SDA Maker Tool utility**

下载链接：https://launchpad.support.sap.com/#/notes/1028961

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092246852-905474761.png" alt="image-20240725161422988" style="zoom:50%;" />



按照要求依次选择版本 、驱动、填入组件SCA文件、包含jar包的文件、以及生成SDA文件的文件夹地址

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092247343-1111263513.png" alt="image-20240725164711475" style="zoom:33%;" />

SDA文件生成

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092247719-1357532979.png" alt="image-20240725164856530" style="zoom:50%;" />



服务器连接和文件部署

### 服务器连接

下载使用finalshell 

下载链接：https://www.hostbuf.com/

SDA文件上传服务器指定路径上

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092248006-1206569697.png" alt="image-20240726170003224" style="zoom:33%;" />

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092248435-904003849.png" alt="image-20240726170427204" style="zoom:50%;" />

## 文件部署

使用telnet方式部署

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092248966-1920458403.png" alt="image-20240726170129995" style="zoom:50%;" />

切换专门部署服务器的账号

账号查询，得到部署账号为hoqadm

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092249468-1749918246.png" alt="image-20240726170700029" style="zoom:50%;" />

使用命令切换账号：

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092249981-2013746588.png" alt="image-20240726171104541" style="zoom:50%;" />

使用命令telnet localhost 55<实例号>08 进入部署界面

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092250640-1536061216.png" alt="image-20240726171927715" style="zoom:33%;" />

输入 user name统一为：Administrator

密码：xxx

![image-20240726174128697](https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092254402-593424766.png)

按照命令部署SDA文件

add deploy

deploy /usr/sap/<SID>/J<instance>/j2ee/temp/com.sap.aii.adapter.lib.sda version_rule=all on_deploy_error=stop

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092251406-1403605719.png" alt="image-20240728104606016" style="zoom:50%;" />

此时 服务器处于停机状态

根据相关命令 重启服务器

检查部署情况

cat /usr/sap/<SID>/J<instance>/work/deploy.0.log

![image-20240728105224803](https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092254815-506494778.png)

部署后 检查库

按照步骤一的方式查看

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092251893-143747815.png" alt="image-20240728105530946" style="zoom:30%;" />

部署完成

## 配置ID和ESR

参考链接：https://community.sap.com/t5/technology-blogs-by-members/sap-pi-proxy-to-jdbc-scenario/ba-p/13326556

配置ID通道

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092252231-830892795.png" alt="image-20240728110735245" style="zoom:33%;" />

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092252766-2073905687.png" alt="image-20240728110615859" style="zoom:50%;" />

配置PI Proxy



<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092253667-545295937.png" alt="image-20240728111649380" style="zoom:50%;" />

<img src="https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092254075-1718720351.png" alt="image-20240728111728468" style="zoom:50%;" />

# 遇到的问题

1、telnet command is not found 

解决方案：在服务器上安装telnet (在root账号下安装)

zypper addrepo https://download.opensuse.org/respositories/network:/utilities/SLE_15_SP3/network:utilities.repo

zypper update

zypper install telnet

2、PO报错

```javascript
com.microsoft.sqlserver.jdbc.SQLServerException: "encrypt" property is set to "true" and "trustServerCertificate" property is set to "false" but the driver could not establish a secure connection to SQL Server by using Secure Sockets Layer (SSL) encryption: Error: sun.security.validator.ValidatorException: PKIX path building failed: sun.security.provider.certpath.SunCertPathBuilderException: unable to find valid certification path to requested target.  ClientConnectionId:c01a30e7-eaee-48ac-a4cc-309e371c0846
```

解决方案：

![image-20240728111430705](https://img2023.cnblogs.com/blog/2606674/202408/2606674-20240802092255222-570961068.png)
