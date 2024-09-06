终端Terminal

什么是终端？

首先了解操作系统的组成：

**操作系统分为两个部分，一部分称作内核，另一部分成为用户交互界面**。

内核部分负责系统的全部逻辑操作，由海量命令组成，这一部分是系统运行的命脉，不与用户接触；交互界面则是开机之后所有我们所看到的东西，比如窗口，软件，应用程序等等。



**终端就是连接内核与交互界面的这座桥**，它允许用户在交互界面上打开一个叫做「Terminal 终端」的应用程序，在其中输入命令，系统会直接给出反馈。

```
查看根路径,当前路径
pwd
```

相对路径：当前路径指的是现在终端所处的位置，若你想**改变当前路径**，则可以输入 `cd /其他文件夹`。

绝对路径：写法 「/文件夹名/文件夹名」，它指的其实就是**绝对路径，你必须指定它从根目录一直到达具体的文件夹。**

**相对路径允许你告诉终端从现在开始，接下来应该怎么走。**相对路径的书写方法实在绝对路径前加一个 `.`。

<img src="/Users/miwenbo/Library/Application%20Support/typora-user-images/image-20240721204214033.png" alt="image-20240721204214033" style="zoom:50%;" />

在当前路径的基础上，降文件夹位置切换到Utilities

<img src="/Users/miwenbo/Library/Application%20Support/typora-user-images/image-20240721204522119.png" alt="image-20240721204522119" style="zoom:50%;" />



最简单的方式就是将你想要获取的文件直接拖拽到终端 即可得到其绝对路径

```
查看当前路径下有哪些文件
ls
```

<img src="/Users/miwenbo/Library/Application%20Support/typora-user-images/image-20240721204743969.png" alt="image-20240721204743969" style="zoom:50%;" />

```
清屏
clear
```

```
当遇到不熟悉的指令 可以通过man指令获取使用指导
man XXX
退出说明文档 按下键盘Q
```

<img src="/Users/miwenbo/Library/Application%20Support/typora-user-images/image-20240721205322481.png" alt="image-20240721205322481" style="zoom:50%;" />

<img src="/Users/miwenbo/Library/Application%20Support/typora-user-images/image-20240721205226188.png" alt="image-20240721205226188" style="zoom:33%;" />

- **不要进入休眠状态：**当你临时不希望电脑进入休眠状态时，可以使用 `caffeinate`命令让电脑时刻清醒。当你需要其恢复正常时，按下 `⌃Control - C` 即可停止该命令。

  

  

  **程序假死需要强退：**有时候程序假死了，强行退出也没用，这时可以使用 `killall`命令。以微信为例，若想强退它，只需输入 `killall WeChat` 即可