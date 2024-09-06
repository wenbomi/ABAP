## 1.创建内表的方式

```ABAP
*1. 定义表结构的一种方式
	types: name01 type 结构类型 occurs 0.  // occurs 直接声明这个类型是表结构
	data name02 type name01.
	
*2. 常规创建表结构
	TYPES: BEGIN of <结构名>，
		<field1> type <structure1>,
		<field2> type <structure2>,
		<field1> type <structure3>,
		...
		END OF <结构名>.
		
	DATA <name> type table of <结构名>.

*3. 第三种创建内表语法
	types: <t_itab> type table of <结构名>. “ 这是创建的表类型
	data <itab> type <t_itab>.  ” 创建内表
	
*4. 直接创建内表方式
 DATA: begain of <表名> ，
		<field1> type <s1>,
	...
 END of <表名>.
 
  "" 这种定义方式自带表头
  data: begin of <表名>  occurs 0， 
		<field1> type <s1>,
	...
 end of <表名>.
 	
```

```ABAP
* 代码示例：
TYPES: BEGIN OF ty_itab,  " 结构类型
         field1 TYPE char10,
         field2 TYPE i,
       END OF ty_itab.

TYPES: t_itab1 TYPE ty_itab OCCURS 0 ,    " 定义表类型 有表头的
       t_itab2 type table of ty_itab  .  	" 定义表类型 无表头的

DATA: itab1 TYPE TABLE OF ty_itab,  "  参照结构类型创建表
      itab2 TYPE t_itab1,           " 参照表类型创建内表 有表头
      itab3 TYPE t_itab2,   				" 参照表类型创建内表 无表头
      itab4 LIKE itab1.    					" 参照定义好的内表创建结构相同的表
      
* 定义结构体的同时，创建内表      
DATA: BEGIN OF ty_itab1 OCCURS 0,  " 定义了内表结构的同时也定义了与自身名称相同的工作区
        field1 TYPE char10,
        field2 TYPE i,
      END OF ty_itab1.

* 参考定义
DATA: itab5 TYPE TABLE OF sflight,           " 参照数据库表创建内表
      itab6 TYPE TABLE OF zalsmex_tabline.   " 参照数据字典结构创建内表
```

## 接口中定义结构

```abap
DATA:BEGIN OF customer_crm,
       code TYPE char10 VALUE '1000001000',
       name TYPE char35 VALUE 'XLevon',
       BEGIN OF information ,
         company_code       TYPE char4 VALUE '1000',
         sales_organization TYPE char4 VALUE '1010',
         country            TYPE char3 VALUE 'CN',
       END OF information,
     END OF customer_crm.
```

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145859869-269556536.png" alt="image-20231229103811687" style="zoom: 40%;" />

## 2.参考定义

```ABAP
1. Like  参照内表或数据库表定义内表，内表结构与所参照数据库表完全相同：
	DATA  <f>  LIKE  <内表或数据库表>.
	
2. 参照数据字典结构或者是数据库表来创建内表
	DATA <itab> type table of <structure/table>.
	
3. 使用INCLUDE STRUCTURE包括已存在的结构体的所有字段
```

## 3.查看内表的属性

```ABAP
1.通过describe获取内表的行
	describe table <itab> lines n.
```

```ABAP
* 代码展示
DATA n TYPE i.
DESCRIBE TABLE itab1 LINES n.
```

## 4.对内表的操作

### 4.1  内表的插入操作

```ABAP
1.初识化内表
* clear 
	"若清空的内表带有表头，则clear清空的是该内表的表头数据，而不能清空内表，若内表没有表头，则清空所有行数据。
	"若想清空内带有表头内表的数据 使用clear table[]的方式。

* refresh
	"清空内表的数据，但是不能清空工作区的内容。
	
2. 插入数据
	* 三个关键字
		- insert (wa-工作区 itab-内表 ):按照内表的具体字段插入一行或者是多行
		// 加了index 干涉 插入行的位置
		insert wa into itab index idx.
		
		 "不加index 要加table  不干涉插入的位置
		insert wa into table itab.
		insert wa initial line into table itab. // 插入一个空行
		
		//插入一个空行 在指定的位置
		insert wa into initial line itab index idx.  
		
		// 插入多行数据
		insert lines of itab1 into itab2 .
		insert lines of itab1 [from idx1] [to idx2] into itab2 [index idx3] .
		
	-- append
	-- collect
	
3.逐行插入数据选择插入方式
		* 要将内表仅作为数据的存储，建议使用append，比较节约性能 (to)
			append wa to itab.
			
		* 要计算数字字段的和或者是想保证内表的数据没有重复行，选择使用collect (into)
			collect wa into itab.
			
		* 要在内表中现有行之前插入新行，使用insert. (into)
			insert wa into itab .
	
4. 将内表内容复制到另一个内表中 
     * 要将内表行附加到另一个内表中，请使用APPEND语句。
     	- append lines of itab1 to itab2.
     	
     * 要将内表行插入另	一个内表中，请使用INSERT语句。
     	- insert lins of itab1 into itab2.
     	
     * 要将内表条目内容复制到另一个内表中，并且覆盖该目标表格，请使用 MOVE语句。
			- move itab1 to itab2.
			或者是
			- itab2 = itab1.
```

### 内表的更新操作

```ABAP
1.更新数据
  wa-field1 = 'fuck'
  wa-field2 = 99
  modify itab from wa index 2. // 对表第二行的数据进行更新操作，将字段1改成fuck 字段2改成99
  
2.更新某行字段的某个字段值
  // 只是修改itab索引为第1行的field1字段，其他字段不改变
  modify itab from wa index 1 transporting field1.
```

## 内表的删除

```ABAP
DELETE
// 按照条件删除 删除的是内表
delete itab where key1 = value1 and key2 = value2.

// 按照工作区的内容删除
wa-f1 = ...
wa-f2 = ...
delete table itab from wa.

// 删除具体行
delete itab index idx.

// 删除具体范围
delete itab from idx1 to idx2.

// 删除重复行，删除前需要对表进行排序
DELETE ADJACENT DUPLICATES FROM  itab. //默认是根据非数字字段为主键，来删除主键相同的行数据
-- 比如 itab-f1 = '0001' 
			 itab-f2 = 66.
			itab-f1 = '0001'
			itab-f2 = 78.
			使用 DELETE ADJACENT DUPLICATES FROM  itab.时上述两行数据会删除其中一行，虽然第一个字段内容相同第二个不同 但是默认还是按照第一个字段作为是否为重复项来匹配。

// 若想避免上述问题
	DELETE ADJACENT DUPLICATES FROM  itab comparing all fields. //在删除重复行的时候会考虑到除了主键以外的其他字段是否重复。
	
"delete 数据库表内容"
delete from db where ...

```

## 内表排序

```abap
sort itab [by key1 key2...kn] [ascending / descending]
// key1 ..keyn 不一定是主键 字段也可以 默认下为升序
```

## READ

```ABAP
1.有表头的读取
read table itab to wa.

2.有条件的读取 按照字段为键
read table itab to wa  with key k1 = v1 k2 = v2 [binary search].

3.按照索引读取
read table itab index i.

" 每次只读取一条数据
" 使用READ操作的表必须有工作区作为查找出的数据存储窗口
" BINARY SEARCH可以提高内表数据查找的速度，但是使用前必须先对内表进行排序
" read操经常搭配if sy-subrc = 0 endif.来对判断是否读取成功并进行后续的操作。
```

## LOOP

```ABAP
CLEAR wa. // 工作区
LOOP AT book1 INTO wa [where 条件]. // 逐条读取内表数据到工作区
  WRITE:/ wa-price.
ENDLOOP.
" 循环读取内表中的每行数据，直到数据读完
```

![image-20240830091947190](../../../Library/Application%20Support/typora-user-images/image-20240830091947190.png)

## FOR

理解为LOOP，是对实现操作符 NEW 和值操作符VALUE的一种增强，作用是构造內表内容

```abap
TYPES: BEGIN OF ty_line,
         col1 TYPE i,
         col2 TYPE i,
         col3 TYPE i,
       END OF ty_line.      "结构体
DATA: gt_tab TYPE TABLE OF ty_line.

gt_tab = VALUE #( FOR j = 11 THEN j + 10 UNTIL j > 40
                  ( col1 = j col2 = j + 1 col3 = j + 2 )
                                ).
cl_demo_output=>display( gt_tab ).
```

```abap
SELECT * FROM zmwb_students INTO TABLE @DATA(gt_data).
DATA: lt_data LIKE gt_data.
lt_data = VALUE #( FOR <fs> IN gt_data WHERE ( zsex = 'F' )
                    ( zcode   = <fs>-zcode
                      zname   = <fs>-zname
                      zsex    = <fs>-zsex
                      zschool = VALUE #( gt_data[ zcode = <fs>-zcode ]-zschool  )
                     )
                  ).

out->write( gt_data ).
out->write( lt_data ).
```

## 操作符FILTER



## debug的操作

```
1.F5 执行下一步 一步一步的去往下执行

2.F6 执行下一步，但是遇到类似于函数的子模块，不进入子模块内部，直接执行完，F5则会进入模块内部，一步一步的执行查看。

3.F7 跳出当前模块的执行 ，比如F5进入到函数内部执行，当我不想执行这个内部模块时，通过F7返回到这个函数外部。
```

##  判断内表有多少行

```ABAP
DESCRIBE TABLE GT_ALV LINES GV_LINES. " 
```

## 内表满足条件的行数

```abap
DATA(lv_lines) = REDUCE i( INIT x = 0
                            FOR lw IN lt_data 
                            WHERE ( prueflos = ls_qals-prueflos AND mbewertg = 'R' )
                            NEXT x += 1 ).

```



## Open SQL

```ABAP
对数据库的主要操作语法包括：
	* select / update / insert / delete / modify
	* 执行状态根据sy-subrc表现
		若为0 表示执行成功
		若为非0 表示执行失败
```

## 1.读取数据

```ABAP
 - select 
  select <result> 
  from <db> 
  into <target> 
  where <条件>
  group by <field> 
  order by <field>.

其中各关键字的属性描述如下：
	SELECT <result>：具体的查询字段。
	SELECT SINGLE：定义单行查询。
  FROM <dbtab>：所查询的透明表。
  INTO <target>：查询结果赋值对象，赋值到具体表或结构体。
  INTO (<f1>...<fn>)：将查询结果赋值到具体字段。
  INTO CORRESPONDING FILES OF <itab>：将查询结果按字段匹配赋值给具体的内表或者结构体。
  WHERE <condition>：查询条件。
  GROUP BY <fileds>：分组查询条件。
  ORDER BY <fields>：排序条件。  
```

```ABAP
 select...endselect.
	"一般是循环读取数据库，其作用其实和一次性读取数据放入到内表中，然后loop内表读取数据效果是一样的 ，但是这样做会比较的浪费性能。
	
	"通过系统参数SY-DBCNT可以获取当前读取数据的行数。
	"一般配合select single来使用 数据量巨大的条件下使用单条读取。

```

### 1.1 SELECT中的常用标准函数

```
      COUNT()：统计查询总数
      SUM()  ：统计表中某个数值字段的总和
      AVG()  ：统计表中某个数值字段的平均值
      MAX()  ：统计表中某个字段的最大值
      MIN()  ：统计表中某个字段的最小值。

```

### 1.2 多表数据连接

```
需要注意的部分：
	1.连接哪些表
	2.连接条件
	3.连接哪些列

多表查询常用的两种方式：
	1.数据库视图的方式 在ABAP字典中创建数据库视图，查询时用这个视图作为数据源（个人理解：在SE11中创建一个内表，表内做好了两个表的连接以及展示的一些字段，在查询时直接查询这张关联其他两张表的数据）
	2.语法
		* 内连接：inner join (左小表右大表)
			查询结果包含两个连接表中彼此相对应的数据记录（两张表共有的部分放到目标内表中显示）
			
		* 左连接：left [out] join
			查询结果包含左表中的所有数据记录，右表中仅查询出包含相对应的匹配条件的数据（先将左边表所有数据，放在目标表中对应字段，再去匹配右边的表，能匹配到加入到目标表中匹配不到的，目标表中该字段部分置空）

		* 全连接  full out join
			包含左右表所有的记录
			
	3.关键字on
		作为连接条件
					
```

### 1.3 表字段的转换

```ABAP
	表字段名转换
	* 在查询过程中我们通常会遇到这样的情况，将透明表查询值直接传递给内表：
  * 两表中字段结构一样，但字段名称却不同
  * 通常的情况下将字段名称设为一致的
  * 除此之那还可以用AS关键字，就像SQL中的用法一样	
  
  select number AS age from <db> into itab. 

```

### 1.4 限制数据库读取的条数

```ABAP
select * into ... up to N rows.
"读取前n行的数据

  "使用PACKAGE SIZE N连续读取数据：
  "使用 UP TO n ROWS可读取前n条，但不能继续读取数据
  "使用使用PACKAGE SIZE N可连续读取数据，每次读取指定条数
  "必须应用于SELECT……ENDSELECT语句中

```

### 1.5 FOR ALL ENTRIES IN

```ABAP
概念：
	以内表中的某字段的值为条件，在数据库表中读数
	check <itab1> is not initial.
	select f1 f2  
	into <itab>
	from <db> 
	for all entries in <itab1>
	where <itab>-f1 = <itab1>-f1
	and   <itab>-f2 = <itab1>-f2.
	
	"注意：使用 for all entries in时首先要判断要参照的表是否为空。
	
	
```

## 2.更新数据

```ABAP
1.语法
	* 单条修改
	UPDATE <dbtab>(标准表，不是内表) SET f1...fn (WHERE <condition>).   
  
	* 多条修改 根据其它结构一样的表来更新
	UPDATE <dbtab> FROM TABLE <itab> (WHERE <condition>).
	
	* 根据工作区修改
	UPDATE <dbtab> FROM <wa>.
```

