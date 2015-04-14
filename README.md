#UNIX高级编程
#####菜鸟读书随记
######第1章 UNIX基础知识
* 1.2 unix 体系结构
unix操作系统体系结构：内核-》系统调用-》SHELL & 公共函数 -》应用程序
* 1.3 登录
登录用户帐号信息存放在 /etc/passwd 里，口令文件中的登录项由7个冒号分隔。
依次是：登录名、加密口令、数字用户ID、数字组ID、注释字段、起始目录、登录后使用的shell程序
* 1.4 文件和目录
+目录是一个包含目录项的文件。在逻辑上，可以认为每个目录项都包含一个文件名，同时还包含说明文件属性的信息。
**stat 和 fstat 函数返回包含所有文件属性的一个信息结构。**
+文件名  创建新目录时会自动创建两个文件名： .(称为点) 和 ..(称为点点)。点指向当前目录，点点指向父目录。
在最高层次的根目录中，点点与点相同。
+路径名  文件系统根的名字（/）是一个特殊的绝对路径名，它不包含文件名。

