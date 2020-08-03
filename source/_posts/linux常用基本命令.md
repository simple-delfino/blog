---
title: 'linux常用基本命令'
id: '20200803003'
tags:
  - linux
date: 2020-20-03 11:37:11
tag: linux
category: other
---

Linux中许多常用命令是必须掌握的，这里将我学linux入门时学的一些常用的基本命令分享给大家一下，希望可以帮助你们。

这个是我将鸟哥书上的进行了一下整理的，希望不要涉及到版权问题。

  

# 1、显示日期的指令： date

![](https://img-my.csdn.net/uploads/201303/22/1363935954_6311.png)

# 2、显示日历的指令：cal

![](https://img-my.csdn.net/uploads/201303/22/1363935980_1369.png)

![](https://img-my.csdn.net/uploads/201303/22/1363935990_8670.png)

![](https://img-my.csdn.net/uploads/201303/22/1363935996_9522.png)

# 3、简单好用的计算器：bc

![](https://img-my.csdn.net/uploads/201303/22/1363936042_6658.png)

![](https://img-my.csdn.net/uploads/201303/22/1363936054_8908.png)

  

怎么10/100会变成0呢？这是因为bc预设仅输出整数，如果要输出小数点下位数，那么就必须要执行 scale=number ，那个number就是小数点位数，例如：

![](https://img-my.csdn.net/uploads/201303/22/1363936069_2896.png)

# 4、重要的几个热键\[Tab\],\[ctrl\]-c, \[ctrl\]-d 

\[Tab\]按键---具有『命令补全』不『档案补齐』的功能

\[Ctrl\]-c按键---让当前的程序『停掉』

\[Ctrl\]-d按键---通常代表着：『键盘输入结束\(End Of File, EOF 戒 End OfInput\)』的意思；另外，他也可以用来取代exit

# 5、man

退出用q，

man \-f man

![](https://img-my.csdn.net/uploads/201303/22/1363936109_3235.png)

# 6、数据同步写入磁盘： sync

输入sync，那举在内存中尚未被更新的数据，就会被写入硬盘中；所以，这个挃令在系统关机戒重新启劢乀前， 径重要喔！最好多执行几次！

![](https://img-my.csdn.net/uploads/201303/22/1363936120_9820.png)

# 7、惯用的关机指令：shutdown

![](https://img-my.csdn.net/uploads/201303/22/1363936132_1780.png)

此外，需要注意的是，时间参数请务必加入指令中，否则shutdown会自动跳到 run-level 1 \(就是单人维护的登入情况\)，这样就伤脑筋了！底下提供几个时间参数的例子吧：

![](https://img-my.csdn.net/uploads/201303/22/1363936145_2758.png)

重启，关机： reboot, halt,poweroff

![](https://img-my.csdn.net/uploads/201303/22/1363936158_4694.png)

# 8、切换执行等级： init

Linux共有七种执行等级：

\--run level 0 :关机

\--run level 3 :纯文本模式

\--run level 5 :含有图形接口模式

\--run level 6 :重新启动

使用init这个指令来切换各模式：

如果你想要关机的话，除了上述的shutdown \-h now以及poweroff之外，你也可以使用如下的指令来关机：  

![](https://img-my.csdn.net/uploads/201303/22/1363936183_6281.png)

# 9、改变文件的所属群组：chgrp

![](https://img-my.csdn.net/uploads/201303/22/1363936198_8642.png)

# 10、改变文件拥有者：chown

他还可以顸便直接修改群组的名称

![](https://img-my.csdn.net/uploads/201303/22/1363936213_8090.png)

# 11、改变文件的权限：chmod

<table border="0" cellspacing="0" cellpadding="0" width="722"><tbody><tr><td valign="top"><p align="left"><span style="font-size:18px;">权限的设定方法有两种， 分别可以使用数字或者是符号来进行权限的变更。</span></p></td></tr></tbody></table>

  

\--数字类型改变档案权限：

![](https://img-my.csdn.net/uploads/201303/22/1363936248_2355.png)

\--符号类型改变档案权限：

![](https://img-my.csdn.net/uploads/201303/22/1363936259_4661.png)

# 12、查看版本信息等

![](https://img-my.csdn.net/uploads/201303/22/1363936285_4549.png)

# 13、变换目录：cd

![](https://img-my.csdn.net/uploads/201303/22/1363936297_1247.png)

# 14、显示当前所在目录：pwd

![](https://img-my.csdn.net/uploads/201303/22/1363936306_6088.png)

# 15、建立新目录：mkdir

![](https://img-my.csdn.net/uploads/201303/22/1363936321_4240.png)

![](https://img-my.csdn.net/uploads/201303/22/1363936335_4740.png)

不建议常用-p这个选项，因为担心如果你打错字，那么目录名称就回变得乱七八糟的

# 16、删除『空』的目录：rmdir

![](https://img-my.csdn.net/uploads/201303/22/1363936353_8065.png)

# 17、档案与目录的显示：ls

![](https://img-my.csdn.net/uploads/201303/22/1363936364_8540.png)

![](https://img-my.csdn.net/uploads/201303/22/1363936378_2040.png)

# 18、复制档案或目录：cp

![](https://img-my.csdn.net/uploads/201303/22/1363936391_3459.png)

![](https://img-my.csdn.net/uploads/201303/22/1363936405_1275.png)

![](https://img-my.csdn.net/uploads/201303/22/1363936421_3936.png)

![](https://img-my.csdn.net/uploads/201303/22/1363936434_7475.png)

![](https://img-my.csdn.net/uploads/201303/22/1363936447_1389.png)

![](https://img-my.csdn.net/uploads/201303/22/1363936473_8128.png)

![](https://img-my.csdn.net/uploads/201303/22/1363936488_3487.png)

# 19、移除档案或目录：rm

![](https://img-my.csdn.net/uploads/201303/22/1363936513_9642.png)

![](https://img-my.csdn.net/uploads/201303/22/1363936523_3283.png)

![](https://img-my.csdn.net/uploads/201303/22/1363936534_8970.png)

# 20、移动档案与目录，或更名：mv

![](https://img-my.csdn.net/uploads/201303/22/1363936546_1932.png)

![](https://img-my.csdn.net/uploads/201303/22/1363936558_1163.png)

# 21、取得路径的文件名与目录名：basename，dirname

![](https://img-my.csdn.net/uploads/201303/22/1363936571_2352.png)

![](https://img-my.csdn.net/uploads/201303/22/1363936582_4520.png)

# 22、由第一行开始显示档案内容：cat

![](https://img-my.csdn.net/uploads/201303/22/1363936609_6282.png)

![](https://img-my.csdn.net/uploads/201303/22/1363936621_3423.png)

# 23、从最后一行开始显示：tac（可以看出 tac 是 cat 的倒着写）

![](https://img-my.csdn.net/uploads/201303/22/1363936634_1541.png)

# 24、显示的时候，顺道输出行号：nl

![](https://img-my.csdn.net/uploads/201303/22/1363936651_6458.png)

![](https://img-my.csdn.net/uploads/201303/22/1363936671_4557.png)

![](https://img-my.csdn.net/uploads/201303/22/1363936684_7797.png)

# 25、一页一页的显示档案内容：more

![](https://img-my.csdn.net/uploads/201303/22/1363936710_9067.png)

# 26、与 more 类似，但是比 more 更好的是，他可以往前翻页：less

![](https://img-my.csdn.net/uploads/201303/22/1363936728_2729.png)

# 27、只看头几行：head

![](https://img-my.csdn.net/uploads/201303/22/1363936748_3301.png)

# 28、只看尾几行：tail

![](https://img-my.csdn.net/uploads/201303/22/1363936798_8160.png)

# 29、以二进制的放置读取档案内容：od

![](https://img-my.csdn.net/uploads/201303/22/1363936808_2619.png)

![](https://img-my.csdn.net/uploads/201303/22/1363936819_8586.png)

# 30、修改档案时间或新建档案：touch

![](https://img-my.csdn.net/uploads/201303/22/1363936832_1025.png)

![](https://img-my.csdn.net/uploads/201303/22/1363936847_2396.png)

![](https://img-my.csdn.net/uploads/201303/22/1363936857_7179.png)

# 31、档案预设权限：umask

![](https://img-my.csdn.net/uploads/201303/22/1363936868_1680.png)

# 32、配置文件档案隐藏属性：chattr

![](https://img-my.csdn.net/uploads/201303/22/1363936877_4715.png)

![](https://img-my.csdn.net/uploads/201303/22/1363936886_3351.png)

![](https://img-my.csdn.net/uploads/201303/22/1363936897_2939.png)

# 33、显示档案隐藏属性：lsattr

![](https://img-my.csdn.net/uploads/201303/22/1363936910_4210.png)

# 34、观察文件类型：file

![](https://img-my.csdn.net/uploads/201303/22/1363936923_3379.png)

# 35、寻找【执行挡】：which

![](https://img-my.csdn.net/uploads/201303/22/1363936932_2949.png)

![](https://img-my.csdn.net/uploads/201303/22/1363936946_1973.png)

# 36、寻找特定档案：whereis

![](https://img-my.csdn.net/uploads/201303/22/1363936956_5666.png)

# 37、寻找特定档案：locate

![](https://img-my.csdn.net/uploads/201303/22/1363937757_9288.png)

# 38、寻找特定档案：find

![](https://img-my.csdn.net/uploads/201303/22/1363937773_3082.png)

![](https://img-my.csdn.net/uploads/201303/22/1363937784_2757.png)

# 39、压缩文件和读取压缩文件：gzip，zcat

![](https://img-my.csdn.net/uploads/201303/22/1363937793_4694.png)

![](https://img-my.csdn.net/uploads/201303/22/1363937802_1953.png)

# 40、压缩文件和读取压缩文件：bzip2，bzcat

![](https://img-my.csdn.net/uploads/201303/22/1363937816_9269.png)

![](https://img-my.csdn.net/uploads/201303/22/1363937900_4463.png)

# 41、压缩文件和读取压缩文件：tar

![](https://img-my.csdn.net/uploads/201303/22/1363937990_3354.png)

![](https://img-my.csdn.net/uploads/201303/22/1363938001_5653.png)

![](https://img-my.csdn.net/uploads/201303/22/1363938031_1319.png)  


![](https://img-my.csdn.net/uploads/201303/22/1363938044_9852.png)

![](https://img-my.csdn.net/uploads/201303/22/1363938052_3223.png)

![](https://img-my.csdn.net/uploads/201303/22/1363938061_6833.png)

好了，累死了，终于搞完了，希望能对的大家有所帮助。

[转自 【csdn】 xiaoguaihai](https://blog.csdn.net/xiaoguaihai/article/details/8705992)