## 3.插入数据

```ABAP
1.单条数据的插入（工作区的插入）
	INSERT <库表> FROM wa.
	
2.多条数据的插入
	INSERT sflight FROM TABLE itab1.
```

## 4.删除数据

```ABAP
1.根据条件删除数据
DELETE FROM sflight WHERE carrid = 'SQ' AND fldate = sy-datum.

2.删除多条数据
DELETE <db> from table itab.

3.删除整个表
DELETE from <db>.

4.删除内表当前行
DELETE itab[].
```

## 5.修改数据

```ABAP
1.从工作区修改内表
	modify <db> from wa.
	* insert和modify的区别：
		- insert 是插入数据，可以重复多次，每次重复插入都会出现一条相同的数据，有时会down掉程序
		- modify 包含insert和update的操作，当库表中没有这条数据时执行插入操作，当存在这条数据时执行修改操作，及时运行多次 都不会出现重复的数据，他只会修改自身尽管自身没有发生改变。
		
2.修改多条数据
	modify <db> from table itab.
```

## 取数优化技巧

```
	* 使用OpenSQL注意事项:
		1.避免在循环语句中，使用SELECT语句，而是通过FOR ALL ENTRIES IN 语句抽取数据到内表中
		2.inner join获取数据时，尽量不要用太多的表关联，特别是大表关联，关联顺序为：小表-大表
    3.查询结果应该只需要自己关系的字段，尽量少用select * 
    4.where 条件里面多用索引、主键，顺序也要遵循小表-大表
    5.不要使用select end-select 语句，可以取出来后再处理
    6.不要用<> ,这样会不使用索引，可以用> or < 来替代
    7.不要用sort by进行排序，取出来后在内表中处理
		内表处理优化：
    		1.尽量少用多重循环
      	2.read时，可以先sort 然后用read  binary search，会快很多
      	3.对内表做批量修改时，可以用modify transporting where 语句进行替换，而不用循环修改
		8.内表求和，能够在sql层次上实现就用sql实现，不能实现的，在内表循环中用at end of之类的进行求和，                collect直接求和在数据量很大时，效率会比较低。
		9.CPU的负载可以通过优化程序来改善，在程序中尽量使用诸如SUM(SQL语句)或者COLLECT(ABAP语句)。
		10.用append lines of *** to ****,代替 loop at ***,append *** to ****,endloop.

```

## 7、where条件的使用

### 7.1 having

```
HAVING语句用于在GROUP BY子句之后对聚合结果进行过滤。HAVING语句与WHERE语句非常相似，但它们用于不同的语句部分。
HAVING语句中的条件必须是聚合函数。
```

#### 7.2 ORDER BY

 除了用在SELECT语句中，ORDER BY语句还可以用在DELETE、UPDATE、INSERT等语句中。例如，可以使用ORDER BY语句删除最后一行记录：

```ABAP
DELETE FROM sflight
ORDER BY price ASCENDING
UP TO 1 ROWS.
```

#### 7.3  子句中的逻辑运算符or 

```ABAP
SELECT * FROM sflight INTO TABLE @DATA(gt_sflight)
      WHERE PRICE = 1500 OR PRICE = 2500.
```

 该段代码使用"OR"逻辑运算符，检索出了SFLIGHT数据库表中所有price等于'1500'或者price等于2500的数据。

#### 7.4 "NOT"逻辑运算符

```ABAP
SELECT * FROM sflight INTO TABLE @DATA(gt_sflight)
      WHERE NOT ( PRICE = 1500 OR PRICE = 2500 ).
```

该段代码使用"NOT"逻辑运算符，将上方使用OR逻辑运算符的那段代码的结果进行了反转，剔除掉了SFLIGHT数据库表中所有price等于'1500'或者price等于2500的数据。

#### 7.5  IN 子句

IN 子句可以用来匹配一系列值中的任何一个。IN 子句可以使用一个列表，列表中包含需要匹配的值。这个列表可以是常量、字段或子查询的结果。

```ABAP
SELECT * FROM sflight INTO TABLE @DATA(gt_sflight)
WHERE price IN ( 1500,2500 ).
```

在这段代码中，使用IN子句利用列表限制了price的值只能为1500或者2500，但是要注意列表中的值需要用逗号分隔并且距离两端括号至少一个空格单位。

#### 7.6 LIKE子句

LIKE 子句用于基于模式的比较，它可以用来匹配一个特定的模式。LIKE 子句可以使用通配符来代替某些字符。通配符有两种：

​                ● 百分号（%）：代表任何字符序列，包括零个字符。

​                ● 下划线（_）：代表任何单个字符。

```ABAP
SELECT * FROM sflight INTO TABLE @DATA(gt_sflight)
      WHERE CARRID LIKE 'A%'.
```

上述代码将从SFLIGHT表中检索所有CARRID列以A开头的行

## 7.8  子查询

子查询是一个 SELECT 语句，它嵌套在另一个 SELECT 语句中作为一个条件。子查询的结果可以是一个单一的值、一个列表或一个表。

```ABAP
SELECT * FROM sflight INTO TABLE @DATA(gt_sflight)
      WHERE CARRID IN 
                      ( SELECT CARRID 
                           FROM spfli 
                           WHERE connid = '0026'  ).q
```

## some嵌套查询

some在sql中的逻辑运算符号，如果在一系列比较中，有些值为True，那么结果就为True。

用等号（=）和以下查询到的值比较，如果与其中一个相等，就返回

```ABAP
"当子查询的查询结果中 存在countryid=xxx的值时 才能查询到结果"
select name 
 from person 
 where countryid = some ( select countryid from country　　　
                         where countryname = '中国').
```

## all嵌套查询

当countryid大于以下返回的所有id，此结果才为True，此结果才返回

```abap
select name from person 
where countryid > all　( select countryid from country　 
                         where countryname = '中国').
```

## exists嵌套查询

exists是sql中的逻辑运算符号。如果**子查询有结果集返回，那么就为True**。exists代表“存在”的意义，它只查找满足条件的那些记录。一旦找到第一个匹配的记录后，就马上停止查找。

如果不存在Person_Id的记录，则子查询没有结果集返回，主语句不执行

```abap
SELECT * FROM Person
WHERE exists 
						( SELECT * 
							FROM Person 
               WHERE Person_Id = 100 ).
```

## 数据汇总

​		读取多个字段，并对于其中某个字段进行汇总，也就是前面几个字段数据相同的情况下，将要汇总的字段数据进行累加求和，最后在报表中只显示一行。

```ABAP
*取数
  SELECT dwerk AS werks, 
  matnr, 
  maktx, 
  zcpph, 
  gltrp, 
  meins, 
  SUM( psmng ) AS psmng 
  INTO TABLE @DATA(lt_009)
  FROM zppt009_1
  WHERE  dwerk IN @s_werks
   AND matnr IN @s_matnr
   AND zcpph IN  @s_zcpph
   AND gltrp IN @s_gltrp
   AND   zjhlx = 'D'
   GROUP BY dwerk,matnr, maktx, zcpph, gltrp, meins.

```

## 条件判断

```ABAP
" TRANSPORTING NO FIELDS WITH KEY：用于判断某个条件在内表中是否满足，但不读取任何字段进行操作.
" 示例：
READ TABLE gt_output TRANSPORTING NO FIELDS WITH KEY msgty = 'E'.
```

```abap
"(lv_filter) 是动态条件"
data:lv_filter TYPE string VALUE 'SELKZ EQ abap_true'.
LOOP AT <lt_data> TRANSPORTING NO FIELDS WHERE (lv_filter).
    
ENDLOOP.
```

## 2. ALV报表开发

### 2.1 FUNCTION_ALV 

```ABAP
" REUSE_ALV_GRID_DISPLAY
```

#### 变量的全部定义

```ABAP
DATA gt_student TYPE TABLE OF gty_student.
DATA gs_student TYPE gty_student.

* BUILD ALV
DATA gt_fieldcat TYPE slis_t_fieldcat_alv.  "fieldcat
DATA gs_fieldcat TYPE slis_fieldcat_alv.

DATA gs_layout TYPE  slis_layout_alv. 			"Layout

DATA gt_sort TYPE slis_t_sortinfo_alv.  		"排序
DATA gs_sort TYPE slis_sortinfo_alv.

DATA gt_filter TYPE slis_t_filter_alv.  		"筛选
DATA gs_filter TYPE slis_filter_alv.

* 定义events事件 和LVC一样
DATA gs_events TYPE slis_alv_event.
DATA gt_events TYPE slis_t_event.
```



```ABAP
TYPE-POOLS: slis.
TYPES:BEGIN OF gty_student.
        INCLUDE STRUCTURE zmwb_students.
TYPES  icon TYPE c LENGTH 4. "图标显示
TYPES  box TYPE c LENGTH 1.
TYPES   line_color TYPE c LENGTH 4. "行颜色的显示
TYPES   field_color TYPE lvc_t_scol.  "单元格颜色
TYPES  check TYPE c LENGTH 1. "复选框

TYPES: END OF gty_student.
```

#### Fieldcat

##### 1.  Fieldcat中的关键字

```ABAP
clear gs_fieldcat.
"在ALV报表中字段显示先后顺序 ，比如为“1”时 就是作为第一列字段显示
gs_fieldcat-col_pos = '1'.   

"在报表中显示的字段的字段名称，比如我想显示学生编号字段。
gs_fieldcat-fieldname = 'ZCODE'. 

"控制该字段输出长度的
gs_fieldcat-outputlen = '10'.

"给输出字段定义一个别名，比如我要显示的字段为'ZCODE'，该字段在报表中展示出来的名字是‘学生编号’
gs_fieldcat-seltext_m = '学生编号'.

gs_fieldcat-key = 'X'.  "设置为主键

gs_fieldcat-hotspot = 'X'. "设置成热点 只有设置成热点才能使用点击操作功能

gs_fieldcat-edit = 'X'.  "设置该字段为可编辑状态

gs_fieldcat-checkbox = 'X'. "在该列内最后一列显示小方块

APPEND gs_fieldcat TO gt_fieldcat.

```

##### Fieldcat另外一种方式(LVC)

```ABAP
FORM bulid_fieldcat .
  DATA:
    lr_tabdescr TYPE REF TO cl_abap_structdescr,
    lr_data     TYPE REF TO data,
    lt_dfies    TYPE ddfields,
    ls_dfies    TYPE dfies,
    ls_fieldcat TYPE lvc_s_fcat.
  CLEAR gt_fieldcat.

  CREATE DATA lr_data LIKE LINE OF gt_output.

  lr_tabdescr ?= cl_abap_structdescr=>describe_by_data_ref( lr_data ).

  lt_dfies = cl_salv_data_descr=>read_structdescr( lr_tabdescr ).

  LOOP AT lt_dfies INTO ls_dfies.
    CLEAR ls_fieldcat.
    MOVE-CORRESPONDING ls_dfies TO ls_fieldcat.

    CASE ls_fieldcat-fieldname .
      WHEN 'MBLNR'.
        ls_fieldcat-hotspot = 'X'.
      WHEN OTHERS.
    ENDCASE.
    APPEND ls_fieldcat TO gt_fieldcat.
  ENDLOOP.
ENDFORM.
```

### 2.2  LVC_ALV

#### 1. 定义变量

```ABAP
TYPE-POOLS:slis.

* 定义fieldcat
DATA gt_fieldcat TYPE lvc_t_fcat.
DATA gs_fieldcat TYPE lvc_s_fcat.

* 定义layout
DATA gs_layout TYPE  lvc_s_layo.

*定义i_save
DATA gs_variant TYPE disvariant.

* 定义排序
DATA gt_sort TYPE lvc_t_sort.
DATA gs_sort TYPE lvc_s_sort.

* 定义events
DATA gt_events TYPE slis_t_event.
DATA gs_events TYPE slis_alv_event.

* 布局保存
DATA gs_variant TYPE disvariant.

*定义filter筛选功能
DATA gs_filter TYPE  slis_filter_alv.
DATA gt_filter TYPE  slis_t_filter_alv.
```

####  2. 宏的定义

```ABAP
* 宏的定义
DATA: gv_pos TYPE i.
DEFINE  %%add_fieldcat.
  CLEAR gs_fieldcat.
  gv_pos = gv_pos + 1.
  gs_fieldcat-col_pos = gv_pos.
  gs_fieldcat-fieldname = &1.  “ 字段名
  gs_fieldcat-seltext_m = &2.  “ 在ALV中显示的名称
  APPEND gs_fieldcat TO gt_fieldcat.
END-OF-DEFINITION.
```

#### 3. 宏的填写

```ABAP
FORM build_fieldcat .
  %%append_fieldcat:   
  		....
ENDFORM.
```

#### 4. LVC-设置单元格颜色

```ABAP
TYPES: field_color TYPE lvc_t_scol.  "单元格颜色
DATA ls_scol TYPE lvc_s_scol. "在get_data里定义
gs_layout-ctab_fname = 'FIELD_COLOR'. "单元字段

IF gs_student-zsex IS INITIAL.
      ls_scol-fname = 'ZSEX'.  "字段名
      ls_scol-color-col =  3.      "颜色
      ls_scol-color-int =  1.       " 加重
      ls_scol-color-inv =  0.      "反色
      APPEND ls_scol TO gs_student-field_color.
ENDIF.
```

#### 5. LVC-设置行颜色

```ABAP
TYPES:  line_color  TYPE c LENGTH 4,  "行颜色的显示（定义中）

gs_layout-info_fname = 'line_color'. "行颜色 在layout中也要设置

IF gs_student-zsex = 'M'.  "读取数据后的判断"
      gs_student-line_color = 'C500'.
ENDIF.
```

#### 6. LVC-Layout设置

