# OS 



![OS](img\OS.jpg)

# User

Linux 是多用户系统，有两种类型的用户，普通用户和管理员。

- 普通用户：具有明确的权限
- 管理员：可以进行系统安装、配置、维护以及用户管理

每个用户都有自己的环境，在操作系统中以 UID 进行标识。

## 相关命令

### 1. passwd

修改用户密码。

# 命令帮助

- 在只记得部分命令关键字的场合，我们可通过`man -k`来搜索，比如`man -k pw`就可搜索出包”pw"这两个字符的命令；`apropos pw`也可以实现相同的效果；
- 需要知道某个命令的简要说明，可以使用`whatis`；而更详细的介绍，则可用`info`命令；
- 查看命令在哪个位置，我们需要使用`which`；
- 而对于命令的具体参数及使用方法，我们需要用到强大的`man`；

## `man`

`man`就是 manual 的缩写，用来在终端中显示一个命令的用法。
```bash
man passwd
```
这样就可以在终端中查看命令`passwd`的用法。在终端中一次只会显示一个 man 页面，有些命令可能有多个 man 页面，可以指定查看某个页面：
```bash
man 5 passwd
```
也可以查看所有的页面：
```bash
man -a passwd
```
当到达一个页面底部，按空格就会进入下一页面。

## `--help`

大多数命令都支持`--help`参数，用来显示这个命令的简易说明。

# File System

![图中的 root 只是指文件树的根，不是指超级用户](img\file system structure.jpg)

> On a Unix system, everything is a file; if something is not a file, it is a process.

文件系统中所包含的文件可以是以下对象：

- 常规文件
- 可执行文件（二进制文件或者 shell 脚本）
- 目录（在 Linux 中，文件和目录实际上是不进行区分的）
- 设备文件（键盘、显示器等等）
- socket
- 管道（pipe）
- 链接（一个让文件或目录在系统的文件树的多个部分都可见的系统）

所有文件和文件系统对象都被组成成一棵具有层次的树，并且具有一个根目录“/”。

文件名由字符序列组成，不能包含“/”，而且最好不要包含空格。Linux 的文件名大小写敏感（Windows 是大小写不敏感的）。

文件可以通过绝对路径和相对路径引用。

## 分区

分区的一个主要目的就是提高数据的安全性。通过分区，数据就被隔离开来，如果系统异常，那么就只有受到影响的分区的数据会被损坏，而其他分区的数据则大概率地不会受到影响。注意，分区只提供了在断电以及系统和存储设备突然中断的情况下的保护，并不能在文件系统发生逻辑错误等情况下依然保护数据。

### 分区的结构和类型



## 几个重要的目录

`/`是整个文件系统的根目录，`/etc`目录中存放的是配置文件。

## 文件名的搜索模式

![search pattern for files](img\search pattern for files.jpg)

## 相关命令

### 1. pwd

查看当前目录。

### 2. cd (directory)

进入指定目录，如果不指定参数，则进入主目录（在 Ubuntu 中，主目录为 /home/{username}，可简写为 $HOME 或~）。

`..`代表父目录，`.`代表当前目录。在主目录下执行`cd ../..`会进入最上面一级的目录，这个目录下放置有整个系统的配置文件等。

### 3. ls (-alR) (file/directory)

列出当前目录中的所有文件或者目录，文件名或者目录名可以是相对路径也可以是绝对路径。两个参数都是可选的。

-alR 参数的几个重要取值：

- `-a`：将隐藏的文件/目录一并列出（隐藏文件一般以`.`开头，这些文件通常都是配置文件，除非需要否则不要修改）
- `-l`：显示权限、用户和组、时间戳、大小等详细信息
- `-R`：递归地列出所有目录的子目录

### 4. mkdir directory & rmdir directory

第一个是创建一个空目录，第二个是删除一个==空==目录。

### 5. cp

复制文件或者目录，源文件或目录不会受到影响。

（1）`cp file1 file2`

将`file1`拷贝到`file2`中，这个操作会覆盖`file2`（如果`file2`原先就存在的话）。

