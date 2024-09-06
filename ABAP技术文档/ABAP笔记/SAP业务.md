## 1、判断日期是否是关账期

账期分为：物料账期（marv）和财务账期(T001B)

物料账期：bukrs(公司代码)  日期在frpe1与tope1之间为关账期
<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230606152005978.png" alt="image-20230606152005978" style="zoom:50%;" />

财务账期：LFMON(当月)   VMMON(前一个月) 在 VMMON-LFMON期间的日期为关账期！否则为开账期<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230606151236749.png" alt="image-20230606151236749" style="zoom:50%;" />
一般物料账期范围比财务账期范围大

```ABAP
ls_month_f = s_zdat-low+4(2). "s_zdat 为输入的日期 开始日期
ls_month_e = s_zdat-high+4(2). ""  结束日期
SELECT SINGLE tope1 FROM t001b INTO @DATA(ls_too1b) WHERE bukrs = '1020' AND  tope1 BETWEEN  @ls_month_f AND  @ls_month_e .
  IF sy-subrc = 0. "关账 继续判断"
    SELECT COUNT(*) FROM marv WHERE bukrs = '1020' AND vmmon >  ls_month_f AND lfmon < ls_month_e.
    IF sy-subrc = 0.
    " 关账"
    ELSE.
      MESSAGE '当前日期在开账期间！' TYPE 'E' DISPLAY LIKE 'W'.
    ENDIF.
  ENDIF.
```



## 4、判断物料是否启动批次管理

```abap
SELECT COUNT(*) FROM marc 
  WHERE matnr = ls_output-matnr 
  AND werks = ls_output-werks 
  AND xchar = 'X'.
```

在表marc中根据主键（物料和工厂）查询该物料的xchar字段是否为‘X’，为X表示启动批次管理 反之 不是

## 5、QALS表

```
SAP 表 QALS 是一个非常重要的质量管理模块 (QM) 相关表。主要存储了所有产品和工序的批次信息，包括检验状态、合格情况、以及具体的检测参数等级别的详细信息。
```

## 6、查内部批次

查内部批次（zbatch）一般从zmmt007表根据 物料（matnr）、工厂(werks)、SAP批次(charg) 查询。

## 7、建立预留单号 300系统

事务码 ：mb21

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230608101255839.png" alt="image-20230608101255839" style="zoom:50%;" />

回车进入下一个界面
<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230608101518106.png" alt="image-20230608101518106" style="zoom:50%;" />

保存 创建预留单号： 0000014007

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230608101545493.png" alt="image-20230608101545493" style="zoom:50%;" />

migo 预留过账

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230608101659219.png" alt="image-20230608101659219" style="zoom:50%;" />

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230608101806504.png" alt="image-20230608101806504" style="zoom:50%;" />

**项目确定**-检查-过账-生成物料凭证



## 8、判断采购订单为调拨单

```ABAP
SELECT SINGLE 'X' INTO @DATA(field)
        FROM ekpo
        INNER JOIN ekko ON ekko~ebeln = ekpo~ebeln
        WHERE ekpo~ebeln = @cs_goitem-ebeln  "采购订单号"
            AND ekpo~ebelp = @cs_goitem-ebelp "采购凭证的项目编号"
            AND ekko~reswk  <> '' "转储单的供应(发出)工厂 "
            AND bstae = ''.  " 确认控制代码"
```

 判断交货单是外向交货

```ABAP
SELECT COUNT(*) FROM lips 
  WHERE vbeln = cs_goitem-vbeln
  AND posnr = cs_goitem-posnr
  AND pstyv = 'NLC'.
```

## 9、屏幕事件

```abap
 " 程序 ZMWB_20231201"
 INITIALIZATON  "在选择界面前触发
 
 AT SELECTION-SCREEN "在选择界面上的事件 主要作用是验证输入信息
 
 AT SELECTION-SCREEN OUTPUT "在程序执行前会优先检查该事件下的代码 可以用来对选择界面输入值的校验
 
 START-OF-SELECTION "选择屏幕结束处理后，点击 执行按钮后触发
 
 TOP-OF-PAGE "由第一条WRITE语句触发以在新页面上显示数据。
 
 END-OF-SELECTION  "START-OF-SELECTION结束后触发
 
 END-OF-PAGE  "触发以在报表的页面末尾显示文本。 请注意，此事件是创建报告时的最后一个事件，应与REPORT语句的LINE-COUNT子句结合使用。

```