```ABAP
gs_layout-zebra = 'X'.              "输出有斑马线 方便数据查看
gs_layout-cwidth_opt = 'X'.         "根据数据长度指定字段长度
gs_layout-keyhot = 'X'.             "将主键一列设置成hotspot形式
gs_layout-confirmation_prompt = 'X'. "跳出ALV报表时打开确认窗口
gs_layout-totals_before_items = 'X'. "合计内容放在首行显示
gs_layout-edit = 'X'.   “可编辑
gs_layout-detailinit = 'X'.          "在细节屏幕显示初始值
gs_layout-grid_title = '学生信息展示'.  "标题栏文本
gs_layout-box_fname = 'BOX'.          "内部表字段的字段名称颜色
gs_layout-info_fname = 'line_color'.   "行颜色
gs_layout-ctab_fname = 'FIELD_COLOR'.  "单元字段
```

## 7. 事件的设置

**在使用事件前，一定要填写i_callback_program = sy-cprog参数。**

```ABAP
* 定义events事件
DATA gs_events TYPE slis_alv_event.
DATA gt_events TYPE slis_t_event.
```

##### 1. 表头事件（TOP_OF_PAGE）

```ABAP
CLEAR gs_events.
  gs_events-name = 'TOP_OF_PAGE'.  "标准事件 设置表头
  gs_events-form = 'FM_TOP_OF_PAGE'.
APPEND gs_events TO gt_events.


FORM fm_top_of_page.
  DATA lt_lisheader TYPE slis_t_listheader.
  DATA ls_listheader TYPE slis_listheader.
  ls_listheader-typ = 'H'.  "H = Header, S = Selection, A = Action"
  ls_listheader-info = '学生信息表信息'.
  APPEND ls_listheader TO lt_lisheader.

  ls_listheader-typ = 'S'.
  ls_listheader-key = '时间'.
  ls_listheader-info = sy-datum.
  APPEND ls_listheader TO lt_lisheader.

  ls_listheader-typ = 'A'.
  ls_listheader-info = '详细信息'.
  APPEND ls_listheader TO lt_lisheader.

  CALL FUNCTION 'REUSE_ALV_COMMENTARY_WRITE'
    EXPORTING
      it_list_commentary = lt_lisheader.
ENDFORM.
```

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145900281-917283270.png" alt="image-20221209105405322" style="zoom:50%;" />

##### 2. 点击事件（USER_COMMAND）

```ABAP
  CLEAR gs_events.
  gs_events-name = 'USER_COMMAND'.
  gs_events-form = 'FM_USER_COMMAND'.
  APPEND gs_events TO gt_events.
```

```ABAP
FORM fm_user_command USING r_ucomm LIKE sy-ucomm
                           rs_selfield TYPE slis_selfield.
  CASE r_ucomm .
    WHEN '&IC1'. "单击 & 双击 都是共享一个标志
      CASE rs_selfield-fieldname . " 判断点击的字段
        WHEN 'ZCODE'.
*          MESSAGE s004 WITH '点击了学号为' rs_selfield-value '的行'.
          SET PARAMETER ID 'LIF' FIELD rs_selfield-value. " 给事务代码FK03的必输字段填充值
          CALL TRANSACTION 'FK03'.  " 调用TCODE为FK03的函
        WHEN 'ZWEIGH'. "单击 点击了体重
          SET PARAMETER ID 'DFD' FIELD rs_selfield-value.
          CALL TRANSACTION 'ZPPR016'.
        WHEN  OTHERS. "双击
          READ TABLE gt_student INTO gs_student
                                INDEX rs_selfield-tabindex.
          PERFORM display_alv_school USING gs_student-zschool. " 查询所有学校编号为索引值的学生信息 如下代码块
      ENDCASE.
    WHEN 'CALLSF'.
      "PERFORM CALL_SMARTFORMS
    WHEN 'DOWNLOAD'.
      "PERFORM DOWNLOAD_FILE.
  ENDCASE.
ENDFORM.
```

## 双击跳转事务码

```abap
METHODS handle_double_click
      FOR EVENT double_click OF cl_gui_alv_grid     "屏幕中的双击事件
      IMPORTING e_row e_column es_row_no.
      
METHOD handle_double_click.
    READ TABLE gt_alv ASSIGNING FIELD-SYMBOL(<line>) INDEX es_row_no-row_id.
    IF sy-subrc = 0.
      SET PARAMETER ID 'MAT' FIELD <line>-matnr.
      CALL TRANSACTION 'MM03' AND SKIP FIRST SCREEN.
      "CALL TRANSACTION 'MM03'. "跳转输入界面"
    ENDIF.

  ENDMETHOD.      
```



```ABAP
"承接上段代码中的 PERFORM display_alv_school USING gs_student-zschool.
FORM display_alv_school USING p_zschool.
  DATA gt_school TYPE TABLE OF zmwb_students.
  SELECT *
    FROM zmwb_students
    INTO CORRESPONDING FIELDS OF TABLE gt_school
    WHERE zschool = p_zschool.

  CALL FUNCTION 'REUSE_ALV_GRID_DISPLAY'
    EXPORTING
      i_structure_name = 'ZMWB_STUDENTS'
    TABLES
      t_outtab         = gt_school
    EXCEPTIONS
      program_error    = 1
      OTHERS           = 2.
  IF sy-subrc <> 0.
* Implement suitable error handling here
  ENDIF.
ENDFORM.
```

##### 3. 按钮事件（PF_STATUS_SET）

```ABAP
CLEAR gs_events.
  gs_events-name = 'PF_STATUS_SET'.  "设置按钮 
  gs_events-form = 'FM_SET_STATUS'.
  APPEND gs_events TO gt_events.
  
FORM fm_set_status USING rt_extab TYPE slis_t_extab.
  SET PF-STATUS 'STATUS' EXCLUDING rt_extab.
  "设置输出界面的标题，g_lines表示的是输出报表数据条数(DESCRIBE TABLE gt_output LINES g_lines.)
  SET TITLEBAR 'TITLE_1000' WITH g_lines.
ENDFORM.
```

###### 1.单击物料凭证跳转至migo

```ABAP
FORM alv_user_command USING r_ucomm LIKE sy-ucomm
                        		rs_selfield TYPE slis_selfield.
   CASE r_ucomm.
   when '&IC1'.
   	CASE rs_selfield-fieldname .
        WHEN 'MBLNR'. "单击
          CLEAR lv_mblnr.
          lv_mblnr = rs_selfield-value.
          CALL FUNCTION 'MIGO_DIALOG'
            EXPORTING
              i_action            = 'A04'
              i_refdoc            = 'R02'
              i_notree            = 'x'
              i_skip_first_screen = 'x'
              i_okcode            = 'OK_GO'
              i_mblnr             = lv_mblnr
              i_mjahr             = gs_output-mjahr "点击
            EXCEPTIONS
              illegal_combination = 0
              OTHERS              = 0.
        WHEN OTHERS. "双击

      ENDCASE.
    WHEN OTHERS.
  ENDCASE.
```

#### 8. 排序

```ABAP
FORM build_sort .
  CLEAR gs_sort.
  gs_sort-spos = '1'.  "排序的顺序,先按照这个字段排序
  gs_sort-fieldname = 'ZSCHOOL'. "先按照学校编号排序
  gs_sort-up = 'X'.  "按照升序排
  APPEND gs_sort TO gt_sort.

  CLEAR gs_sort.
  gs_sort-spos = '2'.  "排序的顺序,再按照这个字段排序
  gs_sort-fieldname = 'ZSEX'. "先按照性别排序
  gs_sort-down = 'X'.  "按照升序排
  APPEND gs_sort TO gt_sort.
ENDFORM.
```

#### 9. 筛选功能

```ABAP
FORM build_filter .
  CLEAR gs_filter.
  gs_filter-fieldname = 'ZSEX'.
  gs_filter-sign0 = 'E'. " 不包含
  gs_filter-optio = 'EQ'. "等于
  gs_filter-valuf = ''.  "空值
  APPEND gs_filter TO gt_filter.
ENDFORM.
```

#### 10. LVC_ALV函数调用

```ABAP
FORM build_alv .
  gs_variant-report = sy-repid.
  CALL FUNCTION 'REUSE_ALV_GRID_DISPLAY_LVC'
    EXPORTING
*     I_INTERFACE_CHECK  = ' '
*     I_BYPASSING_BUFFER =
*     I_BUFFER_ACTIVE    =
      i_callback_program = sy-cprog 
*     I_CALLBACK_PF_STATUS_SET          = ' '
*     I_CALLBACK_USER_COMMAND           = ' '
*     I_CALLBACK_TOP_OF_PAGE            = ' '
*     I_CALLBACK_HTML_TOP_OF_PAGE       = ' '
*     I_CALLBACK_HTML_END_OF_LIST       = ' '
*     I_STRUCTURE_NAME   =
*     I_BACKGROUND_ID    = ' '
*     I_GRID_TITLE       =
*     I_GRID_SETTINGS    =
      is_layout_lvc      = gs_layout
      it_fieldcat_lvc    = gt_fieldcat
*     IT_EXCLUDING       =
*     IT_SPECIAL_GROUPS_LVC             =
*     IT_SORT_LVC        =
*     IT_FILTER_LVC      =
*     IT_HYPERLINK       =
*     IS_SEL_HIDE        =
*     I_DEFAULT          = 'X'
      i_save             = 'A'
      is_variant         = gs_variant
      it_events          = gt_events
*     IT_EVENT_EXIT      =
*     IS_PRINT_LVC       =
*     IS_REPREP_ID_LVC   =
*     I_SCREEN_START_COLUMN             = 0
*     I_SCREEN_START_LINE               = 0
*     I_SCREEN_END_COLUMN               = 0
*     I_SCREEN_END_LINE  = 0
*     I_HTML_HEIGHT_TOP  =
*     I_HTML_HEIGHT_END  =
*     IT_ALV_GRAPHICS    =
*     IT_EXCEPT_QINFO_LVC               =
*     IR_SALV_FULLSCREEN_ADAPTER        =
* IMPORTING
*     E_EXIT_CAUSED_BY_CALLER           =
*     ES_EXIT_CAUSED_BY_USER            =
    TABLES
      t_outtab           = gt_student
    EXCEPTIONS
      program_error      = 1
      OTHERS             = 2.
  IF sy-subrc <> 0.
* Implement suitable error handling here
  ENDIF.

ENDFORM.
```

## fieldcat新填充方式

```ABAP
DATA : 		 lv_tabname  TYPE slis_tabname,
           ls_slis_alv TYPE slis_fieldcat_alv,
           ls_fieldcat TYPE lvc_s_fcat,
           lt_slis_alv TYPE slis_t_fieldcat_alv.
    DATA : lv_field    TYPE string,
           lv_date     TYPE sy-datum,
           lv_date_tmp TYPE sy-datum,
           lv_index    TYPE i,
           l_col       TYPE i.

    lv_tabname = lv_tname.


    CALL FUNCTION 'REUSE_ALV_FIELDCATALOG_MERGE'
      EXPORTING
        i_program_name         = sy-repid
        i_structure_name       = lv_tabname "结构名"
        i_inclname             = sy-repid
        i_bypassing_buffer     = 'X'  " 清除缓存"
      CHANGING
        ct_fieldcat            = lt_slis_alv[] "存放数据内表"
      EXCEPTIONS
        inconsistent_interface = 1
        program_error          = 2
        OTHERS                 = 3.
    CHECK sy-subrc = 0.

    REFRESH gt_fieldcat.


    LOOP AT lt_slis_alv INTO ls_slis_alv.
      MOVE-CORRESPONDING ls_slis_alv TO ls_fieldcat.

      MOVE : ls_slis_alv-seltext_l TO ls_fieldcat-scrtext_l,
             ls_slis_alv-seltext_m TO ls_fieldcat-scrtext_m,
             ls_slis_alv-seltext_s TO ls_fieldcat-scrtext_s,
             ls_slis_alv-ddictxt TO ls_fieldcat-colddictxt,
             ls_slis_alv-ddictxt TO ls_fieldcat-selddictxt,
             ls_slis_alv-ddictxt TO ls_fieldcat-tipddictxt,
             ls_slis_alv-ref_fieldname TO ls_fieldcat-ref_field,
             ls_slis_alv-ref_tabname TO ls_fieldcat-ref_table,
             ls_slis_alv-roundfieldname TO ls_fieldcat-roundfield,
             ls_slis_alv-decimalsfieldname TO ls_fieldcat-decmlfield,
             ls_slis_alv-decimals_out TO ls_fieldcat-decimals_o,
             ls_slis_alv-text_fieldname TO ls_fieldcat-txt_field,
             ls_slis_alv-reptext_ddic TO ls_fieldcat-reptext,
             ls_slis_alv-ddic_outputlen TO ls_fieldcat-dd_outlen.
      CASE ls_fieldcat-fieldname.
        WHEN 'ZACTIONID'.
          ls_fieldcat-scrtext_l = ls_fieldcat-scrtext_m
          = ls_fieldcat-scrtext_s = '操作标识'.
        WHEN 'MBLNR' OR 'SMBLN' .
          ls_fieldcat-hotspot = 'X'.
        WHEN 'STATUS' .
          ls_fieldcat-no_out = 'X'.
        WHEN OTHERS.
      ENDCASE.

```

## REUSE_ALV_FIELDCATALOG_MERGE 用法

第一种：根据定义的结构作为入参获取fieldcat

要求结构中的字段都是以LIKE参照，将创建好的结构 赋值给i_internal_tabname参数上

踩坑：使用like定义的结构时，调用该函数 要求所有代码行长度不能高于73 

```ABAP
DATA:BEGIN OF wa_user,
       id      LIKE zmwb_students-zcode,
       name    LIKE zmwb_students-zname,
       age     LIKE zmwb_students-zschool,
       address LIKE zmwb_students-zaddr,
     END OF wa_user,
     it_user LIKE STANDARD TABLE OF wa_user.
     
CALL FUNCTION 'REUSE_ALV_FIELDCATALOG_MERGE'
    EXPORTING
      i_program_name         = sy-repid
      i_internal_tabname     = 'WA_USER'
*     i_structure_name       = ''
      i_inclname             = sy-repid
      i_bypassing_buffer     = 'X'
    CHANGING
      ct_fieldcat            = lt_slis_alv
    EXCEPTIONS
      inconsistent_interface = 1
      program_error          = 2
      OTHERS                 = 3.
```