（2）`cp file1 (file2 file3 ...) dir`

如果目录`dir`存在，那么列出的所有文件都会拷贝到该目录下。如果目录`dir`不存在，那么当`cp`的参数多于两个时，会报错；如果`cp`的参数刚好有两个，那么`cp file dir`就会被解释为将文件`file`拷贝到文件`dir`中。

（3）`cp -r dir1 dir2`

如果`dir2`已经存在，那么`dir1`就会递归地拷贝到`dir2`中；如果`dir2`不存在，那么就会创建`dir2`并递归地拷贝`dir1`。

（4）`cp -r dir1 [dir2 ...] dirn`

如果`dirn`存在，那么前面列出的所有文件/目录都会拷贝到`dirn`中；如果`dirn`不存在，就会报错。

### 6. mv

用于重命名或者移动文件。源文件将会被删除。

（1）`mv file1 file2`

将文件`file1`重命名为`file2`。

（2）`mv file1 [file2 ...] filen`

将列出的文件移动到`filen`中。

### 7. rm

`rm (-option) file`这个命令用来删除文件，被删除的文件将无法恢复。可以使用文件搜索模式。

有几个常用的可选参数：

- `-i`：在询问确认后删除
- `-r`：目录将会被递归地删除（删除所有子目录）
- `-f`：强制删除

`rm -r *`将会删除整棵文件数，除了那些隐藏的文件（以`.`开头的）

### 8. chmod

UNIX 文件系统中，文件通常有三种访问权限：

- r：只读
- w：创建、修改、删除，如果要创建或者删除文件，那么该文件的父文件也要具有同样的权限
- x：执行可执行文件或者 shell 脚本

文件访问权限的所有者通常被定义为：

- u：文件所有者
- g：组
- o：所有者和组以外的其他用户
- a：所有用户

要修改文件的访问权限，使用命令`chmod [who] [operator] [right] file`。只有文件所有者和超级用户（sudo）才能执行这条命令。

参数`who`的取值就是`u`、`g`、`a`、`o`，`right`的取值则是`r`、`w`、`x`。`operator`的取值有：

- `+`：为指定的用户类型增加权限
- `-`：去除指定用户类型的权限
- `=`：设置指定用户的权限，即将该类型的用户的所有权限重新设定

比如`chmod u+x file`是给文件所有者加上执行权限。

### 9. gedit

`gedit file`就是用 Ubuntu 自带的文本编辑器来编辑文件。当然也可以安装 Vim 然后用`vim file`来编辑文件。

### 10. cat

（1）`cat file`

将文件输出到标准输出流。

（2）`cat > file`

这个命令用来从标准输入流中读取字符，因此可以直接创建一个文本文件。输入完成之后回车换行，然后`ctrl D`退出输入。

```bash
maxwell@maxwell-VirtualBox:~$ cat > test
aaa
maxwell@maxwell-VirtualBox:~$ cat test
aaa
```

### 11. more

`more file`用来以分页的方式查看比较大的文件。

### 12. diff

`diff file1 file2`比较两个文件，这个命令的最大特点是会逐行比较两个文件然后告诉用户如果想让两个文件相同应该作何修改。

```
​```a.txt
Gujarat
Uttar Pradesh
Kolkata
Bihar
Jammu and Kashmir

​```b.txt
Tamil Nadu
Gujarat
Andhra Pradesh
Bihar
Uttar pradesh
```

```bash
$ diff a.txt b.txt
0a1
> Tamil Nadu
2,3c3
< Uttar Pradesh
< Kolkata
---
> Andhra Pradesh
5c5
< Jammu and Kashmir
---
> Uttar pradesh
```

要读懂输出，就要先弄懂输出中的数字、字母、符号是什么意思。

（1）数字

指代文件中的第几行。0则代表文件的开头，即第1行之前。

（2）字母

`a`代表 add，`c`代表 change，`d`代表 delete

（3）符号

- `<`：来自第一个文件的文本
- `>`：来自第二个文件
- `---`：将来自两个文件的把内容区分开，增加可读性