```ABAP
 AT SELECTION-SCREEN OUTPUT "在程序执行前会优先检查该事件下的代码 可以用来对选择界面输入值的校验
```

```abap
PARAMETERS:p_werks TYPE werks_d,
           p1      TYPE spfli-carrid.

AT SELECTION-SCREEN OUTPUT.
  IF p_werks IS INITIAL .
    MESSAGE |XXX| TYPE 'S' DISPLAY LIKE 'E'.
  ENDIF.
```

```abap
"实现程序执行前对指定域数据检查，执行该事件时，其他输入域的输入状态会被锁定。
AT SELECTION-SCREEN ON [para|selcrit].
  PARAMETERS:p_werks TYPE werks_d.
  AT SELECTION-SCREEN ON p_werks.
    IF p_werks IS INITIAL .
      MESSAGE |XXX| TYPE 'E' . " MESSAGE 'XXX' TYPE 'S' DISPLAY LIKE 'E'. LEAVE SCREEN. "
    ENDIF.
```

案例 

```ABAP
REPORT zmwb_0324
LINE-SIZE 75
LINE-COUNT 30(3)
NO STANDARD PAGE HEADING.

TABLES: mara.
TYPES: BEGIN OF itab,

         matnr TYPE mara-matnr,
         mbrsh TYPE mara-mbrsh,
         meins TYPE mara-meins,
         mtart TYPE mara-mtart,

       END OF itab.

DATA: wa_ma TYPE itab,
      it_ma TYPE STANDARD TABLE OF itab.

SELECT-OPTIONS: mats FOR mara-matnr OBLIGATORY.

INITIALIZATION. " 选择界面上的初始值
  mats-low = '1'.
  mats-high = '500'.

  APPEND mats.

AT SELECTION-SCREEN.   " 对选择界面上输入的值进行检查
  IF mats-low = ''.
    MESSAGE i000(zkmessage).
  ELSEIF mats-high = ''.
    MESSAGE i001(zkmessage).
  ENDIF.

TOP-OF-PAGE.   "第一个WRITE语句触发
  WRITE:/ 'CLASSICAL REPORT CONTAINING GENERAL MATERIAL DATA  FROM THE TABLE MARA' COLOR 7.
  ULINE.
  WRITE:/ 'MATERIAL' COLOR 1,

  24 'INDUSTRY' COLOR 2,
  38 'UNITS' COLOR 3,
  53 'MATERIAL TYPE' COLOR 4.
  ULINE.

END-OF-PAGE.

START-OF-SELECTION. "取数等操作"
  SELECT matnr mbrsh meins mtart FROM mara
  INTO TABLE it_ma WHERE matnr IN mats.

  LOOP AT it_ma INTO wa_ma.
    WRITE:/  wa_ma-matnr,

    25 wa_ma-mbrsh,
    40 wa_ma-meins,
    55 wa_ma-mtart.
  ENDLOOP.
  
END-OF-SELECTION.

  ULINE.
  WRITE:/ 'CLASSICAL REPORT HAS BEEN CREATED' COLOR 7.
  ULINE.
  SKIP.
```

## 10、QA32 更改检验批的数据信息

## 11、生产中的移动类型

```
101： 收货到库存
102： 收货冲销
261： 有关订单的发货
262： 有关订单的收货
Z01:  样品收货
Z02:  样品收货冲销
```

###        生产中的借贷标识

```
H：贷方 
S: 借方
```

## 生产订单常用表

```
AFKO : 储存订单表头数据 
```

```
AFPO: 订单行项
```

```
AUFK: 订单主数据
```

```
AUFM 具有货物移动的订单
```

## 日期校验

```abap
CALL FUNCTION 'DATE_CHECK_PLAUSIBILITY'
    EXPORTING
      date                      = ls_output-hsdat "校验的日期"
    EXCEPTIONS
      plausibility_check_failed = 1
      OTHERS                    = 2.
  IF sy-subrc NE 0. " 如果返回非0，则日期不合法
    lv_msgty = 'E'.
    lv_msgtxt = lv_msgtxt &&  '非法日期/'.
  ENDIF.
```