第二种：se11定义的全局结构 

将在se11定义的结构名称、表、视图 赋值给i_structure_name 参数上

```abap
DATA:lt_slis_alv type SLIS_T_FIELDCAT_ALV.
CALL FUNCTION 'REUSE_ALV_FIELDCATALOG_MERGE'
    EXPORTING
      i_program_name         = sy-repid
      i_structure_name       = 'ZPPS047'
      i_inclname             = sy-repid
      i_bypassing_buffer     = 'X'
    CHANGING
      ct_fieldcat            = lt_slis_alv
    EXCEPTIONS
      inconsistent_interface = 1
      program_error          = 2
      OTHERS                 = 3.
```

获取的返回参数lt_slis_alv就是要赋值给alv函数的fieldcat，返回参数类型为slis_t_fieldcat_alv，如果直接使用则后续使用的alv函数只能是function_alv（REUSE_ALV_GRID_DISPLAY）,不能使用在REUSE_ALV_GRID_DISPLAY_LVC、OOALV的展示函数:



若想使用在后续两种ALV展示函数中，可以参照以下方式,转化后的lt_fcat 类型为lvc_t_fcat 符合后两种ALV—fieldcat 参数需求。

```abap
DATA:lt_fcat TYPE lvc_t_fcat,
		 it_data TYPE TABLE OF char128.
"" slis_t_fieldcat_alv->lvc_t_fcat	 
CALL FUNCTION 'LVC_TRANSFER_FROM_SLIS'
    EXPORTING
      it_fieldcat_alv = lt_slis_alv
    IMPORTING
      et_fieldcat_lvc = lt_fcat
    TABLES
      it_data         = it_data " 好像没啥用 但是必须要给 不给就dump TMD
    EXCEPTIONS
      it_data_missing = 1
      OTHERS          = 2.  
```



## 3.屏幕逻辑流

```
PBO（Process Before Output）：就是在屏幕输出之前做的一些操作，例如：控制按钮的显示和键值、屏幕初始化的操作等 
PAI（Process After Input）：就是在屏幕已经展示出来，在屏幕上做的操作,会触发PAI，例如：点击按钮触发的事件可以在PAI中去定义， 

整体流程如下： 
PBO -> 画面 -> PAI -> PBO -> 画面 -> PAI -> PBO... ... 
```

## USING和CHANGING用法

```ABAP
DATA: 	 i_num1 TYPE I VALUE 10,
      　　i_num2 TYPE I VALUE 20,
      　　i_num3 TYPE I. "0
"引用传递（CALL BY REFERENCE），i_num3会在自程序中发生改变 由最初的0 变为30"
PERFORM CALCULATOR USING i_num1 i_num2 CHANGING i_num3. 

FORM CALCULATOR USING NUM1 NUM2 CHANGING NUM3.
  　　NUM3 = NUM1 + NUM2.
  　　WRITE: / 'NUM1=',NUM1,
              'NUM2=',NUM2,
              'NUM3=',NUM3.


"值传递（CALL BY VALUE）
"在子程序中结束后，i_num1 i_num2 都不发生改变 ，而i_num3会改变
PERFORM CALCULATOR_TWO USING i_num1 i_num2 CHANGING i_num3.
 　　WRITE: / 'I_NUM1=',i_num1,
          　　'I_NUM2=',i_num2,
          　　'I_NUM3=',i_num3.
          　　
FORM CALCULATOR_TWO USING VALUE(NUM1) VALUE(NUM2) CHANGING SUM.
  　　SUM = NUM1 + NUM2.
 　　 NUM1 = NUM1 * NUM2.
 　　 NUM2 = NUM1 * NUM2.
  　　WRITE: / 'NUM1=',NUM1,
           　　'NUM2=',NUM2,
          　　 'SUM=',SUM.
ENDFORM.
```

## 消息类

```ABAP
* 维护
	SE91 
	
* 第一种方式：直接将消息展示出来，不用引入message-id.
	MESSAGE '这是一条信息' TYPE 'S'.
	
* 第二种方式：在REPORT后 引入 message-id.
	MESSAGE E000 .
	
* 第三种方式：消息的内容在内部定义
	MESSAGE TEXT-001 TYPE 'S'.
```

```
消息类型：
A ABORT 中止消息
E ERROR 错误消息
I INFORMATION 对话框消息 弹出对话框
S STATUS 常用的提示消息
W WARNING 警告消息
X EXCEPTION 异常消息，通常是DUMP
```

```
TYPE 和 DISPLAY LIKE
真正起作用的是TYPE的类型，DISPLAY LIKE的类型只是显示效果。
```

## 字符串拼接

```ABAP
* 第一种方式
CONCATENATE c1 '+++' c2  INTO c4.

* 第二种方式
c3 = c1 && c2.

* 第三中方式
lv_where = |{ lv_where } { ' AND MATNR = @IS_DATA-MATNR' }|.
"大括号内 括号符号和变量之间存在space" 
```

拼接C1 和 C2 以 sep("-")进行分割开

```abap
DATA: C1(10)  VALUE  'Summer',
      C2(10)   VALUE  'holiday ',
      C3(30),
      SEP(3)  VALUE ' - '.
CONCATENATE C1 C2 INTO C3 SEPARATED BY SEP.

结果为： Summer - holiday
```

## ||、{}、&&的使用

```abap
"|| 里面内容除{}内的原样输出，包括空格"
"|| 里面可以使用{}引入ABAP代码"
DATA(lv_str01) = |有空格: { gt_test[ name = 'XLevon' ]-code }|.
DATA(lv_str02) = '没空格: ' && gt_test[ name = 'XLevon' ]-code .
```

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145900608-1536607859.png" alt="image-20231229134240653" style="zoom: 50%;" />

## 字符串截取

```abap
string+4(2).
"从字符串string的第四位开始 往后截取两位"
```



## 字符串替换 REPLACE

```
replace a in b with 字符串.
```

```abap
DATA(lv_string1) = 'XLevon'.
DATA(lv_string2) = 'Leo'..
DATA(lv_string3) = replace(   val  = lv_string1
                              sub  = lv_string1+1(2)
                              case = abap_true
                              with = to_upper( lv_string2 ) "如果发现sta,用lv_string2的大写替换
                              occ = 1   "val中只有1处替换
                            ).
```

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145901001-284685553.png" alt="image-20231229134904150"  />



```abap
DATA(lv_str) = 'AAA'.
DATA(lv_str2) = 'BABAAADDD'.

FIND lv_str IN lv_str2.
IF sy-subrc = 0.
  out->write( '存在' ).
  REPLACE lv_str in lv_str2 WITH 'HHH'.
  out->write( lv_str2 ).
ELSE.
  out->write( '不存在' ).
ENDIF.
```

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145901201-1001077465.png" alt="image-20240105164518140" style="zoom: 50%;" />

## 图标添加

```ABAP
* 在初始界面添加图标
SELECTION-SCREEN:FUNCTION KEY 1.
INITIALIZATION.
  sscrfields-functxt_01 = icon_export && '下载模板'.
```

## 判断内表里是否存在某个数据

```ABAP
IF gt_stu is not INITIAL.
"判断gt_stu表里是否存在zname字段值为‘小黑’的这条信息
  IF line_exists( gt_stu[ zname = '小hei'] ).
    WRITE:/ 'AAAAAAAAAA'.
    ELSE.
      WRITE:/ 'BBBBBB'.
  ENDIF.
ENDIF.
```

```abap
DATA(lv_exist) = xsdbool( line_exists( gt_test[ code = '20' name = 'XLevon' ] ) ).
"存在是 X 不存在 false"
```

## TRANSPORTING NO FIELDS

```ABAP
"TRANSPORTING NO FIELDS

READ TABLE gt_output TRANSPORTING NO FIELDS WITH KEY msgty = 'E'.
" 判断gt_output中是否有消息类型为E的行

"用来判断表中是否存在某个数据，一般只是判断，对数不进行处理，通常后面跟 sy-subrc进行后续操作。
```

## OO事件

```ABAP
* 1.事件的定义类 (在类1中)
" 事件
"EVENTS 事件名 EXPORTING VALUE(ps_school) LIKE gs_school.
EVENTS data_exist EXPORTING VALUE(ps_school) LIKE gs_school.
    
* 2. 事件的触发
IF sy-subrc = 0.
	RAISE EVENT data_exist
					EXPORTING
						ps_school = gs_school.
ENDIF.

* 3. 事件触发后的具体做法（在类2中）
METHODS write_data FOR EVENT data_exist OF lcl_airplane
															IMPORTING ps_school.
* 4. 事件的注册和调用
SET HANDLER event_handler->write_data FOR obj1.
  CALL METHOD obj1->get_school
    EXPORTING
      p_school = '001'.
```

## 跳出循环

```
continue, check, exit、STOP、RETURN.

continue跳过当前循环继续下一次循环

check：如果条件为假，则 CHECK 之后的其余语句将被忽略，系统开始下一个循环。

exit结束整个循环，结束的是当前最近的循环 / exit如果在form中可以结束form程序。

在LOOP循环中，应当尽量避免对当前表进行插入或者填充操作，如果一旦循环终止条件，则出现死循环，要防止进入死循环

RETURN用来退出当前执行的程序块，例如一个FORM、METHOD、报表事件块，不管是否出现在循环(LOOP)中，RETURN都会退出当前执行的程序块，而不仅仅是退出循环（如果是在Form、METHOD中，只会退出Form、METHOD，不会退出Form、METHOD被调用所在的报表事件块，即退Form、METHOD后继续向被调用点后面执行）

INITIALIZATION中的STOP会导致跳转到AT SELECTION-SCREEN OUTPUT事件块；
如果STOP在AT SELECTION-SCREEN OUTPUT块里，则只是退出当前块（STOP后面语句不执行而已），仅接着是显示选择屏幕；
AT SELECTION-SCREEN [ON]…选择屏幕事件块中的STOP也只是退出当前事件块，继续后面的事件块；
另外，即使STOP在循环中,还是在FORM，METHOD，也是直接从被调用的点退出所在事件块，而不仅仅只退出当前循环、FORM、METHOD，这与直接在事件块中的效果是一样的；
```

## FOR ALL ENTRIES IN使用细节

```ABAP
" 查库存数据 根据物料和库存地点 分别在1021 和 1023 上查询
SELECT matnr, werks, lgort, labst INTO TABLE @DATA(lt_mard)  
    FROM mard
    FOR ALL ENTRIES IN @lt_data
    WHERE matnr = @lt_data-matnr
    AND werks IN ('1021','1023')  "@lt_data-werks
    AND lgort = @lt_data-lgort.
   "这样的做法就是参照lt_data的条件去分别在工厂为1021 和 1023两个场子下去查询mard
   "比如 lt_data 只有一条数据 工厂为1021 该段代码表示 这条数据的其他条件不变 只是工厂换成了1021的一条和1023的一条 分别去作为条件去mard表中查询。
```

## MES物料号 转为SAP物料号

```ABAP
DATA:lv_zsfmat TYPE zmmt003-zsfmat,
     lv_zsfname TYPE zmmt003-zsfname.
LOOP AT it_data INTO ls_data.
    lv_zsfmat = ls_data-matnr. 
    CALL FUNCTION 'ZMMFM013'   " 将mes物料转化为SAP物料
      EXPORTING
        i_zsfname = lv_zsfname  "第三方软件名称"
        i_zsfmat  = lv_zsfmat   "MES物料号"
        i_werks   = ls_data-werks 
      IMPORTING
        e_matnr   = ls_data-matnr.  "输出查询后的MES物料号"
    APPEND ls_data TO lt_data.
    CLEAR ls_data.
  ENDLOOP.
  CLEAR lv_zsfmat.
```

## 查询条件中嵌套条件/子查询

```ABAP
SELECT
      matnr,
      werks,
      lgort,
      charg,
      clabs,
      cinsm,
      cspem
      INTO TABLE @lt_mchb
      FROM mchb
      FOR ALL ENTRIES IN @lt_data
      WHERE matnr = @lt_data-matnr
        AND werks = @lt_data-werks
        AND lgort = @lt_data-lgort
        AND EXISTS (
                      SELECT
                        charg
                        FROM zmmt007
                        WHERE  matnr = @lt_data-matnr
                           AND werks = @lt_data-werks
                           "AND lgort = @lt_data-lgort
                           AND zbatch = @lt_data-zbatch
                           AND charg = mchb~charg
                           AND zscbs = ' '
                      ).
```

## JOIN 内表

```ABAP
" 提取交付款申请金额 (付款申请金额（审批中）)
  SELECT SUM( zmmt015~dmbtr ) AS dmbtr,zmmt015~belnr,zmmt015~gjahr
      FROM zmmt015
      JOIN @gt_rbkp AS a
      ON zmmt015~belnr = a~belnr
      AND   zmmt015~gjahr = a~gjahr
      WHERE   zspzt <> 'Z5'
      AND     zspzt <> 'Z3'
      GROUP BY zmmt015~belnr ,zmmt015~gjahr
      INTO TABLE @DATA(lt_015_1).
```

## 内表分组处理

```abap
"在内表gt_vbrp中以vbeln字段为分组条件 升序分组 <key>为分组条件信息"
LOOP AT gt_vbrp INTO DATA(ls_data) GROUP BY ( vbeln = ls_data-vbeln ) ASCENDING ASSIGNING FIELD-SYMBOL(<key>).
    LOOP AT GROUP <key> ASSIGNING FIELD-SYMBOL(<fs_key>).
      lv_sum += <fs_key>-netwr_bb.
      lv_sum1 += <fs_key>-mwsbp_bb.
    ENDLOOP.
```

