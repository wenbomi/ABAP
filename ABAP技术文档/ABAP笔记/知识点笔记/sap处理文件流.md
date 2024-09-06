业务场景：外围系统通过PO给SAP发送PDF文件流base64格式，SAP接收文件流，转成PDF且在GUI内打开PDF展示

展示效果：

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240829095121111.png" alt="image-20240829095121111" style="zoom:30%;" /<img src="../../../../Library/Application%20Support/typora-user-images/image-20240829095149741.png" alt="image-20240829095149741" style="zoom:33%;" />



PO部分设计略过，这是展示部分文件流数据格式,`雀氏看不懂`

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240829094927631.png" alt="image-20240829094927631" style="zoom:50%;" />

将BASE64数据转为二进制数据，按照需求还可以存表

```abap
DATA:indata TYPE string,
		iv_buffer TYPE xstring. "文本格式"
indata = <fs>-zpdfurl. "BASE64数据"
CALL FUNCTION 'SSFC_BASE64_DECODE'
  EXPORTING
    b64data                  = indata
*   B64LENG                  =
*   B_CHECK                  =
  IMPORTING
    bindata                  = iv_buffer
  EXCEPTIONS
    ssf_krn_error            = 1
    ssf_krn_noop             = 2
    ssf_krn_nomemory         = 3
    ssf_krn_opinv            = 4
    ssf_krn_input_data_error = 5
    ssf_krn_invalid_par      = 6
    ssf_krn_invalid_parlen   = 7
    OTHERS                   = 8.
```

文本格式转为二进制数据

```abap
DATA:   lt_datatab        TYPE TABLE OF ssfdata,
        iv_buffer         TYPE xstring,
        lv_full_name      TYPE string,
        lv_file_length    TYPE i,
        lv_temp_dir       TYPE string,
        lv_file_separator TYPE c LENGTH 1.

CALL FUNCTION 'SCMS_XSTRING_TO_BINARY'
    EXPORTING
      buffer        = iv_buffer
    IMPORTING
      output_length = lv_file_length
    TABLES
      binary_tab    = lt_datatab.
```

创建展示数据的对象

```abap
DATA:
  go_html_control TYPE REF TO cl_gui_html_viewer,
  go_container    TYPE REF TO cl_gui_docking_container.
  
FORM initial_0100 .
  IF go_html_control IS NOT BOUND.
    "创建容器与组件对象
    PERFORM create_container_assembly.
    "设置组件展示用的内容
    PERFORM set_assembly_data.
  ENDIF.
ENDFORM.

FORM create_container_assembly .
  "实例化容器对象
  IF go_container IS INITIAL.
    CREATE OBJECT go_container
      EXPORTING
        repid     = sy-repid
        dynnr     = sy-dyngr
        side      = cl_gui_docking_container=>dock_at_left
        extension = 2000.
  ENDIF.

  "实例化组件对象
  CREATE OBJECT go_html_control
    EXPORTING
      parent = go_container.
ENDFORM.
FORM set_assembly_data .
*--------------------------Variables-----------------------------------*
  DATA:
    lv_url          TYPE c LENGTH 200.
*----------------------------Logic-------------------------------------*
  "展示PDF文件
    CALL METHOD go_html_control->load_data(
      EXPORTING
        type                 = 'APPLICATION'
        subtype              = 'PDF'
      IMPORTING
        assigned_url         = lv_url
      CHANGING
        data_table           = lt_datatab
      EXCEPTIONS
        dp_invalid_parameter = 1
        dp_error_general     = 2
        cntl_error           = 3
        OTHERS               = 4 ).

*  "展示组件内容
  CALL METHOD go_html_control->show_url(
      url      = lv_url
      in_place = 'X' ).
ENDFORM.
```

