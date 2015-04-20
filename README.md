#UNIX高级编程
#####菜鸟读书随记
######第1章 UNIX基础知识
* 1.2 __unix 体系结构__
unix操作系统体系结构：内核-》系统调用-》SHELL & 公共函数 -》应用程序
* 1.3 __登录__
登录用户帐号信息存放在 /etc/passwd 里，口令文件中的登录项由7个冒号分隔。
依次是：登录名、加密口令、数字用户ID、数字组ID、注释字段、起始目录、登录后使用的shell程序
* 1.4 __文件和目录__
1. 目录是一个包含目录项的文件。在逻辑上，可以认为每个目录项都包含一个文件名，同时还包含说明文件属性的信息。
**stat 和 fstat 函数返回包含所有文件属性的一个信息结构。**
2. 文件名  创建新目录时会自动创建两个文件名： .(称为点) 和 ..(称为点点)。点指向当前目录，点点指向父目录。
在最高层次的根目录中，点点与点相同。
3. 路径名  文件系统根的名字（/）是一个特殊的绝对路径名，它不包含文件名。

* 代码实例 《列出一个目录中的所有文件》
```c
#include	<sys/types.h>
#include	<dirent.h>
#include	"ourhdr.h"

int
main(int argc, char *argv[])
{
	DIR				*dp;
	struct dirent	*dirp;

	if (argc != 2)
		err_quit("a single argument (the directory name) is required");

	if ( (dp = opendir(argv[1])) == NULL)
		err_sys("can't open %s", argv[1]);

	while ( (dirp = readdir(dp)) != NULL)
		printf("%s\n", dirp->d_name);

	closedir(dp);
	exit(0);
}
/*
cc ls1.c 
编译
历史上，cc是C编译器。在配置了GNU C 编译系统的系统中，C编译器是gcc， cc 通常链接至 gcc。

*/
```

```shell
http://www.apuebook.com/code3e.html
http://www.yendor.com/programming/unix/apue/apue.html
本书所有源码下载地址：http://www.apuebook.com/src.3e.tar.gz
tar zxvf src.3e.tar.gz
make
可能会出错这时按下面的操作：
1）ubuntu
sudo apt-get install libbsd-dev
2) fedora, redhat, centos
2.1) 下载两个包
2.1a) i386
ftp://ftp.univie.ac.at/systems/linux/fedora/epel/6/i386/libbsd-0.6.0-1.el6.i686.rpm
ftp://ftp.univie.ac.at/systems/linux/fedora/epel/6/i386/libbsd-devel-0.6.0-1.el6.i686.rpm
2.1b) x86_64
ftp://ftp.univie.ac.at/systems/linux/fedora/epel/6/x86_64/libbsd-0.6.0-1.el6.x86_64.rpm
ftp://ftp.univie.ac.at/systems/linux/fedora/epel/6/x86_64/libbsd-devel-0.6.0-1.el6.x86_64.rpm

```
```c
/*
DIR这一结构体，以下为DIR结构体的定义：
*/

struct __dirstream
   {
    void *__fd;
    char *__data;
    int __entry_data;
    char *__ptr;
    int __entry_ptr;
    size_t __allocation;
    size_t __size;
    __libc_lock_define (, __lock)
   };
typedef struct __dirstream DIR;

/*
DIR结构体类似于FILE，是一个内部结构，以下几个函数用这个内部结构保存当前正在被读取的目录的有关信息（摘自《UNIX环境高级编程（第二版）》）。函数 DIR *opendir(const char *pathname)，即打开文件目录，返回的就是指向DIR结构体的指针，而该指针由以下几个函数使用:
*/

struct dirent *readdir(DIR *dp);
void rewinddir(DIR *dp);
int closedir(DIR *dp);
long telldir(DIR *dp);
void seekdir(DIR *dp,long loc);
/*
接着是dirent结构体，首先我们要弄清楚目录文件（directory file）的概念：这种文件包含了其他文件的名字以及指向与这些文件有关的信息的指针（摘自《UNIX环境高级编程（第二版）》）。从定义能够看出，dirent不仅仅指向目录，还指向目录中的具体文件，readdir函数同样也读取目录下的文件，这就是证据。以下为dirent结构体的定义：
*/
struct dirent
{
　　long d_ino; /* inode number 索引节点号 */
    off_t d_off; /* offset to this dirent 在目录文件中的偏移 */
    unsigned short d_reclen; /* length of this d_name 文件名长 */
    unsigned char d_type; /* the type of d_name 文件类型 */
    char d_name [NAME_MAX+1]; /* file name (null-terminated) 文件名，最长255字符 */
}
/*
从上述定义也能够看出来，dirent结构体存储的关于文件的信息很少，所以dirent同样也是起着一个索引的作用，如果想获得类似ls -l那种效果的文件信息，必须要靠stat函数了。
通过readdir函数读取到的文件名存储在结构体dirent的d_name成员中，而函数
int stat(const char *file_name, struct stat *buf);
的作用就是获取文件名为d_name的文件的详细信息，存储在stat结构体中。以下为stat结构体的定义：
*/
struct stat {
        mode_t     st_mode;       //文件访问权限
        ino_t      st_ino;       //索引节点号
        dev_t      st_dev;        //文件使用的设备号
        dev_t      st_rdev;       //设备文件的设备号
        nlink_t    st_nlink;      //文件的硬连接数
        uid_t      st_uid;        //所有者用户识别号
        gid_t      st_gid;        //组识别号
        off_t      st_size;       //以字节为单位的文件容量
        time_t     st_atime;      //最后一次访问该文件的时间
        time_t     st_mtime;      //最后一次修改该文件的时间
        time_t     st_ctime;      //最后一次改变该文件状态的时间
        blksize_t st_blksize;    //包含该文件的磁盘块的大小
        blkcnt_t   st_blocks;     //该文件所占的磁盘块
      };
/*
这个记录的信息就很详细了吧，呵呵。
最后，总结一下，想要获取某目录下（比如a目下）b文件的详细信息，我们应该怎样做？
首先，我们使用opendir函数打开目录a，返回指向目录a的DIR结构体c。
接着，我们调用readdir( c)函数读取目录a下所有文件（包括目录），返回指向目录a下所有文件的dirent结构体d。
然后，我们遍历d，调用stat（d->name,stat *e）来获取每个文件的详细信息，存储在stat结构体e中。
总体就是这样一种逐步细化的过程，在这一过程中，三种结构体扮演着不同的角色。
*/
```
注部分内容源地址：http://www.liweifan.com/2012/05/13/linux-system-function-files-operation/

