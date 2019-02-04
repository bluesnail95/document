因为自己平时使用的比较多的是centos7.2，所以以下记录的命令均是实践在centos7.2系统上。

> 遇到问题如何解决
-  linux数据文件：/usr/share/doc
- The Linux Documentation Project：http://www.tldp.org
- 鸟哥的网站：http://linux.vbird.org/
- 酷学园讨论区：http://phorum.study-area.org
- 鸟哥的私房菜讨论区：http://phorum.vbird.org

> 如何远程登录
- XShell
- WinScp
- MobaXterm

> 使用linux需要注意的问题
- 命令英文大小写敏感。
- 无论几个空格，shell都会当做一个空格，空格是一个特殊字符。
- 平时使用一般账号进行操作，root账号拥有全部权限，容易误操作。
- root账号的提示符是#，一般账号的提示符是$。
- 在关闭linux之前需要查看当前是否有其他人在线或是有其他正在执行的进程，使用shutdown或是reboot命令。

> linux的权限
- 文件的权限分为User(使用者)、Group(群组)和Others(其他)。
- root相关信息存放在/etc/passwd文件中，个人密码存放在/etc/shadow文件中，linux的所有组名存放在/etc/group文件中。
- chgrp改变文件的所属群组；chown改变文件的所有者；chmod改变文件的权限。
- 文件权限：r(read)表示读取文件的实际内容；w(write)表示增改文件内容;x(execute)表示具有被系统执行的权限。
- 目录权限：r(read contents in directory)表示具有读取目录结构列表的权限；w(modify contents of directory)表示具有增删更名搬移文件和目录的权限；x(access directory)表示用户能否进入该目录成为当前工作目录。



> 常用的快捷键

|快捷键|描述|
|:------|:----|
|Tab|两个Tab跟在一串命令的第一个命令后面表示命令补齐，跟在第二个命令后面表示文件/参数/选项补齐,如果要进行参数/选项补齐，需要安装bash-completion软件。|
|Ctrl+C|中断正在运行的命令|
|Ctrl+D|键盘输入结束或是代表exit|
|Shift+PageUp|向上翻页|
|Shift+PageDown|向下翻页|

>  如何查找Linux命令的信息
- 指令添加--help可以查询该指令的相关功能，如cal --help。
- 使用man或info查询指令的信息，如man cal。
- /usr/share/doc文件夹下会有一些服务的文件说明。

> 常用的命令

|命令|描述|
|:------|:----|
|su 用户名|切换用户，如su root，表示切换至root用户，输入exit切换回原来的用户。|

> 参考资料
- 《鸟哥的Linux私房菜》基础篇第四版