## migo查看 物料凭证

```
A04显示 -》R02物料凭证 -> 物料凭证填写-> 年度
```

## 1、A01采购入库

### 建立采购信息记录 -me11

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231209101805657.png" alt="image-20231209101805657" style="zoom:33%;" />

供应商填写完后 回车会跳出一个维护界面

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231209102156496.png" alt="image-20231209102156496" style="zoom:25%;" />

me13查看采购信息记录单号

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231209102648633.png" alt="image-20231209102648633" style="zoom:25%;" />

点击文本  esc返回 保存 后 到了 价格维护界面 下面为必填项

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231209103002665.png" alt="image-20231209103002665" style="zoom:25%;" />

## 2、创建采购订单

me21n

输入供应商 然后回车

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231209103924922.png" alt="image-20231209103924922" style="zoom:33%;" />

填入弹出框内的必填信息

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231209104003292.png" alt="image-20231209104003292" style="zoom:33%;" />



回车后 填写物料 回车

![image-20231209104048378](/Users/miwenbo/Library/Application Support/typora-user-images/image-20231209104048378.png)

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231209104530411.png" alt="image-20231209104530411" style="zoom:33%;" />

订单数量和工厂必填

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231209104110260.png" alt="image-20231209104110260" style="zoom:33%;" />

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231209104203338.png" alt="image-20231209104203338" style="zoom:33%;" />

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231209104616236.png" alt="image-20231209104616236" style="zoom:33%;" />

保存

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231209104647167.png" alt="image-20231209104647167" style="zoom:33%;" />

me22n 修改采购订单信息

me23n 查看采购订单信息



3、采购订单的审批 me29n

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231209105016131.png" alt="image-20231209105016131" style="zoom:33%;" />



<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231209105032196.png" alt="image-20231209105032196" style="zoom:33%;" />

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231209105056990.png" alt="image-20231209105056990" style="zoom:33%;" />

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231209105112841.png" alt="image-20231209105112841" style="zoom:33%;" />

3、建立交货单 vl31n

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231209104920147.png" alt="image-20231209104920147" style="zoom:33%;" />

本应收10个料 本次只交货8个 填入库存地点

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231209105441997.png" alt="image-20231209105441997" style="zoom:33%;" />

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231209105530266.png" alt="image-20231209105530266" style="zoom:33%;" />

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231209105754647.png" alt="image-20231209105754647" style="zoom:33%;" />

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231209105817150.png" alt="image-20231209105817150" style="zoom:33%;" />

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231209105851961.png" alt="image-20231209105851961" style="zoom:33%;" />



## 常见采购业务

```
1、一般采购业务：正常的采购过程（供应商采购-供应商送货-发票校验-。。。）
2、寄售采购业务：先把供应商提供的原料放在我们仓库里，并不先结算金额，生产中消耗多少给多少的钱
3、委外加工采购业务：让别的工厂给我加工产品
4、工序委外采购业务：生产一个产品有三道工序，比如第二个工序我们没法做，就将第二道工序交给其他能做的公司代加工。
5、集团内公司间交易：常州工厂因为缺少一些材料向海门工厂采购，两家企业都是隶属于矿业集团的。
6、服务型采购：采购的不是实体，而是服务，比如 系统的维护
7、第三方采购业务：某个产品我们不生产，但是可以作为黄牛，向生产该产品的供应商采购，然后卖给指定的用户
等等
```

## 库存状态和类型

```
1、非限制使用库存：根据业务需求可以任意使用的库存
2、冻结库存：供应商送过还没进行质检的还不能直接投入使用
3、质检库存 
4、在途库存 a公司发出 b公司还没收到
5、寄售库存
6、分包商库存
```

## 库存盘点

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231215135359393.png" alt="image-20231215135359393" style="zoom:33%;" />

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20231215140548631.png" alt="image-20231215140548631" style="zoom:50%;" />

## 创建物料主数据

1、TCode-mm01

# QM知识点

## 1、主检验特征

主检验特性通常被认为是最重要的检测点，因为它可以对物料或批次的最终质量评估产生最重要的影响。

1.1 创建主检验特征

事务码：qs21

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230626160948816.png" alt="image-20230626160948816" style="zoom:50%;" />

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230626161515386.png" alt="image-20230626161515386" style="zoom:50%;" />