```abap
  LOOP AT gt_vbrp ASSIGNING FIELD-SYMBOL(<fs_vbrp1>).
    lv_last_vbeln = <fs_vbrp1>-vbeln.
    lv_sum += <fs_vbrp1>-netwr_bb.
    lv_sum1 += <fs_vbrp1>-mwsbp_bb.
    AT END OF vbeln.  "在处理某个字段（vbelnr）时，在该字段发生变化时执行一次处理逻辑"
      SELECT SINGLE vbeln,netwr,mwsbk,kurrf INTO @DATA(ls_vbrk1) FROM vbrk WHERE vbeln = @lv_last_vbeln.
      IF sy-subrc = 0.
        IF lv_sum < 0.
          lv_result =  ls_vbrk1-netwr *  -1 * ls_vbrk1-kurrf - lv_sum .
          lv_result1 = ls_vbrk1-mwsbk * -1 * ls_vbrk1-kurrf - lv_sum1.
        ELSE.
          lv_result =  ls_vbrk1-netwr * ls_vbrk1-kurrf - lv_sum.
          lv_result1 = ls_vbrk1-mwsbk * ls_vbrk1-kurrf - lv_sum1.
        ENDIF.
        
        READ TABLE gt_vbrp INTO DATA(ls_data2) WITH KEY vbeln = lv_last_vbeln.
        ls_data2-netwr_bb = ls_data2-netwr_bb + lv_result.
        ls_data2-mwsbp_bb = ls_data2-mwsbp_bb + lv_result1.
        ls_data2-kzwi1_bb = ls_data2-netwr_bb + ls_data2-mwsbp_bb.
        MODIFY TABLE gt_vbrp FROM ls_data2 TRANSPORTING netwr_bb mwsbp_bb kzwi1_bb .
        CLEAR:ls_vbrk1,lv_result,lv_result1.
      ENDIF.
    ENDAT.
  ENDLOOP.
```

## at end of ... endat. 

AT END OF" 是一种 SAP ABAP 语言的控制结构，用于在循环结束时执行一些特定的操作。它通常与 "LOOP AT" 或 "WHILE" 等循环语句结合使用。

```abap
DATA: itab TYPE STANDARD TABLE OF t001,
      total TYPE i.
LOOP AT itab INTO wa.
  total = total + wa-field1.
  " ... 在此处执行循环体代码 ...
  AT END OF.
    WRITE: / 'Total:', total.
  ENDAT.
ENDLOOP.
```

​		在这个例子中，我们定义了一个变量 total 来存储合计值。在循环中，每一行的 field1 值都会被加到 total 中。当循环结束时，"AT END OF" 中的 WRITE 语句会输出合计值到屏幕上。

## Clear Refresh Free

内表：如果使用有表头行的内表，**CLEAR** **仅清除表格工作区域**。

要重置整个内表而不清除表格工作区域，使用**REFRESH**语句或CLEAR 语句**CLEAR** <itab>[].

REFRESH加不加中括号都是只清内表，另外REFRESH是专为清内表的，不能清基本类型变量，但CLEAR可以

以上都不会释放掉内表所占用的空间，如果想初始化内表的同时还要**释放所占用的空间**，请使用：**FREE** <itab>.

## 获取内表行数

方式1：通过获得内部表的属性，将内部表行数赋值给n，n为I型变量

```abap
DESCRIBE TABLE itab LINES n.
```

方式2：

```abap
n = lines( itab ).
```

内表满足条件行索引

```abap
DATA(lv_index) = line_index( gt_test[ code = '20' name = 'XLevon' ] ).
```

## COND/SWITCH

COND 相当于 if ... else...

```abap
lv_str = COND #( WHEN lv_str IS INITIAL THEN 'new value' 
							   ELSE lv_str ).
```

```abap
DATA(lv_indicator) = 7.
DATA(lv_day) = COND char10( WHEN lv_indicator = 1 THEN 'MONDAY'
                            WHEN lv_indicator = 2 THEN 'TUESDAY'
                            WHEN lv_indicator = 3 THEN 'WEDNESDAY'
                            WHEN lv_indicator = 4 THEN 'THURSDAY'
                            WHEN lv_indicator = 5 THEN 'FRIDAY'
                            WHEN lv_indicator = 6 THEN 'SATURDAY'
                            WHEN lv_indicator = 7 AND sy-langu EQ 'E' THEN 'SUNDAY'
                            WHEN lv_indicator = 7 AND sy-langu EQ 'F' THEN 'DIMANCHE'
                            WHEN lv_indicator = 7 AND sy-langu EQ '1' THEN '星期天'
                            ELSE '404' && '-ERROR'
                           ).
CALL METHOD cl_demo_output=>display( lv_day ).
```

SWITCH 相当于 case ... when...

```abap
  lv_str = SWITCH #( n WHEN 1 THEN 'A'
                     WHEN 2 THEN 'B' ).
```

```abap
DATA(lv_indicator) = 1.
DATA(lv_day) = SWITCH char10( lv_indicator
                              WHEN 1 THEN 'MONDAY'
                              WHEN 2 THEN 'TUESDAY'
                              WHEN 3 THEN 'WEDNESDAY'
                              WHEN 4 THEN 'THURSDAY'
                              WHEN 5 THEN 'FRIDAY'
                              WHEN 6 THEN 'SATURDAY'
                              WHEN 7 THEN 'SUNDAY'
                              ELSE '404' && '-ERROR'
                             ).
CALL METHOD cl_demo_output=>display( lv_day ).
```

## select内表

1.当内表作为数据库底表查询时，一定要取别名。

2.当内表作为数据库底表查询时，不能与 FOR ALL ENTRIES IN 一起使用。

3.如果无主键相连，并且数据量巨大，建议使用FOR ALL ENTRIES IN （运行时间会短一些）

```abap
SELECT * FROM zmwb_students INTO TABLE tab .

SELECT zname
  FROM @tab AS a
  INTO TABLE @DATA(lt_tab).
  
cl_demo_output=>display( lt_tab ).
```

## VALUE赋值

```abap
TYPES: BEGIN OF TY_DATA,
         MATNR TYPE MARA-MATNR,
         MTART TYPE MARA-MTART,
         MATKL TYPE MARA-MATKL,
         TEXT1 TYPE CHAR50
       END OF TY_DATA.
       
DATA lt_data TYPE TABLE OF ty_data.
"表赋值"
lt_data = VALUE #( ( matnr = 'Material-001'
                     mtart = 'WATR'
                     matkl = '1020'
                     text1 = 'First Material' )
                   ( matnr = 'Material-002'
                     mtart = 'WATR'
                     matkl = '1030'
                     text1 = 'Second Material' ) ).
                     
"若要继续添加数据 不能缺少 BASE lt_data  一旦缺少 之前的数据将被清空"
lt_data = VALUE #( BASE lt_data ( matnr = 'Material-003'
                     mtart = 'WATR'
                     matkl = '1020'
                     text1 = 'First Material' ) ).
```

```abap
"参照结构赋值"
TYPES: BEGIN OF ty_person,
         code TYPE char4,
         name TYPE char20,
       END OF ty_person,
       tty_person TYPE TABLE OF ty_person.
       
DATA(gw_person) = VALUE ty_person( code = '100' name = 'Alex' ).
DATA:g1 LIKE gw_person.
g1 = VALUE #( code = '200' name = 'Java' ).
       
g1 = VALUE #( BASE g1 code = '201' ). " 更细内表字段 没有base就会被覆盖"
```

## CORRESPONDING

```abap
lt_table = CORRESPONDING #( lt_data ).
```

```abap
""在 MAPPING 语句中，需要注意两边的字段类型，以免类型不兼容而导致程序 dump""
lt_table = CORRESPONDING #( lt_data  
                            MAPPING matnr_c = matnr  "不同字段映射 lt_table-matnr_c = lt_data-matnr"
                            EXCEPT  matkl ). "字段不传值"
```

```abap
"当lt_table本身有值 又要从其他表添加值进来 使用BASE ( lt_table )" 
"EXCEPT matkl：要传值内表中的matkl 字段值不接收"
lt_table = CORRESPONDING #( BASE ( lt_table ) lt_data EXCEPT matkl ).

"同理 结构赋值也是"
ls_table = CORRESPONDING #( BASE ( ls_table ) ls_data EXCEPT matkl ).

ls_table = CORRESPONDING #( BASE ( ls_table ) ls_data  ).
```

## CONV

conv 是 ABAP 7.4 及更高版本中引入的一种新语法，用于将字符串、数字、日期等类型的数据进行转换。

```abap
"conv 语法的基本格式为"
DATA(result) = CONV type(data).
"其中，result 是要转换后的结果，type 是要转换到的数据类型，data 是要转换的原始数据。"
```

```abap
"将字符串转换为整数"
DATA(num) = CONV i( '123' ).

"将字符串转换为日期"
DATA(date) = CONV d( '20220101' ).
```

## 类创建

```ABAP
  DATA:lo TYPE REF TO zcl_zmmt007. "创建zcl_zmmt007类的类型"
  lo = NEW zcl_zmmt007( ).					"创建实例 之后才可以调用类的方法等操作"
"或"
  DATA(lo) = new ZCL_ZMMT007().   "参照类直接创建类的实例"
```

### 类：Returning参数

```abap
DATA(lo) = NEW zcl_xiaomi_methods( ).
squan = lo->covert_to_float( '0.003' ).
```

covert_to_float中只有一个输入参数，且是必输的 还有一个Returning类型返回值

RETURNING ：用来替代EXPORTING、CHANGING，不能同时使用。定义了一个形式参数 r 来接收返回值，并且只能是值传递

### 类：方法调用

```ABAP
 " meth( )
"此种方式仅适用于没有输入参数（IMPORTING）、输入\输出参数（CHANGING）、或者有但都是可选的、或者不是可选时但有默认值也可"
```

```abap
"meth( a )"
"此种方式仅适用于只有一个必选输入参数（IMPORTING）（如果还有其他输入参数，则其他都为可选，或者不是可选时但有默认值也可），
"或者是有多个可选输入参数（IMPORTING）（此时没有必选输入参数情况下）的情况下但方法声明时通过使用PREFERRED PARAMETER选项指定了其中某个可选参数为首选参数（首选参数 即在使用meth( a )方式传递一个参数进行调用时，通过实参a传递给设置为首选的参数）"
```

```abap
"meth( p1 = a1 p2 = a2 ... )"
" 此种方式适用于有多个必选的输入参数（IMPORTING）方法的调用（其它如CHANGING、EXPORTING没有，或者有但可选），如果输入参数（IMPORTING）为可选，则也可以不必传
```

## 去除表中重复内容

```ABAP
"采用sort排序 然后DELETE ADJACENT DUPLICATES（先排序 才能使用该语法）
SORT lt_selrow BY row_id.
DELETE ADJACENT DUPLICATES FROM lt_selrow COMPARING row_id. "删除row_id重复的行数据"
DELETE ADJACENT DUPLICATES FROM lt_selrow . 								"删除全部字段都重复的行数据"
```

## 无法使用FOR ALL ENTRIES IN取数时

```ABAP
 SELECT zhtbh,buzei,invoice_no,dmbtr,dmbtr_tax,zmmt032~waers,uuid 
 FROM zmmt032  
 JOIN @lt_rbkp AS a 
 ON zmmt032~uuid = a~zuonr
 WHERE
     invoice_no IN @s_in_no
 AND zhtbh      IN @s_zhtbh
 INTO TABLE @DATA(lt_zmmt032).
```

## ?=

```abap
lr_protocol_messageid ?= lr_protocol.
```

当lr_protocol不为空时 才会将值赋值给lr_protocol_messageid

## 去除空格

作用：去掉字符串中的前面和后面的空格，如果指定NO-GAPS，则去掉字符串中的所有空格。
常用场合：获得字符串的精确长度，用于判断。

```ABAP
CONDENSE 字符串 NO-GAPS.

"去掉字符串中左边或右边的空格：
 SHIFT string LEFT DELETING LEADING SPACE.
 SHIFT string right DELETING LEADING SPACE.
```

## 字符串长度计算

```abap
"方法1
DATA(lv_len) = cl_abap_list_utilities=>dynamic_output_length( gs_out-zyear ).	
"方法2
DATA(lv_len) = STRLEN( gs_out-zyear ).
```

## 事务码处理和查看

se93

## 查看内表中是否有不为空的某行数据

```abap
" 在gt_output内表中查找 该工厂、内部批次、物料 且 批次不为空的 一旦找到就退出循环 
" 并将改行的批次赋值给<fs_output>-charg"
LOOP AT gt_output INTO ls_output
      WHERE werks = <fs_output>-werks
      AND matnr = <fs_output>-matnr
      AND zbatch = <fs_output>-zbatch
      AND charg <> ''. " 批次不为空
        EXIT.
ENDLOOP.
 IF sy-subrc = 0.
   <fs_output>-charg = ls_output-charg.
endif.
```

## 查询zmmt007表时 要注意删除标识符（zscbs）和标签分类（zbqfl）

```abap
SELECT COUNT(*) FROM zmmt007 WHERE matnr = ls_zmmt007-matnr
                                    AND werks = ls_zmmt007-werks
                                    AND charg = ls_zmmt007-charg
                                    AND zbqfl = 'E'
                                    AND zscbs = ''.
```

## 插入zmmt007表 常用函数

ZCL_ZMMT007->insert_by_z15

## TRANSPORTING NO FIELDS WITH  KEY

用于读取内表的时候，只是判断该内表中是否存在该条件数据 不需要读取到工作区中。

```abap
READ TABLE lt_menge_sum_gr TRANSPORTING NO FIELDS WITH  KEY aufnr = lw_maufnr-aufnr
                                                                    matnr = lw_maufnr-matnr
                                                                    charg = lw_menge_rel-charg .
  IF sy-subrc = 0.
     " do something "
  ENDIF.
```

