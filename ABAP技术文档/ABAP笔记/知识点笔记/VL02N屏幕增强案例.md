

# **事务码 VL02N 屏幕增强**

实现效果：增加一个物流标签和对应页签面展示两个字段的信息

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725092111810.png" alt="image-20240725092111810" style="zoom: 25%;" />



因为这是自定义的两个参数，该事务码主要展示的**LIKP**表中字段，但是展示字段物流公司和物流号在likp中不存在，首先需要对likp表进行表字段增强 

在增强结构ZZLIKP下增加两个展示字段

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725092306660.png" alt="image-20240725092306660" style="zoom:50%;" />

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725092322099.png" alt="image-20240725092322099" style="zoom:50%;" />



1、网上查询该事务码增强得知是第三代badi增强

事务码 se18-> LE_SHP_TAB_CUST_OVER->显示

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725092552551.png" alt="image-20240725092552551" style="zoom:50%;" />

双击

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725092629380.png" alt="image-20240725092629380" style="zoom: 25%;" />双击 实施

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725092655118.png" alt="image-20240725092655118" style="zoom:25%;" />

这就是该增强下 创建的实施

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725092727123.png" alt="image-20240725092727123" style="zoom:50%;" />

2、若是要创建新的实施，回到初始界面->实施->创建

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725092803492.png" alt="image-20240725092803492" style="zoom:20%;" />

填写实施名称，一般是自定义的

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725092844422.png" alt="image-20240725092844422" style="zoom:25%;" />

填写该实施的简介

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725092930076.png" alt="image-20240725092930076" style="zoom:15%;" />



3、编写增强相关代码

双击创建的实施 zme001

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725093009406.png" alt="image-20240725093009406" style="zoom:33%;" />

双击实施类->进入方法界面

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725093038826.png" alt="image-20240725093038826" style="zoom:25%;" />

根据描述得知第一个方法是增加页签、第二个方法为获取数据传输到页签屏幕上、方法3为在页签上对数据处理后在送回主数据、方法4没用到不知道

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725093108423.png" alt="image-20240725093108423" style="zoom:25%;" />

双击进入方法一代码编写页面

参数1：页签名称

参数2：页签位置

参数3：创建的函数组对应的主程序名称（函数组：LZFG_VL02N_ENHANCEMENT）

参数4：屏幕编号

参数5：客户化 = ‘X’ 不知道这个参数含义 估计是该增强是标准屏幕上私人定制

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725093130089.png" alt="image-20240725093130089" style="zoom:50%;" />

一般函数、屏幕都是要存放在函数组中，既然我们要在标准屏幕上增加新的屏幕，这时就要创建一个新的函数组LZFG_VL02N_ENHANCEMENT

在函数组中创建一个子屏幕9100（属性为子屏幕）

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725093147940.png" alt="image-20240725093147940" style="zoom:25%;" />

在屏幕上画出两个我们要展示的字段。实例中是展示物流公司（ZZLOGIS）和快递号（ZZLOGNO）

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725093207309.png" alt="image-20240725093207309" style="zoom:25%;" />



如何将标准程序中的数据读取到我们定义的屏幕显示，就需要定义一个接收函数zset_delivery_item_scrn_val（名称自定义）

因为增强点上全局参数为likp表类型的，所以这里也定义一个接收参数类型为likp表类型的

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725094126861.png" alt="image-20240725094126861" style="zoom:25%;" />

上面的SI_LIKP是定义在函数组主程序的TOP模块中，方便整个函数组全局使用

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725094245234.png" alt="image-20240725094245234" style="zoom:25%;" />

最后就是如何将数据绑定到对应的屏幕字段

在子屏幕上画出的两个参数展示框，对应的字段为物流公司（ZZLOGIS）和快递号（ZZLOGNO），在全局top中也定义两个相同名称的参数

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725094310160.png" alt="image-20240725094310160" style="zoom: 33%;" />

根据全局的接收数据，将数据中对应的参数绑定给全局的展示字段，该段代码写在子屏幕的PBO中

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725094345128.png" alt="image-20240725094345128" style="zoom:33%;" />

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725094411791.png" alt="image-20240725094411791" style="zoom:33%;" />



回到增强点的第二个方法就是接收主程序数据的位置

调用刚创建的函数接收VL02N数据，is_lips是全局参数，增强点自带的

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725094433458.png" alt="image-20240725094433458" style="zoom:33%;" />

5、激活增强

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240725094452466.png" alt="image-20240725094452466" style="zoom:33%;" />