点击第二步的-控制标识符 弹出如下界面

选择规格下限、规格上限、采样过程  -点击 ✅

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230626161637106.png" alt="image-20230626161637106" style="zoom:50%;" />

弹出另外一个维护窗口

参数默认 点击✅ 进入下一步

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230626162015857.png" alt="image-20230626162015857" style="zoom:50%;" />

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230626162114985.png" alt="image-20230626162114985" style="zoom:50%;" />

继续回车或者✅ 进入维护界面 如下

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230626162458256.png" alt="image-20230626162458256" style="zoom:50%;" />



回到主界面 - 点击保存-创建成功

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230626162602646.png" alt="image-20230626162602646" style="zoom:50%;" />

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230626162637494.png" alt="image-20230626162637494" style="zoom:50%;" />

定量涉及的主要字段参数：

定量特征-QUANTITAET

状态 ： QPMK-LOEKZ

复制模块/参考特性   QPMK-KONSISTENT

短文本; VSKT-KURZTEXT

搜索字段：QPMK-SORTFELD

小数位;QPMK-STELLEN

计量单位: RMQSD-MASSEINHSW

目标值：QFLTP-SOLLWERT

规格下限：QFLTP-TOLERANZUN

规格上限：QFLTP-TOLERANZOB



主检验特征-定性创建

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230626163735563.png" alt="image-20230626163735563" style="zoom:50%;" />

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230626163859764.png" alt="image-20230626163859764" style="zoom:50%;" />

点击控制标识符

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230626165018019.png" alt="image-20230626165018019" style="zoom:50%;" />

点击✅

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230626165128818.png" alt="image-20230626165128818" style="zoom:50%;" />

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230626165302558.png" alt="image-20230626165302558" style="zoom:50%;" />

回到主界面上 ，此时目录上被打钩 点击目录可以回到上一个界面进行修改维护

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230626165352677.png" alt="image-20230626165352677" style="zoom:50%;" />

点击主界面的保存按钮

-- 特性 1021 MWB_01 2023.06.26 已建立



定性使用到的主要字段：

定性特征-RMQSD-QUALITAET

工厂： QPMZ-AUSWMGWRK1

状态 ： QPMK-LOEKZ

复制模块/参考特性   QPMK-KONSISTENT

短文本; VSKT-KURZTEXT

搜索字段：QPMK-SORTFELD

代码组或选择设置：QPMZ-AUSWMENGE1







## 2、qa01 创建检验批

创建检验批最重要的两个字段 批次 和 检验批数量

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230712225319520.png" alt="image-20230712225319520" style="zoom:50%;" />

创一个检验批 890000015336

### QA01-取消批次

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230722101142822.png" alt="image-20230722101142822" style="zoom:30%;" />

## 3、QA02更改检验批

创建好的检验批可以通过表 qals查询到

对检验批做决策和过账 QA11

做决策主要是填入决策代码和代码组 一旦做了决策就不能使用qa11查询该批次信息
可以使用QA12更改决策或者是QA13查看批次决策信息

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230712225829516.png" alt="image-20230712225829516" style="zoom:50%;" />

针对89*开头的物料是不能进行过账的 

QA03 查看检验批的内容

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230722100614472.png" alt="image-20230722100614472" style="zoom:30%;" />

## QM中常用表

```
QALS 检验批记录 检验批的基本信息 
```

```
QAMV 存储检验批特性 包含 工序节点 上下限
```

```
QAVE 检验批的使用决策相关信息
```

```
QPMT 检验方法记录
```

```
QDSV 采样过程信息记录
```

```
QPMK 主检验特性存放记录
```

```
QPMZ  检验方法/主检验特性
```



## QA32 事务码 查看检验批的基本信息和修改

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20230808093435448.png" alt="image-20230808093435448" style="zoom: 33%;" />



## 供应商的创建

事务码 bp 100413

物料创建

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20240101112534306.png" alt="image-20240101112534306" style="zoom:33%;" />

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20240101112634561.png" alt="image-20240101112634561" style="zoom:50%;" />

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20240101112648418.png" alt="image-20240101112648418" style="zoom:50%;" />

<img src="/Users/miwenbo/Library/Application Support/typora-user-images/image-20240101112804903.png" alt="image-20240101112804903" style="zoom: 50%;" />