## 选择屏幕界面上添加按钮

第一种方式

```ABAP
*初始化
INITIALIZATION.
  functxt-icon_id   = icon_warehouse.  “ 按钮图标
  functxt-quickinfo = TEXT-014.   "按钮信息文本"
  functxt-icon_text = TEXT-014.   "按钮信息文本"
  sscrfields-functxt_01 = functxt.	" "
```

第二种方式

```abap
FORM frm_init.
  PERFORM frm_create_icon USING 'ICON_SAVE_AS_TEMPLATE' TEXT-f03
                       CHANGING sscrfields-functxt_01.

  PERFORM frm_create_icon USING 'ICON_MATERIAL' TEXT-f04
                     CHANGING sscrfields-functxt_02.
  PERFORM frm_create_icon USING 'ICON_VAL_QUANTITY_STRUCTURE' TEXT-f05
                   CHANGING sscrfields-functxt_03.
ENDFORM.                    " FRM_INIT

*&---------------------------------------------------------------------*
*&      FORM  FRM_CREATE_ICON
*&---------------------------------------------------------------------*
*       创建 图标 文本
*----------------------------------------------------------------------*
FORM frm_create_icon USING VALUE(i_icon_name) TYPE c
                           VALUE(i_text) TYPE c
                  CHANGING VALUE(o_icon) TYPE c.
  CLEAR o_icon.
  CALL FUNCTION 'ICON_CREATE'
    EXPORTING
      name   = i_icon_name
      text   = i_text
      info   = i_text
    IMPORTING
      result = o_icon
    EXCEPTIONS
      OTHERS = 0.
ENDFORM.                    "FRM_CREATE_
```

## PACKAGE SIZE

内表 每读取10行数据开始进行处理 

使用PACKAGE SIZE 需要使用 select ... endselect

``` abap
SELECT * FROM spfli INTO TABLE @DATA(lt_data) PACKAGE SIZE 10.
  WRITE:/ '循环读取10条数据'.

  LOOP AT lt_data INTO DATA(ls_data).
    WRITE:/ ls_data-carrid ,ls_data-connid ,ls_data-airpfrom.
  ENDLOOP.
ENDSELECT.
```

## 弹窗(确认弹窗、填写值弹窗)

 确认弹窗代码

```abap
"弹出确认框
  DATA:lv_yn TYPE STRING.
   CALL FUNCTION 'POPUP_TO_CONFIRM_STEP'
    EXPORTING
      titel          = '重要提示'
      textline1      = '你确定要这个么？'
      cancel_display = 'X'  "space 不显示cancel按钮，'X'是显示取消按钮
    IMPORTING
      answer         = lv_yn.  "确定= J  否=N  取消 = A
  WRITE :lv_yn.
```

## F4搜索帮助 （OOALV版本）

第一步：在类中定义定义事件

```abap
" f4 搜索帮助
    METHODS: handle_onf4 FOR EVENT onf4 OF cl_gui_alv_grid
      IMPORTING e_fieldname
                es_row_no
                er_event_data
                sender .
```

第二步：在类中实现方法

```abap
METHOD handle_onf4 .
    PERFORM fm_handle_onf4 USING e_fieldname
                                 es_row_no
                                 er_event_data
                                 sender .

  ENDMETHOD.
```

第三步：实现方法

```abap
FORM fm_handle_onf4  USING    e_fieldname   TYPE lvc_fname
                              es_row_no     TYPE lvc_s_roid
                              er_event_data TYPE REF TO cl_alv_event_data
                              sender        TYPE REF TO cl_gui_alv_grid.
                              
IF e_fieldname = 'ZSYXMBH'. "我们自定义搜索的字段名
    READ TABLE gt_alv INTO gs_alv INDEX es_row_no-row_id. "获取搜索帮助那行"
    CHECK sy-subrc = 0.
    CALL FUNCTION 'F4IF_INT_TABLE_VALUE_REQUEST'
      EXPORTING
        retfield        = 'ZSYXMBH'  "搜索帮助选择值后 返回填充的字段"
        value_org       = 'S'  " 返回的是结构
      TABLES
        value_tab       = lt_zqmi003 " 取值的范围
        return_tab      = lt_ret_tab " 选择值后 该值的信息存放的表
      EXCEPTIONS
        parameter_error = 1
        no_values_found = 2
        OTHERS          = 3.
IF sy-subrc = 0.
      READ TABLE lt_ret_tab ASSIGNING FIELD-SYMBOL(<fs>) INDEX 1.
      IF sy-subrc = 0.
        gs_alv-zsyxmbh = <fs>-fieldval. " 将选择的值填充alv
        MODIFY gt_alv FROM gs_alv INDEX es_row_no-row_id TRANSPORTING zsyxmbh.
      ENDIF.
```

第四步：注册事件

```abap
DATA: lt_f4 TYPE lvc_t_f4,
        ls_f4 TYPE lvc_s_f4.
        
" 注册搜索帮助事件    
CREATE OBJECT :event_handle.
SET HANDLER :event_handle->handle_onf4 FOR g_grid1 .
CLEAR ls_f4.
ls_f4-fieldname = 'ZSYXMBH'.
ls_f4-register = 'X'.
ls_f4-chngeafter = 'X'.
APPEND ls_f4 TO lt_f4

CALL METHOD g_grid1->register_f4_for_fields
  EXPORTING
    it_f4 = lt_f4.
```

## SMARTFORMS

在程序中调用设计好的SMARTFORMS 表单函数 

```abap
 CALL FUNCTION 'SSF_FUNCTION_MODULE_NAME'
    EXPORTING
      formname           = 'ZFISF002'
    IMPORTING
      fm_name            = w_sfname
    EXCEPTIONS
      no_form            = 1
      no_function_module = 2
      OTHERS             = 3.
```

## 调用外部子程序

调用zmwb_test程序中的子程序frm_sum

```abap
DATA: lv_num TYPE i.
PERFORM frm_sum(zmwb_test) USING 10 20 lv_num IF FOUND.
WRITE:/ lv_num.
```

## CASE WHEN

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145901572-1774786806.png" alt="image-20231130155621300" style="zoom:33%;" />

```abap
SELECT
    a~zcode,
    a~zname,
    CASE a~zcode WHEN '00001' THEN 'X' 
    						 ELSE @abap_false END AS del_flag
    FROM zmwb_students AS a
    INTO TABLE @DATA(gt_data).
```

## 交货单上锁

方法一：

1、创建锁对象：TCODE：se11.选择最下面的lock object

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145901911-818403078.png" alt="image-20231205091927634" style="zoom:33%;" />

2、自建的锁对象以EZ或者EY开头，新建完之后可以看到三个标签页。Attributes(属性),Tables(表),Lock Parameter（锁参数）. 

3、如果是接口对表进行操作，注意需要选上allow RFC.如图：

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145902246-397412120.png" alt="image-20231205092300362" style="zoom:33%;" />

```
锁的三种模式
模式E：当更改数据的时候设置为此模式。 （一般情况下，E就可以了）
模式S：本身不需要更改数据，但是希望显示的数据不被别人更改。 
模式X：和E类似，但是不允许累加，完全独占。 
```

LOCK parameters里面默认的参数时表的主键，这样因为可以唯一的确定表的一行，通常情况下不用修改。

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145902562-101051384.png" alt="image-20231205092528752" style="zoom:33%;" />

在激活之后，会产生两个function module，一个用来对对象进行锁定，另一个是释放对象。

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145902948-792530849.png" alt="image-20231205092629370" style="zoom:33%;" />

添加锁代码

```abap
 LOOP AT lt_alv ASSIGNING FIELD-SYMBOL(<fs>).
    CALL FUNCTION 'ENQUEUE_EZ_ZMMT107'
      EXPORTING
        mode_zmmt107   = 'E'
        mandt          = sy-mandt
        guid           = <fs>-guid
        zdocnum        = <fs>-zdocnum
        zwmsid         = <fs>-zwmsid
        x_guid         = ' '
        x_zdocnum      = ' '
        x_zwmsid       = ' '
        _scope         = '2'
        _wait          = ' '
        _collect       = ' '
      EXCEPTIONS
        foreign_lock   = 1
        system_failure = 2
        OTHERS         = 3.
    IF sy-subrc <> 0.
      CONTINUE.
    ENDIF.
  ENDLOOP.
```

**锁定**
  当用逻辑锁来锁定表条目的时候，系统会自动向LOCK TABLE中写入记录。 
  当调用设置锁的FM时，LOCK PARAMETERS如果没有指明，系统会锁定整个表。
   当然，LOCK PARAMETER：CLIENT有点特殊，如果不指定，默认是SY-MANDT；
        																			如果指定相应的CLIENT，会锁定对应CLIENT上的相应的表记录；
																					如果设置为SPACE，则锁定涉及所有的CLIENT。 
  当逻辑锁设置失败后，一般会有两种例外。一个是EXCEPTION：FOREIGN_LOCK，意思是已经被锁定了；
																		   另一个是EXCEPTION：SYSTEM_FAILURE。 
**解锁**

​		有些情况下，程序中设置成功的逻辑锁会隐式的自己解锁。比如说程序结束发生的时候（MESSAGE TYPE为A或者X的时候）;使用语句LEAVE PROGRAM，LEAVE TO TRANSACTION;或者在命令行输入/n回车以后。 
​		在程序的结束可以用DEQUEUE FUNCTION MODULE来解锁（当然如果你不写这个，程序结束的时候也会自动的解锁），这个时候，系统会自动从LOCK TABLE把相应的记录删除。使用DEQUEUE FUNCTION MODULE来解锁的时候，不会产生EXCEPTION。要解开你在程序中创建的所有的逻辑锁，可以用FM：DEQUEUE_ALL. 

上锁的一般步骤 
  先上锁，上锁成功之后，从数据库取数据，然后更改数据，接着更新到数据库，最后解锁。按照这个步骤，才能保证更改完全运行在锁的保护机制下。 

## 程序上锁

```abap
 START-OF-SELECTION.
   CALL FUNCTION 'ENQUEUE_ES_PROG'
      EXPORTING
        mode_trdir     = 'E'
        name           = 'ZMMR016'
        x_name         = ' '
        _scope         = '3'
        _wait          = ' '
        _collect       = ' '
      EXCEPTIONS
        foreign_lock   = 1
        system_failure = 2
        OTHERS         = 3.
    IF sy-subrc <> 0.
      MESSAGE ID sy-msgid TYPE 'S' NUMBER sy-msgno
                        WITH sy-msgv1 sy-msgv2 sy-msgv3 sy-msgv4 
                        DISPLAY LIKE sy-msgty.

      LEAVE LIST-PROCESSING .

    ENDIF.
```



## fieldcat简便版

```abap
DATA:it_fieldcat TYPE lvc_t_fcat.
CLEAR it_fieldcat.
  CALL FUNCTION 'LVC_FIELDCATALOG_MERGE'
    EXPORTING
      i_structure_name   = 'ZSMMI012' " 结构名"
      i_bypassing_buffer = 'X' " 清理缓存 不然还是显示修改结构之前的字段
    CHANGING
      ct_fieldcat        = it_fieldcat.

  IF sy-subrc <> 0.
* Implement suitable error handling here
  ENDIF.
```

除了上述的i_structure_name 参数存放 se11创建的结构名外，
对于没有在SE11 定义结构的，他还有个参数可用： I_INTERNAL_TABNAME 



## 数据库表 增加查询视图和事务码

1、SE11给表创建表维护生成器就是增加查询视图

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145903546-663479237.png" alt="image-20231206093005543" style="zoom:33%;" />

2、给表维护视图增加事务码

se93创建事务码（选择‘Transaction with parameters（parameter transaction）’）

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145903858-1521528138.png" alt="image-20231206093125504" style="zoom: 25%;" />

UPDATE = 'X'

VIEWNAME = '要维护的表'



## 邮件发送函数

```abap
DATA:lt_field              TYPE TABLE OF zxxsfield,
       lw_field              TYPE zxxsfield,
       lt_xml_table_forecast TYPE TABLE OF solix,
       lv_sender             TYPE adr6-smtp_addr VALUE 'SAP@EASPRING.COM',
       lv_title              TYPE so_obj_des,
       lt_mail_text          TYPE  bcsy_text,
       lw_mail_text          TYPE soli,
       lt_reciver            TYPE bcsy_smtpa,
       lw_reciver            TYPE ad_smtpadr,
       lt_carboncopy         TYPE bcsy_smtpa,
       lt_bcarboncopy        TYPE bcsy_smtpa,
       lt_attachments        TYPE zcommtt001,
       lw_attachments        TYPE zcomms001,
       lw_return             TYPE zcomms002.
       
LOOP AT it_fieldcat INTO DATA(wa_fieldcat). "it_fieldcat展示alv时设置的格式"
    MOVE-CORRESPONDING wa_fieldcat TO lw_field.
    APPEND lw_field TO lt_field.
    CLEAR:lw_field.
ENDLOOP.

" 获取发送的邮箱
SELECT  email_address FROM zmmt025 INTO TABLE lt_reciver WHERE werks = '1021'.
  
"获取附件
CALL FUNCTION 'ZXXFM_CONV_EXCEL'
  EXPORTING
    i_width  = 80
  IMPORTING
    o_data   = lw_attachments-file_binary
  TABLES
    it_field = lt_field "格式"
    it_data  = gt_alv "数据"
    ot_data  = lt_xml_table_forecast.  
        
" 发送邮件
lv_title = 'SAP库存' && sy-datum . "邮件名称"
CONDENSE lv_title NO-GAPS.
lw_mail_text-line = 'SAP库存明细' . "邮件简介"
APPEND lw_mail_text TO lt_mail_text.

lw_attachments-file_type   = 'XLS'."邮件格式"
lw_attachments-file_name   = lv_title .
lw_attachments-file_lagu   = sy-langu.
APPEND lw_attachments TO lt_attachments.
    
CALL FUNCTION 'ZXXFM_SEND_EMAIL'
  EXPORTING
    i_sender      = lv_sender
    i_title       = lv_title
  IMPORTING
    e_return      = lw_return
  TABLES
    t_mail_text   = lt_mail_text
    t_reciver     = lt_reciver
    t_carboncopy  = lt_carboncopy
    t_bcarboncopy = lt_bcarboncopy
    t_attachments = lt_attachments.  
    
IF lw_return-type = 'S' AND sy-batch IS INITIAL.
  MESSAGE |发送成功|  TYPE 'S'.
ENDIF.    
```

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145904329-457946369.png" alt="image-20231207101520748" style="zoom:33%;" />