* 1.5 __输入和输出__
1. 文件描述符
__文件描述符__ 通常是一个小的非负整数，内核用以标识一个特定的进程正在访问的文件，当内核打开一个现有文件或创建一个新文件时，它都返回一个文件描述符。在读、写文件时，可以使用这个文件描述符。
2. 标准输入、标准输出和标准错误
按惯例，每当运行一个新程序时，所有的shell都为其打开3个文件描述符，即标准输入、标准输出、标准错误。
如果不做特殊处理，如ls命令，则这3个描述符都链接向终端（显示器）。大多数shell都提供一种方法，使其中任何一个或所有3个描述符都能重新定向到某个文件。例如 ls > file.list
3. 不带缓冲的i/o
函数 open 、 read 、 write 、 lseek 以及 close 提供了不带缓冲的I/O 。 这些函数都使用文件描述符。

*实例 《将标准输入复制到标准输出》
```c
// mycat.c

#include	"ourhdr.h"

#define	BUFFSIZE	8192

int
main(void)
{
	int		n;
	char	buf[BUFFSIZE];

	while ( (n = read(STDIN_FILENO, buf, BUFFSIZE)) > 0)
		if (write(STDOUT_FILENO, buf, n) != n)
			err_sys("write error");

	if (n < 0)
		err_sys("read error");

	exit(0);
}
```
相关说明：
STDIN_FILENO 和 STDOUT_FILENO 是 POSIX 标准的一部分，定义在unistd.h 头文件中，他们指定了标准输入和标准输出的文件描述符。在POSIX标准中，他们的值分别是0和1，考虑到可读性，我们将使用这些名字来表示这些常量。

read 函数返回0，程序停止执行。如果发生错误则返回-1.

* 标准I/O 
使用标准I/O函数无需担心如何选择最佳的缓冲区大小。
如 fgets 函数读取一行完整的行，而read函数读取指定字节数。

这段代码跟上面的功能相似，但使用的是标准I/O函数

标准I/O常量 stdin 和 stdout 定义在头文件 stdio.h 中。

```c
#include	"ourhdr.h"

int
main(void)
{
	int		c;

	while ( (c = getc(stdin)) != EOF)
		if (putc(c, stdout) == EOF)
			err_sys("output error");

	if (ferror(stdin))
		err_sys("input error");

	exit(0);
}
```
* 1.6 __程序和进程__
1. 程序 是一个存储在磁盘上某个目录中的可执行文件。内核使用 exec 函数（7个exec函数之一），将程序读入内存，并执行程序。
2. 进程和进程ID 程序的执行实例称为程序。
某些操作系统用任务表示正在执行的程序。
UNIX系统确保每个进程都有一个唯一的数字标识符，称为进程ID，进程ID总是一个非负整数。
可以使用getpid()函数获取当前进程ID，getpid 返回一个pid_t数据类型，不确定具体大小，但跟据相关标准可以确定其返回值可以保存在一个long长整型中。
3. 进程的控制 有3个用于进程控制的主要函数： fork 、 exec 和 waitpid。（exec函数有7种变体，但经常把它们统称为exec函数）
实例： 《从标准输入读入命令并执行》
```c
#include	<sys/types.h>
#include	<sys/wait.h>
#include	"ourhdr.h"

int
main(void)
{
	char	buf[MAXLINE];
	pid_t	pid;
	int		status;

	printf("%% ");	/* print prompt (printf requires %% to print %) */
	while (fgets(buf, MAXLINE, stdin) != NULL) {
		buf[strlen(buf) - 1] = 0;	/* replace newline with null */

		if ( (pid = fork()) < 0)
			err_sys("fork error");

		else if (pid == 0) {		/* child */
			execlp(buf, buf, (char *) 0);
			err_ret("couldn't execute: %s", buf);
			exit(127);
		}

		/* parent */
		if ( (pid = waitpid(pid, &status, 0)) < 0)
			err_sys("waitpid error");
		printf("%% ");
	}
	exit(0);
}
/*
注
execlp 函数要求的参数是以null结束的
*/


```

4 线程和线程ID 一个进程内的所有线程共享迥一地址空间、文件描述符、栈以及与进程相关的属性，它们能够访问同一存储区，各线程在访问共享数据时需要采取同步措施避免不一致性

* 1.7 __出错处理__
* 1.8 __用户标识__
POSIX要求系统里的用户至少应支持8个附属组，一般可以属于多至16个其它用户组。

* 1.9 __信号__
信号 用于通知进程发生了某种情况。
进程有以下3种处理信号的方式：
1.忽略信号
2.按系统默认方式处理
3.提供一个函数，信号发生时调用该函数，称为捕捉该信号。通过提供自编的函数，我们就能知道什么时候产生了信号，并按期望的方式处理它。
向一个进程发送信号时，我们必须是那个进程的所有者或者是超级用户。