输出的第一行`0a1`则表示要在第1个文件的开头加上第2个文件的第1行内容，第二行则列出第2个文件的第1行内容。剩下的类似。

### 13. touch

`touch file`将文件的时间戳设置为当前时间。如果`file`不存在，那么就会创建它。

### 14. find

（1）`find /`

查看文件树种的所有文件。

（2）`find ~ -name '*jpg'`

查看主目录中的 jpg 文件，`-name`参数用来设置搜索模式。

`find ~ -iname '*jpg'`

`-i`的含义是 insensitive，即大小写敏感。

（3）`find ~ ( -iname '*jpeg' -o iname '*jpg')`

`-o`代表 or，可以将多个参数结合起来。

（4）`find ~ \( -iname '*jpeg' -o -iname '*jpg' \) -type f `

`-type`参数用来指定是要查看文件还是目录，如果是文件，那就是`f`，目录则是`d`

（5）`find ~ \( -iname '*jpeg' -o -iname '*jpg' \) -type f -mtime -7`

`-mtime`代表 modification time，`-7`代表 less than 7 days。类似的参数还有：

- `-ctime`：change time
- `-atime`: access time
- `+`： more than

change 和 modify 的区别：

modify 意味着对文件内容的修改，change 则是修改文件的元数据比如权限等。

（6）`find /var/log -size +1G`

在某个目录下查找大于 1G 的文件。`-size`参数用于指定大小。

（7）`find /data -owner bcotton`

`-owner`参数用于指定文件所有者。

（8）`find ~ -perm -o=r`

`-perm`用于指定权限。

### 15.grep

用正则表达式在一个文件中搜索文本。第一个参数是正则表达式，第二个参数是需要搜索的文件。

```bash
grep go hello.go
```

### 16. tar

这个命令用于压缩和解压文件。

（1）`tar -cvf file.tar dir`

上面的命令将目录/文件 dir 压缩为 file.tar。参数`-cvf`中每个字母的含义为：

- `c` – Creates a new .tar archive file.
- `v` – Verbosely show the .tar file progress.
- `f` – File name type of the archive file.

（2）`tar -cvzf file.tar.gz dir`

加上参数`z`则压缩为 gzip 文件。

`tar -cvfj file.tar.bz2 dir`，参数`j`表示压缩为 bz2 文件，这种文件比 gzip 小，但压缩和解压的耗更长。

（3）`tar -xfv file.tar`

在当前目录下解压文件。`x`代表解压 extract。

`tar -xvf file.tar -C dir`将文件夹解压到指定目录，`C`代表 specific directory。

` tar -xvf file.tar.gz`解压 gzip 文件。

（4）`tar -tvf file.tar`

不解压文件，指示列出压缩文件中的内容，`t`代表 content。

（5）`tar -xvf file1.tar file1`

从压缩文件中解压出 file1，而不是解压整个压缩文件。

`tar -xvf file1.tar "file1" "file2"`，解压多个文件。

`tar -xvf file1.tar --wildcards '*jpg'`使用通配符解压一组文件。

# UNIX Shell

shell 就是一个用户用来和操作系统交互的程序，它会解释用户输入的命令。

## shell 的种类

shell 并不是系统自带的，比较常用的 shell 有：

- Bourne shell（sh）：由 Steven Bourne 发明，是一个十分著名而且也被广泛使用的 shell。它的高级版本 bash 在 Linux 中十分流行。
- C-shell（csh）：伯克利发明的，使用类 C 语法。
- Bash shell（bash）：Bourne shell 的高级版，在许多系统上成为了标准的 shell。

## shell 脚本

shell 脚本是由 UNIX 命令和 shell 程序特有的指令组成的小程序，它和 UNIX 程序的区别在于 shell 脚本以文本形式而非二进制存储。shell 会解释 shell 脚本然后执行。不同种类的 shell 的 shell 脚本语法是不一样的。

一些 shell 脚本会在特定情况下自动执行：

- `.profile`和/或`.login`（如果存在）会在用户登录时执行且仅执行一次
- `.bashrc`和`.cshrc`/`.tcshrc`会在运行一个新的 shell 或 csh/tcsh 时分别执行。