## 视图更新

https://blog.csdn.net/Pegasus666/article/details/115464711

表维护视图 当自建表插入新的字段后 需要重新删除之前的维护视图 然后建立新的维护视图 在sm30中才会更新具有新字段的维护视图

表修改状态下进入表维护生成器 点击垃圾桶删除旧的维护视图

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145904669-568285190.png" alt="image-20231219144031362" style="zoom:33%;" />

## 外键

外键的作用：限制外键表的数据，比如下列 只有在学生表中有学校编号信息，才能在学校表中创建对应的学校信息

两个透明表zmwb_students表，zmwb_school表，为学生表中字段学校编号字段创建外键，确保要创建外键的字段是主键

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145905046-753413074.png" alt="image-20231220095453356" style="zoom:33%;" />

选中要创建外键的行，点击外键钥匙按钮

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145905457-1103561783.png" alt="image-20231220095553486" style="zoom:33%;" />

跳出界面中 输入检查表中的外键表 回车 系统会自动将两个表中相同字段作为条件关联起来

最后点击复制 完成外键的创建



## 视图

### 维护视图

se11创建维护视图- ZVSTUDENTS

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145905841-1526269002.png" alt="image-20231220141114293" style="zoom: 33%;" />

先输入表ZMWB_STUDENTS 然后点击下面的关系 直接弹出与该表有外键连接的表 选中ZMWB_SCHOOL就把该表代入

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145906208-1499918354.png" alt="image-20231220141232117" style="zoom:33%;" />

视图字段主要是展示要维护的字段

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145906581-1012949052.png" alt="image-20231220141314734" style="zoom:33%;" />

选择条件 查询这些数据的条件

创建完毕后，还要给改维护视图添加表维护生成器 ，这样就可以在SM30对该视图进行数据维护。

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145906980-122258374.png" alt="image-20231220141441618" style="zoom:33%;" />

### 数据库视图

## AMDP

```abap
DATA: lt_spfli TYPE spfli_tab.
DATA(lo) = NEW zamdp_demo_01( ).

*START-OF-SELECTION.
*  IF cl_abap_dbfeatures=>use_features(
*              EXPORTING
*                requested_features = VALUE #( ( cl_abap_dbfeatures=>call_amdp_method )
*                                              ( cl_abap_dbfeatures=>amdp_table_function ) ) ).
*
*    zamdp_demo_01=>get_data_go_back(
*            EXPORTING
*              ip_mandt = sy-mandt
*            IMPORTING
*              et_spfli = lt_spfli ).
*
*    cl_demo_output=>display( lt_spfli ).
*
*  ELSE.
*    cl_demo_output=>display( '警告！当前系统不支持AMDP.' ).
*
*  ENDIF.

zamdp_demo_01=>get_data_go_back
           EXPORTING
              ip_mandt = sy-mandt
            IMPORTING
              et_spfli = lt_spfli ).

cl_demo_output=>display( lt_spfli ).
```

## 常见数据类型

```abap
d、t类型
"d:Date Types:8 characters yyyymmdd
"t:6 characters hhmmss

DATA:l_data   TYPE d,
     l_time   TYPE t.
l_data = sy-datum.
l_time = sy-uzeit.
```

## 前导零

```abap
"要保证全是数字字符串 如果存在非数字字符 补零失败"
ls_number = |{ ls_number ALPHA = IN }|. 
```

```abap
"按指定宽度（18）位置（RIGHT）填充指定字符（0），不管是否含非数字
DATA(lv_matnr3) = |{ lv_matnr1  WIDTH = 18 ALIGN = RIGHT PAD = '0' }|.
```

```abap
"物料标准转换例程，不含非数字，则补前导0至18位
CALL FUNCTION 'CONVERSION_EXIT_MATN1_INPUT'
  EXPORTING
    input        = lv_matnr2
  IMPORTING
    output       = lv_matnr7
  EXCEPTIONS
    length_error = 1
    OTHERS       = 2.
IF sy-subrc <> 0.
* Implement suitable error handling here
ENDIF."
```

## 拆分内表为字符串

```abap
TYPES: BEGIN OF ty_test,
         code TYPE char4,
         name TYPE char10,
       END OF ty_test,
       tty_test TYPE TABLE OF ty_test.
       
DATA: gt_test  TYPE tty_test,
      lt_split LIKE gt_test.
      
gt_test = VALUE #( ( code = '10' name = 'Leo' )
                   ( code = '20' name = 'XLevon' )
                   ( code = '30' name = 'Leo' )
                   ( code = '40' name = 'XLevon' ) ).
DATA(lv_concat_lines) = concat_lines_of( table = gt_test sep = '/' )." 内表拼接为字符串
CONDENSE lv_concat_lines NO-GAPS." 去除空格
```

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145907373-1458986030.png" alt="image-20231229141425207" style="zoom:50%;" />

## 字符串拆分成内表

```abap

SPLIT lv_concat_lines AT '/' INTO TABLE lt_split.    " 字符串拆分成内表
```

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145907626-1815000181.png" alt="image-20231229141446412" style="zoom:50%;" />

## 字符串查找FIND

```abap
DATA(lv_str) = 'AAA'.
DATA(lv_str2) = 'BABAAADDD'.

FIND lv_str IN lv_str2.
IF sy-subrc = 0.
  out->write( '存在' ).
ELSE.
  out->write( '不存在' ).
ENDIF.
```



## 异常捕获

``` abap
TRY.
    DATA(lv_value) = 10 / 0.
    WRITE: / , '10 / 0，触发错误：'.
  CATCH cx_root INTO DATA(exc).
    CASE TYPE OF exc.
      WHEN TYPE cx_sy_arithmetic_error.
        WRITE: 'Arithmetic error' .
      WHEN TYPE cx_sy_conversion_error.
        WRITE:  'Conversion error' .
      WHEN OTHERS.
        WRITE:  'Other error' .
    ENDCASE.
ENDTRY.
```



## 分组循环 GROUP BY

```abap
DATA lt_groupby_name LIKE gt_test.
LOOP AT gt_test INTO DATA(ls_test) GROUP BY ( name = ls_test-name ) ASCENDING INTO DATA(lt_group).
  "组内循环
  LOOP AT GROUP lt_group INTO DATA(ls_group).
    APPEND ls_group TO lt_groupby_name.
  ENDLOOP.

  out->write( lt_groupby_name ).

  CLEAR lt_groupby_name[].
ENDLOOP.
```

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145907892-868230907.png" alt="image-20231229141609215" style="zoom:50%;" />

## 动态内表

```ABAP
1.定义一般类型
FIELD-SYMBOLS <fs_field> TYPE any." 任意变量或者是结构
FIELD-SYMBOLS <fs_field_02> . "和上述效果一致,都是任意类型

FIELD-SYMBOLS <fs_table> TYPE ANY TABLE. " 任意表 但是没有结构

2. 定义完整类型
FIELD-SYMBOLS <fs_char> TYPE c. “ 类型为字符型的
FIELD-SYMBOLS <fs_makt> TYPE makt. “ makt表类型
```

## 变量的分配

### 静态分配

```ABAP
1. 分配变量
DATA: lv_char   TYPE c LENGTH 13 VALUE '安徽'.
ASSIGN lv_char TO <fs_field>.

2. 分配结构体
FIELD-SYMBOLS <fs_zstudents> TYPE zmwb_students. " 内表 具有特定结构
DATA: ls_mwb  TYPE zmwb_students. " 工作区
SELECT SINGLE  * FROM zmwb_students INTO CORRESPONDING FIELDS OF ls_mwb.
ASSIGN ls_mwb TO <fs_zstudents>.

3. 将结构体中的某个字段赋值给 FIELD-SYMBOLS
ASSIGN COMPONENT 'MATNR' OF STRUCTURE lv_makt TO <fs_field>. 

4.* 通过字段的序号来获取
ASSIGN COMPONENT 1 OF STRUCTURE lv_makt TO <fs_field>.

```

### 动态分配

```abap
DATA: ls_char01    TYPE c LENGTH 5 VALUE 'A',
      ls_char02    TYPE c LENGTH 5 VALUE 'B',
      ls_char03    TYPE c LENGTH 5 VALUE 'C',
      ls_char04    TYPE c LENGTH 5 VALUE 'D',
      ls_char05    TYPE c LENGTH 5 VALUE 'E',
      ls_char06    TYPE c LENGTH 5 VALUE 'F',
      ls_char07    TYPE c LENGTH 5 VALUE 'G',

      ls_index     TYPE n LENGTH 2, " 2 位的目的是匹配 01 02 ...

      ls_fieldname TYPE c LENGTH 30. " 放置变量的名称

DO 7 TIMES.
  CLEAR ls_fieldname.
  ls_index = sy-index.
  ls_fieldname = 'ls_char'.
  CONCATENATE ls_fieldname ls_index INTO ls_fieldname.
  ASSIGN (ls_fieldname) TO <fs_field>.   " (ls_fieldname)小括号的方式指向的是字段名对应的值
  WRITE:/ ls_index , ':',<fs_field>.
ENDDO.

* 结构体的动态分配
ls_fieldname = 'matnr'.
ASSIGN COMPONENT ls_fieldname OF STRUCTURE lv_makt TO <fs_field>.

* 动态分配表字段
DATA: gv_name1 TYPE c LENGTH 20 VALUE 'SFLIGHT-CARRID', " SFLIGHT-CARRID 存在的表-字段

FIELD-SYMBOLS: <fs_test1>.

SELECT SINGLE carrid  FROM sflight INTO gv_name1.
WRITE:/ 'gv_name1:', gv_name1.

ASSIGN TABLE FIELD (gv_name1) TO <fs_test1>. " gv_name1 -- 'SFLIGHT-CARRID'       (gv_name1) -- SFLIGHT-CARRID

* 结构体字段分配到field-symble中
DATA: BEGIN OF gs_str,
        col1 TYPE char5 VALUE 'CHINA',
        col2 TYPE char10 VALUE 'BEIJING',
        col3 TYPE char15 VALUE 'TWEIN BUILDING',
      END OF gs_str.

DATA lt_str LIKE TABLE OF gs_str.  " gs_str为 内表lt_str相同结构的工作区
```

## 类型转换

```ABAP
TYPES: BEGIN OF gty_score,
         name  TYPE c LENGTH 4,
         score TYPE n LENGTH 3,
       END OF gty_score.

lv_char = 'abcd100'.

ASSIGN lv_char TO <fs_field> CASTING TYPE gty_score. " 当后续<fs_field> 为不固定类型时,需要使用type来为其指定转换后的类型
```

## ASSIGN赋值

```ABAP
* 常规使用
DATA: string1(10) TYPE c  VALUE '0123456789'.
WRITE:/ string1+5. " 56789  从第5位开始，不包含第五位输出之后的字符

* FIELD-SYMBOLS上的赋值使用
FIELD-SYMBOLS <fs> TYPE any . 
ASSIGN string1+5(*) TO <fs>. " 赋值的时候需要使用（*）来表示第五位之后的所有数字
```

## 显示键值

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145908351-367483075.png" alt="image-20231228154531283" style="zoom:33%;" />

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145908813-1125873587.png" alt="image-20231228154604182" style="zoom:33%;" />

## 1、远程调用RFC程序的实现

```abap
DATA:line LIKE line.
DATA: options1 LIKE rfc_db_opt OCCURS 10 WITH HEADER LINE.
DATA: nametab1 LIKE rfc_db_fld OCCURS 10 WITH HEADER LINE.
DATA: stmp_dokh1 LIKE rfc_db_fld OCCURS 100000 WITH HEADER LINE.

line = 'MATNR = ''A0019010100007'''.
APPEND line TO options1.
CONCATENATE 'AND SPRAS = ''' sy-langu '''' INTO line.
APPEND line TO options1.

nametab1-fieldname = 'MATNR'.
APPEND nametab1.

nametab1-fieldname = 'MAKTX'.
APPEND nametab1.

CALL FUNCTION 'RFC_READ_TABLE'
  DESTINATION 'SAP_RFC1'  "SM59配置 通信连接"
  EXPORTING
    query_table = 'MAKT'
  TABLES
    options     = options1
    fields      = nametab1
    data        = stmp_dokh1.
IF sy-subrc = 0.
  DATA: matnr TYPE matnr,
        maktx TYPE maktx.
        
  LOOP AT stmp_dokh1.
    SPLIT stmp_dokh1 AT space INTO matnr maktx.
    WRITE:/ matnr,maktx.
  ENDLOOP.
ENDIF.
```

## 2、动态获取内表指定字段值

```abap
DATA:lt_2a LIKE t552a.

DATA:l_data   TYPE d,
     l_time   TYPE t,
     l_field  TYPE string,
     l_day(2).
FIELD-SYMBOLS:<field> TYPE any.
DATA:field TYPE string.

SELECT SINGLE * FROM t552a INTO CORRESPONDING FIELDS OF lt_2a
  WHERE zeity = '1'
  AND mofid = '99'
  AND mosid = '65'
  AND schkz = 'NORM'
  AND kjahr = '2011'
  AND monat = '01'.

l_data = '20110101'. "  日期类型

WHILE l_data <= '20110201'.

  l_day = l_data+6(2).
  CONCATENATE 'TPR' l_day INTO l_field.

  ASSIGN COMPONENT l_field OF STRUCTURE lt_2a TO <field>.

  cl_demo_output=>write( '日期' && l_day && '日变量' && <field> ).

  l_data += 1.
ENDWHILE.
cl_demo_output=>display( lt_2a ).
```

