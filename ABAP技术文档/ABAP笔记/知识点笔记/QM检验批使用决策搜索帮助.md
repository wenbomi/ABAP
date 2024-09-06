<img src="../../../../Library/Application%20Support/typora-user-images/image-20240619085429099.png" alt="image-20240619085429099" style="zoom:50%;" />

<img src="../../../../Library/Application%20Support/typora-user-images/image-20240619085514649.png" alt="image-20240619085514649" style="zoom:50%;" />

写在搜索帮助事件里

```abap
AT SELECTION-SCREEN ON VALUE-REQUEST FOR s_vcode-low.
```

```ABAP
DATA: l_qpk1ac LIKE qpk1ac.
CALL FUNCTION 'QPK1_UD_CODE_PICKUP_LEAN'
    EXPORTING
      i_werks              = '*' "s_vwerks-low
      i_auswahlmge         = '*' "s_vauswa-low
      i_codegruppe         = '*' "s_vcodeg-low
      i_code               = '*' "s_vcode-low
      i_no_usageindication = 'X'
    IMPORTING
      e_qpk1ac             = l_qpk1ac
    EXCEPTIONS
      OTHERS               = 1.
```