在 Ubuntu 中，`~/.bashrc`常用来设置环境变量。（需要 sudo）

在本机上可以用`bash`命令执行`.sh`shell 脚本程序。

## 重定向输入/输出

一般用户是通过键盘输入 shell 命令，然后 shell 将结果输出到屏幕上。命令也可以从文件中读取，将结果输出到文件。

用`>`重定向输出，用`<`重定向输入。`>>`会将输出追加到已存在的文件，如果文件不存在，那么效果就和`>`一样。

```bash
$ cat a.txt >> b.txt
```

## 管道

一个命令的输出结果可以作为另一个命令的输入，这是通过管道实现的。

![管道](img\管道.jpg)

通过管道可以编写命令链，命令之间用`|`隔开，注意有些命令只能处在命令链的末尾，比如`>` 和`>>`。

### 应用示例：校验文件

从网上下载的文件有些会带有校验码，比如 SHA256 cheksum，要进行检验的话可以通过管道操作实现：

```bash
sha256sum file | grep checksum
```

首先用命令`sha256sum`计算 SHA256 哈希值，然后用命令`grep`将运算得到的结果和文件带有的 checksum 进行正则匹配。如果一致那么就会有返回结果，不一致则不会返回结果。

## Bash 快捷键

常用快捷键：

（1）`ctrl+A`：将光标移动到行首

（2）`ctrl+E`：将光标移动到行尾

（3）`ctrl+L`：相当于`clear`命令

（4）`ctrl+C`：终止进程

（5）`ctrl+Z`：暂停进程

（6）`↑ ↓`：查看最近命令

（7）`Shift+PgUp Shift+PgDn`：翻页

（8）`Tab`：自动补完命令或者文件名；如果有多种可能，系统会给出相关提示q

（9）`Tab Tab`:显示文件或命令补全的所有选项

# 进程管理

进程是一个正在运行的程序或者脚本，由程序/脚本本身和相应的环境组成。这里所说的环境由一些用来确保正确的程序流的额外信息组成。

一个进程具有以下特点：

- 具有一个唯一的进程 ID （PID）
- 带有父进程的 ID （PPID）
- 进程的优先级
- 用户和用户组

通常，一个进程运行从终端中运行起来之后这个终端就不能接收其他命令，除非进程结束（可以输入`ctrl C`即发送中断信号）。但是进程可以在后台运行，启动后台进程的方法是在命令末尾加上`&`： 

```bash
$ chrome &
```

## 相关命令

### 1. ps

`ps (prperty) (user)`用来显示正在运行的进程，如果不带任何参数，那么只列出当前用户在当前 shell 运行的进程。

property 的常用取值：

- `-a`：列出所有终端上运行的所有进程，包括后台进程和不属于当前用户的进程
- `-l`：列出所有信息

user 参数用来指定列出某个用户的进程，格式为`-u user`。

执行`ps`命令之后可以看到输出结果分为四列：

- PID：进程 id
- TTY：和这个进程相关的终端，即这个进程是从哪个终端启动的
- TIME：该进程总共使用的 CPU 时间
- CMD：进程的名称（如果有任何参数也会列出）

### 2. kill

`kill (-9) PID`用来终止 PID 指定的进程。这条命令只能被进程所有者和超级用户执行。可选参数`-9`用来终止那些“顽固”进程。  

### 3. 暂停进程

从终端运行进程之后输入`ctrl Z`会将进程暂停，此时进程处于僵尸状态，无法响应任何外部的命令，但是此时进程会把这些命令记住，一旦唤醒，进程就会进行响应。还可以通过`kill -STOP (PID)`命令暂停一个进程。

可以通过`fg`命令唤醒进程，也可以通过`kill -CONT (PID)`命令唤醒进程。

## job

在后台运行的进程称为 job，执行`jobs`命令就可以列出当前正在运行的后台进程，每一个后台进程都有一个 job ID。

后台进程是无法直接从终端进行管理的，除非将其移到前台。执行`fg %jobid`命令就可以把 jobid 指向的后台进程变为前台进程，