## 3、动态表名

```abap
DATA:dref    TYPE REF TO data, " 能接纳任何数据类型的参照物
     tabname TYPE tabname. " 动态表名存储对象

FIELD-SYMBOLS:<row>       TYPE any,
              <component> TYPE any.

tabname = 'SPFLI'.

CREATE DATA dref TYPE (tabname). " 创建一个类型为tabname的动态工作区

ASSIGN dref->* TO <row>. " 参照dref定义一个指针<row>来分配给该工作区

SELECT * FROM (tabname) INTO <row> UP TO 5 ROWS.
  cl_demo_output=>write( <row> ).
ENDSELECT.

*cl_demo_output=>write( ).
cl_demo_output=>display( ).
```

## 动态where条件

```abap
DATA: lt_where TYPE TABLE OF edpline.
DATA:lt_data TYPE TABLE OF spfli.

APPEND 'CARRID = ''TT''' TO lt_where.
APPEND 'OR' TO lt_where.
APPEND 'CARRID = ''AA''' TO lt_where.

SELECT * FROM spfli INTO TABLE lt_data WHERE (lt_where).
.
*cl_demo_output=>write( ).

cl_demo_output=>display( lt_data ).
```

```abap
DATA: lv_where TYPE string.
DATA:lt_data TYPE TABLE OF spfli.

"lv_where = |SPFLI~CARRID = 'TT'  OR SPFLI~CARRID = 'AA' |.
lv_where = |CARRID = 'TT'  OR CARRID = 'AA' |.

SELECT * FROM spfli INTO TABLE lt_data WHERE (lv_where).
.
*cl_demo_output=>write( ).

cl_demo_output=>display( lt_data ).
```

## 创建检验批号

```ABAP
DATA:				 l_qals     TYPE qals,
             l_rmqed    TYPE rmqed,
             l_prueflos TYPE qals-prueflos,
             l_subrc    TYPE sy-subrc,
             insp_msg   TYPE c LENGTH 100.
             
IF <fs_data>-zart = 'Z001' OR <fs_data>-zart = 'Z011'. " 01
          l_qals-lifnr = <fs_data>-lifnr.    " 供应商帐户号
          l_qals-herkunft = '01'.          " 检验批源
          l_qals-ekorg = '1020'.           " 采购组织

        ELSEIF <fs_data>-zart = 'Z042'.  " 89
          l_qals-herkunft = '89'.          "检验批源
        ENDIF.

        l_qals-matnr = <fs_data>-matnr. " 物料
        l_qals-werk  = '1021'. " 工厂
        l_qals-art = <fs_data>-zart.     " 检验类型
        l_qals-losmenge = <fs_data>-losmenge.  " 检验批数量
        l_qals-lmengeist = <fs_data>-losmenge.  " 检验批实际数量
        l_qals-charg = <fs_data>-charg.  " 批次
        l_qals-lagortchrg = '1001'. " 存储地点
        l_qals-ktextlos = <fs_data>-ktextlos.   " 检验批说明

        l_rmqed-dbs_steuer = '01'.
        l_rmqed-dbs_flag   = 'X'.
        l_rmqed-dbs_edunk  = 'X'.
        l_rmqed-dbs_fdunk  = 'X'.
        l_rmqed-dbs_noerr  = 'X'.
        l_rmqed-dbs_nowrn  = 'X'.
        l_rmqed-dbs_nochg  = 'X'.
        l_rmqed-dbs_noauf  = 'X'.

        CALL FUNCTION 'QPL1_INSPECTION_LOT_CREATE'
          EXPORTING
            qals_imp   = l_qals
            rmqed_imp  = l_rmqed
          IMPORTING
            e_prueflos = l_prueflos  "创建的检验批号"
            e_qals     = l_qals
            subrc      = l_subrc.

        IF l_subrc = 0. " 检验批创建成功
          CALL FUNCTION 'QPL1_UPDATE_MEMORY'
            EXPORTING
              i_qals  = l_qals
              i_updkz = 'I'.

          CALL FUNCTION 'QPL1_INSPECTION_LOTS_POSTING'.
          COMMIT WORK AND WAIT.
          IF l_prueflos <> ''.
            CONCATENATE '检验批号:' l_prueflos '创建成功！' INTO insp_msg SEPARATED BY ' '.
            <fs_data>-prueflos = l_prueflos.
            <fs_data>-msgtype = 'S'.
            <fs_data>-message = insp_msg.
          ENDIF.

          CLEAR:l_qals. " 不清空 多条记录时 down掉
```

## JOB



## 调用外部程序

```abap
SUBMIT 程序名 VIA SELECTION-SCREEN.
```

跳转到程序所在选择界面，但是不执行

```abap
SUBMIT rfidcn_acctbln
                 WITH p_bukrs = <fs_alv>-rbukrs
                 WITH p_year = <fs_alv>-gjahr
                 WITH r_hkont = <fs_alv>-racct
                 AND RETURN.
```

跳转程序的选择界面 ，自动填充参数并执行

```abap
SET PARAMETER ID 'BLN' FIELD <fs_alv>-belnr.
      SET PARAMETER ID 'BUK' FIELD <fs_alv>-rbukrs.
      SET PARAMETER ID 'GJR' FIELD <fs_alv>-gjahr.
      CALL TRANSACTION 'FB03' AND SKIP FIRST SCREEN.
```

跳转到事务码FB03并执行并带有参数。使用set parameter id来赋值参数。这种方法使用MEMORY ID。



## 物料段查询

参照表ZPPT055

```abap
data: r_matnr type RANGE OF mara-matnr.
SELECT 'I' AS sign ,
       'CP' AS option,
        zmatd AS low
      INTO TABLE @r_matnr
      FROM zppt055
      WHERE  werks = @<fs>-werks.
    IF sy-subrc = 0 and <fs>-matnr in  r_matnr AND r_matnr[] IS NOT INITIAL.
    endif.
```



## 队列

出站队列似乎无需注册,可以直接执行 队列注册事物代码 SMQS(如图)

入站队列似乎需要注册,才能执行,否则队列一致处于ready状态 队列注册事物代码 SMQR(如图)

1、点击注册->填入队列名称->✅

<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145909243-211398982.png" alt="image-20240401110308931" style="zoom: 33%;" />



<img src="https://img2023.cnblogs.com/blog/2606674/202407/2606674-20240719145909544-171259180.png" alt="image-20240401110143523" style="zoom:50%;" />

2、队列的使用

参考PP400接口

```abap
" 设置队列
    CALL FUNCTION 'TRFC_SET_QIN_PROPERTIES'
      EXPORTING
        qin_name           = 'XINBOUND_PO_CO11N'
      EXCEPTIONS
        invalid_queue_name = 1
        OTHERS             = 2.
    IF sy-subrc = 0.
      CALL FUNCTION 'ZPPFM019' IN BACKGROUND TASK
        EXPORTING
*         IS_REQ   =
          i_sfname = '102'
          is_data  = ls_data.

      COMMIT WORK.
   ENDIF.   
```

3、队列监控

出站队列：SMQ1

出站队列：SMQ2



# sRFC，aRFC，tRFC，qRFC和bgRFC

参考：https://mp.weixin.qq.com/s/8X4iiXBuoGcWxqt1vgo6Gw

远程函数调用（Remote Function Call，以下简称RFC）是一种标准的通信方式

所有RFC类型都通过CPI-C或TCP/IP协议进行传输。它们构成了一种Gateway通信。

同步RFC（Synchronous RFC，sRFC）是最基本的RFC形式。在sRFC调用中，调用者会等待远程被调用者的处理过程。

```abap
" 它的语法形式是：
CALL FUNCTION func DESTINATION dest.
```

异步RFC（Asynchronous RFC，aRFC）类似于tRFC，用户在继续调用会话之前，不需要等待它们的完成。不过，aRFC和tRFC之间也存在几点不同的地方：

- 当调用者开始一个aRFC的时候，被调用的服务器必须可以接收请求。aRFC的参数不会记录在数据库中，而是直接发送给对方服务器。
- aRFC允许用户与远程系统进行交互式对话。
- 调用程序可以从aRFC接收结果。

你可以在当你需要建立和一个远端系统的连接、但是希望在调用RFC后不希望等待结果而是希望继续处理时使用aRFC。aRFC也可以发送给相同的系统。在这种情况下，系统打开一个新的会话（窗口）。你可以在调用对话和被调用会话间切换。使用下面的语句开启一个aRFC：

```abap
CALL FUNCTION Remotefunction STARTING NEW TASK Taskname

DESTINATION ...

EXPORTING...

TABLES ...

EXCEPTIONS...
```

RECEIVE RESULTS FROM FUNCTION Remotefunction 用于一个子程序内接受aRFC的调用结果。可以使用以下接收参数：

- IMPORTING
- TABLES
- EXCEPTIONS

## **事务RFC：tRFC**

在使用事务RFC（ transactional RFC，tRFC）的时候，被调用的函数模块在被调用系统中正好运行一次（Exactly Once）。

远端系统不需要在RFC客户端程序运行tRFC的时候可用。tRFC组件将被调用的RFC函数和相关数据存储在SAP系统的数据库里，包含一个唯一的事务标识符（transaction identifier，TID）。

如果调用发送了，接收系统却是宕机状态，调用会保留在本地队列中一段时间。调用对话程序可以在不等待远程调用成功/失败的情况下继续运行。如果接收系统在一段时间后仍然不可用，调用将被计划为后台作业运行。



## SE16N 刷新缓存

事务码：/$SYNC

## 获取ALV内容并且不显示ALV

```ABAP
DATA: l_data  TYPE REF TO data.
FIELD-SYMBOLS: <fs_data> TYPE table.

"初始设置
 cl_salv_bs_runtime_info=>set( display  = abap_false
                               metadata = abap_false
                               data     = abap_true ).
" 调用目标程序                               
SUBMIT zppr048  WITH s_ebeln  IN s_ebeln
                  WITH s_lifnr  IN s_lifnr
                  WITH s_ekorg  IN s_ekorg
                  WITH s_ekgrp  IN s_ekgrp
                  WITH s_werks  IN s_werks
                  WITH s_bedat  IN s_bedat
                  WITH s_zhtbh  IN s_zhtbh
                  WITH s_matnr  IN s_matnr
                  WITH s_matkl  IN s_matkl
                  AND RETURN.
  TRY.
      cl_salv_bs_runtime_info=>get_data_ref( IMPORTING r_data = l_data ).
      ASSIGN l_data->* TO <fs_data>.
    CATCH cx_salv_bs_sc_runtime_info.
      MESSAGE `无法取得ALV数据` TYPE 'E'.
  ENDTRY.
 
 " 结束
 cl_salv_bs_runtime_info=>clear_all( ).
 
 "处理数据"
 DATA:lt_data TYPE TABLE OF zpps048.
 IF <fs_data> IS ASSIGNED .
    MOVE-CORRESPONDING <fs_data>  TO lt_data.
 ENDIF.
```

## 头订单查找

```abap
SELECT DISTINCT afko~lead_aufnr
       FROM afko
        INNER JOIN afpo ON afpo~aufnr = afko~aufnr
        INNER JOIN aufk ON aufk~aufnr = afko~aufnr
        INNER JOIN afru ON afru~aufnr = afko~aufnr
      WHERE  aufk~autyp = '10'
        AND aufk~werks IN @s_werks
        AND afko~aufnr IN @s_aufnr
        AND lead_aufnr IS NOT INITIAL
        AND afru~budat IN @s_budat
        AND NOT EXISTS (
                        SELECT
                          objnr
                          FROM jest
                          WHERE objnr = aufk~objnr
                            AND stat  = 'I0076'
                            AND inact = ''
                        )  INTO TABLE @et_lead_aufnr.
```

## 正则表达式

```abap
DATA: matcher  TYPE REF TO cl_abap_matcher.
"这里写正则表达式"
matcher = cl_abap_matcher=>create( pattern = '^(0|[1-9]\d*)?$' text = is_input-zgzsl ).
DATA(lv_match) = matcher->match( ).
IF lv_match = 'X'.
"为X表示匹配上了 否则就是没匹配上"
ENDIF.
```

```abap
"使用该方式匹配去校验的字符串 要使用String类型的 不能是char类型 否则匹配失败"
DATA:lv_match2 TYPE abap_bool.
DATA:lv_char TYPE string.
FIND REGEX '^[+-]?(\d+(\.\d+)?|\.\d+)([eE][+-]+\d+)+$' IN lv_char MATCH COUNT lv_match2.
IF lv_match2 > 0.
  " 匹配上了
ELSE.
  " 匹配失败
ENDIF.
```

## ALV红绿灯校验显示

```abap
"1、在定义alv展示结构时 添加字段
data:zlight(10) TYPE c .

"2、校验数据时 
通过 zlight = c_icon_green 
失败 zlight = c_icon_red
```

## APPEND INITIAL LINE 向内表中插入空行追加数据

```abap
DATA:lt_students TYPE STANDARD TABLE OF zmwb_students.
SELECT SINGLE * INTO @DATA(ls_data) FROM zmwb_students.

APPEND INITIAL LINE TO lt_students ASSIGNING FIELD-SYMBOL(<fs1>).
<fs1> = CORRESPONDING #( ls_data ).
```

## 离开屏幕/回到屏幕

```abap
LEAVE LIST-PROCESSING.
```

## UUID/GUID的生成方式 

###### 参考链接：https://mp.weixin.qq.com/s/pW-kD2m4E3BxxyL3lz_wIA

