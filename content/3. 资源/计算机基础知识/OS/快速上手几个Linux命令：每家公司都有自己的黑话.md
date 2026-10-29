---
id: 4
title: 快速上手几个Linux命令：每家公司都有自己的黑话
sidebar_label: 快速上手几个Linux命令：每家公司都有自己的黑话
description: 来自极客时间
type: knowledge
status: active
moc:
  - "[[计算机基础 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---

如果你还没有上手用过 Linux，那么接下来的课程，你可能会感受到困惑。因为没有一手的体验，你可能很难将Linux的机制和你的使用行为关联起来。所以这一节，咱们先介绍几个上手 Linux 的命令，通过这些命令，我们试试先把 Linux 用起来。

为什么我把 Linux 命令称为“黑话”呢？就像上一节我们介绍的，Linux 操作系统有很多功能，我们有很多种方式可以使用这些功能，其中最简单和直接的方式就是命令行（Command Line）。命令行就相当于你请求服务使用的专业术语。干任何事情，第一步就是学会使用正确的术语。这样，Linux 作为服务方，才能听懂。这些术语可不就是“黑话”吗？

Windows 系统你肯定很熟悉吧？现在，我就沿着你使用Windows的习惯，来给你介绍相应的Linux命令。

## 用户与密码

当我们打开一个新系统的时候，第一件要做的事就是登录。系统默认有一个Administrator用户，也就是系统管理员，它的权限很大，可以在这个系统上干任何事。Linux上面也有一个类似的用户，我们叫Root。同样，它也具有最高的操作权限。

接下来，你需要输入密码了。密码从哪里来呢？对于 Windows 来讲，在你安装操作系统的过程中，会让你设置一下Administrator的密码；对于 Linux，Root 的密码同样也是在安装过程中设置的。

![img](./assets/ee95d03b1390ae08ca9c752621b03476.png)

对于 Windows，你设好之后，可以多次修改这个密码。比如说，我们在控制面板的账户管理里面就可以完成这个操作。但是对于 Linux 呢？不好意思，没有这么一个统一的配置中心了。你需要使用命令来完成这件事情。这个命令很好记，**<font color="red">passwd</font>**，其实就是 password的简称。

```shell
# passwd
Changing password for user root.
New password:
```

按照这个命令，我们就可以输入新密码啦。

在 Windows 里，除了 Administrator 之外，我们还可以创建一个以自己名字命名的用户。那在 Linux 里可不可以创建其他用户呢？当然可以了，我们同样需要一个命令useradd。

```shell
 useradd cliu8
```

执行这个命令，一个用户就被创建了。它不会弹出什么让你输入密码之类的页面，就会直接返回了。因为接下来你需要自己调用 passwd cliu8 来设置密码，再进行登录。

在 Windows 里设置用户的时候，用户有一个“组”的概念。你可能没注意过，不过我一说名字你估计就能想起来了，比如“Administrator 组”“Guests 组”“Power User 组”等等。同样，Linux 里也是分组的。前面我们创建用户的时候，没有说加入哪个组，于是默认就会创建一个同名的组。

能不能在创建用户的时候就指定属于哪个组呢？我们来试试。我们可以使用-h参数看一下，使用 useradd 这个命令，有没有相应的选项。

```shell
[root@deployer ~]# useradd -h
Usage: useradd [options] LOGIN
       useradd -D
       useradd -D [options]


Options:
  -g, --gid GROUP               name or ID of the primary group of the new account
```

一看还真有这个选项。以后命令不会用的时候，就可以通过 -h 参数看一下，它的意思是help。

如果想看更加详细的文档，你可以通过man useradd获得，细细阅读。

![img](./assets/179b8fdca3d8d57e8f1d32f3aab60a2d.png)

上一节我们说过，Linux 里是“命令行 + 文件”模式。对于用户管理来说，也是一样的。咱们通过命令创建的用户，其实是放在/etc/passwd 文件里的。这是一个文本文件。我们可以通过cat命令，将里面的内容输出在命令行上。组的信息我们放在/etc/group文件中。

```shell
# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
......
cliu8:x:1000:1000::/home/cliu8:/bin/bash


# cat /etc/group
root:x:0:
......
cliu8:x:1000:
```

在/etc/passwd文件里，我们可以看到root用户和咱们刚创建的cliu8用户。x的地方应该是密码，密码当然不能放在这里，不然谁都知道了。接下来是用户ID和组ID，这和/etc/group里面就对应上了。

/root和/home/cliu8是什么呢？它们分别是root用户和cliu8用户的主目录。主目录是用户登录进去后默认的路径。其实Windows里面也是这样的。当我们打开文件夹浏览器的时候，左面会有“文档”“图片”“下载”等文件夹，路径在 C:\Users\cliu8下面。要注意，同一台电脑，不同的用户情况会不一样。

![img](./assets/d21ce3cd2ade7b71300df6a805b45aa7.png)

/bin/bash的位置是用于配置登录后的默认交互命令行的，不像Windows，登录进去是界面，其实就是 explorer.exe。而Linux登录后的交互命令行是一个解析脚本的程序，这里配置的是/bin/bash。

## 浏览文件

终于登录进来啦，接下来你可以在文件系统里面随便逛一逛、看一看了。

可以看到，Linux的文件系统和Windows是一样的，都是用文件夹把文件组织起来，形成一个树形的结构。这一点没有什么差别。只不过在Linux下面，大多数情况，我们需要通过命令行来查看Linux的文件。

其实在 Windows 下也有命令行，例如**<font color="red">cd</font> **就是change directory，就是切换目录；cd .表示切换到当前目录；cd ..表示切换到上一级目录；使用dir，可以列出当前目录下的文件。Linux基本也是这样，只不过列出当前目录下的文件我们用的是**<font color="red">ls</font>**，意思是list。

![img](./assets/27cc0efe8d33b730eba8aee7d51cda2e.png)

我们常用的是 ls -l，也就是用列表的方式列出文件。

> “ls -l”,可以直接用 “ll” 来列出文件

```shell
# ls -l
drwxr-xr-x 6 root root    4096 Oct 20  2017 apt
-rw-r--r-- 1 root root     211 Oct 20  2017 host
```

其中第一个字段的第一个字符是**文件类型**。如果是“-”，表示普通文件；如果是 d，就表示目录。当然还有很多种文件类型，咱们后面遇到的时候再说，你现在先记住我说的这两个就行了。

第一个字段剩下的9个字符是**模式**，其实就是**权限位**（access permission bits）。3个一组，每一组rwx表示“读（read）”“写（write）”“执行（execute）”。如果是字母，就说明有这个权限；如果是横线，就是没有这个权限。

这三组分别表示文件所属的用户权限、文件所属的组权限以及其他用户的权限。例如，上面的例子中，-rw-r--r-- 就可以翻译为，这是一个普通文件，对于所属用户，可读可写不能执行；对于所属的组，仅仅可读；对于其他用户，也是仅仅可读。如果想改变权限，可以使用命令 chmod 711 hosts。

第二个字段是**硬链接**（hard link）**数目**，这个比较复杂，讲文件的时候我会详细说。

第三个字段是**所属用户**，第四个字段是**所属组**。第五个字段是文件的大小，第六个字段是文件被修改的日期，最后是文件名。你可以通过命令**<font color="red">chown</font>**改变所属用户，**<font color="red">chgrp</font>**改变所属组。

> chmod命令是改变该文件的权限位。 
>
> chmod XXX 文件名。 
>
> X是二进制的十进制表示法，-为0，则代表该位无权限，-为1，则代表该位有权限。最后转为十进制数字即可。
>
> chown命令是改变该文件所属用户。
>
> chgrp命令是改变该文件所属组。也就是chmod、chown、chgrp命令都是针对文件的操作，赋予该文件具有什么权限，该文件属于哪个用户，该文件属于哪个组，之后，用户在登录的时候，根据与此文件的关系，就知道有没有权限操作此文件。 该文件所属用户可以使用chmod命令改变该文件的权限位。
>
> 但是该文件所属用户没有办法使用chown、chgrp命令改变文件所属用户，以及文件所属组。 改变文件所属用户，以及改变文件所属组这个操作只能root用户来做。

## 安装软件

好了，你现在应该会浏览文件夹了，接下来应该做什么呢？当然是开始安装那些“装机必备”的软件啦！

在 Windows 下面，在没有类似软件管家的软件之前，我们其实都是在网上下载 installer，然后再进行安装的。

就以我们经常要安装的 JDK 为例子。应该去哪里下载呢？为了安全起见，一般去官网比较好。如果你去 JDK 的官网，它会给你一个这样的列表。

![img](./assets/5e54fe2dba0e86e14a7a92d9ea46c202.jpg)

对于 Windows 系统，最方便的方式就是下载 exe，也就是安装文件。下载后我们直接双击安装即可。

对于 Linux 来讲，也是类似的方法，你可以下载rpm或者deb。这个就是Linux下面的安装包。为什么有两种呢？因为Linux现在常用的有两大体系，一个是CentOS体系，一个是Ubuntu体系，前者使用rpm，后者使用deb。

在Linux上面，没有双击安装这一说，因此想要安装，我们还得需要命令。CentOS下面使用**<font color="red">rpm -i jdk-XXX_linux-x64_bin.rpm</font>**进行安装，Ubuntu下面使用**<font color="red">dpkg -i jdk-XXX_linux-x64_bin.deb</font>**。其中 -i 就是 install 的意思。

在Windows下面，控制面板里面有程序管理，我们可以查看目前安装了哪些软件，可以删除这些软件。

![img](./assets/4c0cddd6f5ea77bc4aeabc135e6e8a9b.png)

在 Linux 下面，凭借**<font color="red">rpm -qa</font>**和**<font color="red">dpkg -l</font>**就可以查看安装的软件列表，-q 就是query，a 就是all，-l 的意思就是 list。

如果真的去运行的话，你会发现这个列表很长很长，很难找到你安装的软件。如果你知道要安装的软件包含某个关键词，可以用一个很好用的搜索工具grep。

**<font color="red">rpm -qa | grep jdk</font>**，这个命令是将列出来的所有软件形成一个输出。|是管道，用于连接两个程序，前面rpm -qa的输出就放进管道里面，然后作为grep的输入，grep将在里面进行搜索带关键词jdk的行，并且输出出来。grep支持正则表达式，因此搜索的时候很灵活，再加上管道，这是一个很常用的模式。同理**<font color="red">dpkg -l | grep jdk</font>**也是能够找到的。

如果你不知道关键词，可以使用**<font color="red">rpm -qa | more</font>**和**<font color="red">rpm -qa | less</font>**这两个命令，它们可以将很长的结果分页展示出来。这样你就可以一个个来找了。

我们还是利用管道的机制。more是分页后只能往后翻页，翻到最后一页自动结束返回命令行，less是往前往后都能翻页，需要输入q返回命令行，q就是quit。

如果要删除，可以用**<font color="red">rpm -e</font>**和**<font color="red">dpkg -r</font>**。-e就是 erase，-r就是remove。

我们刚才说的都是没有软件管家的情况，后来 Windows 上有了软件管家，就方便多了。我们直接搜索一下，然后点击安装就行了。

![img](./assets/4c0cddd6f5ea77bc4aeabc135e6e8a9b-1667530430444-13.png)

Linux 也有自己的软件管家，CentOS下面是**<font color="red">yum</font>**，Ubuntu 下面是**<font color="red">apt-get</font>**。

你可以根据关键词搜索，例如搜索**<font color="red">yum search jdk</font>**和**<font color="red">apt-cache search jdk</font>**，可以搜索出很多很多可以安装的jdk版本。如果数目太多，你可以通过管道 grep、more、less 来进行过滤。

选中一个之后，我们就可以进行安装了。你可以用**<font color="red">yum install java-11-openjdk.x86_64</font>**和**<font color="red">apt-get install openjdk-9-jdk</font>**来进行安装。

安装以后，如何卸载呢？我们可以使用**<font color="red">yum erase java-11-openjdk.x86_64</font>**和**<font color="red">apt-get purge openjdk-9-jdk</font>**。

Windows 上的软件管家会有一个统一的服务端，来保存这些软件，但是我们不知道服务端在哪里。而 Linux 允许我们配置从哪里下载这些软件的，地点就在配置文件里面。

对于 CentOS 来讲，配置文件在**<font color="red">/etc/yum.repos.d/CentOS-Base.repo</font>**里。

```shell
[base]
name=CentOS-$releasever - Base - 163.com
baseurl=http://mirrors.163.com/centos/$releasever/os/$basearch/
gpgcheck=1
gpgkey=http://mirrors.163.com/centos/RPM-GPG-KEY-CentOS-7

```

对于 Ubuntu 来讲，配置文件在**<font color="red">/etc/apt/sources.list</font>**里。

```shell
deb http://mirrors.163.com/ubuntu/ xenial main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ xenial-security main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ xenial-updates main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ xenial-proposed main restricted universe multiverse
deb http://mirrors.163.com/ubuntu/ xenial-backports main restricted universe multiverse
```

这里为什么都是 163.com 呢？因为 Linux 服务器遍布全球，不能都从一个地方下载，最好选一个就近的地方下载，例如在中国，选择 163.com，就不用跨越重洋了。

**其实无论是先下载再安装，还是通过软件管家进行安装，都是下载一些文件，然后将这些文件放在某个路径下，然后在相应的配置文件中配置一下。**例如，在 Windows 里面，最终会变成 C:\Program Files 下面的一个文件夹以及注册表里面的一些配置。对应 Linux 里面会放的更散一点。例如，主执行文件会放在 /usr/bin 或者 /usr/sbin 下面，其他的库文件会放在 /var 下面，配置文件会放在 /etc 下面。

所以其实还有一种简单粗暴的方法，就是将安装好的路径直接下载下来，然后解压缩成为一个整的路径。在JDK的安装目录中，Windows有 jdk-XXX_Windows-x64_bin.zip，这是Windows下常用的压缩模式。Linux 有 jdk-XXX_linux-x64_bin.tar.gz，这是 Linux 下常用的压缩模式。

如何下载呢？Linux 上面有一个工具 wget，后面加上链接，就能从网上下载了。

下载下来后，我们就可以进行解压缩了。Windows 下可以有 winzip 之类的解压缩程序，Linux 下面默认会有 tar 程序。如果是解压缩 zip 包，就需要另行安装。

```shell
yum install zip.x86_64 unzip.x86_64
apt-get install zip unzip
```

如果是 tar.gz 这种格式的，通过 tar xvzf jdk-XXX_linux-x64_bin.tar.gz 就可以解压缩了。

对于 Windows 上 jdk 的安装，如果采取这种下载压缩包的格式，需要在系统设置的环境变量配置里面设置JAVA_HOME和PATH。

![img](./assets/ab4e83ac1300658649989a2e016ac0be.png)

在 Linux 也是一样的，通过 tar 解压缩之后，也需要配置环境变量，可以通过 `export` 命令来配置。

```shell
`export` JAVA_HOME=/root/jdk-XXX_linux-x64
`export` PATH=$JAVA_HOME/bin:$PATH
```

``export``命令仅在当前命令行的会话中管用，一旦退出重新登录进来，就不管用了，有没有一个地方可以像 Windows 里面可以配置永远管用呢？

在当前用户的默认工作目录，例如/root或者/home/cliu8 下面，有一个.bashrc 文件，这个文件是以点开头的，这个文件默认看不到，需要ls -la 才能看到，a就是 all。每次登录的时候，这个文件都会运行，因而把它放在这里。这样登录进来就会自动执行。当然也可以通过 source .bashrc 手动执行。

要编辑.bashrc 文件，可以使用文本编辑器vi，也可以使用更加友好的vim。如果默认没有安装，可以通过yum install vim 及 apt-get install vim 进行安装。

vim 就像 Windows 里面的 notepad 一样，是我们第一个要学会的工具。要不然编辑、查看配置文件，这些操作你都没办法完成。vim 是一个很复杂的工具，刚上手的时候，你只需要记住几个命令就行了。

vim hello，就是打开一个文件，名字叫 hello。如果没有这个文件，就先创建一个。

我们其实就相当于打开了一个 notepad。如果文件有内容，就会显示出来。移动光标的位置，通过上下左右键就行。如果想要编辑，就把光标移动到相应的位置，输入**i**，意思是 insert。进入编辑模式，可以插入、删除字符，这些都和 notepad 很像。要想保存编辑的文本，我们使用esc键退出编辑模式，然后输入“:”，然后在“:”后面输入命令**w**，意思是 write，这样就可以保存文本，冒号后面输入**q**，意思是 quit，这样就会退出 vim。如果编辑了，还没保存，不想要了，可以输入**q!**。

好了，掌握这些基本够用了，想了解更复杂的，你可以自己去看文档。

通过 vim .bashrc，将 `export` 的两行加入后，输入:wq，写入并且退出，这样就编辑好了。

## 运行程序

好了，装好了程序，可以运行程序了。

我们都知道 Windows 下的程序，如果后缀名是 exe，双击就可以运行了。

Linux 不是根据后缀名来执行的。它的执行条件是这样的：只要文件有 x 执行权限，都能到文件所在的目录下，通过<font color="red">./filename</font>运行这个程序。当然，如果放在 PATH 里设置的路径下面，就不用./了，直接输入文件名就可以运行了，Linux会帮你找。

这是 **Linux 执行程序最常用的一种方式，通过 shell 在交互命令行里面运行。**

这样执行的程序可能需要和用户进行交互，例如允许让用户输入，然后输出结果也打印到交互命令行上。这种方式比较适合运行一些简单的命令，例如通过 date 获取当前时间。这种模式的缺点是，一旦当前的交互命令行退出，程序就停止运行了。

这样显然不能用来运行那些需要“永远”在线的程序。比如说，运行一个博客程序，我总不能老是开着交互命令行，博客才可以提供服务。一旦我要去睡觉了，关了命令行，我的博客别人就不能访问了，这样肯定是不行的。

于是，我们就有了 Linux 运行程序的第二种方式，后台运行。

这个时候，我们往往使用<font color= "red">nohup</font>命令。这个命令的意思是 no hang up（不挂起），也就是说，当前交互命令行退出的时候，程序还要在。

当然这个时候，程序不能霸占交互命令行，而是应该在后台运行。最后加一个 &，就表示后台运行。

另外一个要处理的就是输出，原来什么都打印在交互命令行里，现在在后台运行了，输出到哪里呢？输出到文件是最好的。

最终命令的一般形式为nohup command >out.file 2>&1 &。这里面，“1”表示文件描述符 1，表示标准输出，“2”表示文件描述符 2，意思是标准错误输出，“2>&1”表示标准输出和错误输出合并了。合并到哪里去呢？到 out.file 里。

> 重定向可以看看这个链接：https://www.runoob.com/linux/linux-shell-io-redirections.html

那这个进程如何关闭呢？我们假设启动的程序包含某个关键字，那就可以使用下面的命令。

```shell
ps -ef |grep 关键字  |awk '{print $2}'|xargs kill -9
```

从这个命令中，我们多少能看出 shell 的灵活性和精巧组合。

其中 ps -ef 可以单独执行，列出所有正在运行的程序，grep 上面我们介绍过了，通过关键字找到咱们刚才启动的程序。

awk工具可以很灵活地对文本进行处理，这里的awk '{print $2}'是指第二列的内容，是运行的程序ID。我们可以通过xargs传递给kill -9，也就是发给这个运行的程序一个信号，让它关闭。如果你已经知道运行的程序ID，可以直接使用kill关闭运行的程序。

> 我们可以通过xargs传递给kill -9 。 也就是发给这个运行的程序一个信号，让它关闭。 xargs kill -9 可以批量kill 系统程序 (我们的网管项目jar包众多，这个命令是一个不错的选择)。 杀掉java进程的第二种方式: 通过 jps -l 命令列出所有正在运行的java进程，以及该进程对应的进程号，然后根据kill -9 进程号 的方式将此进程结束掉。(此种方式只能一个一个的杀掉进程，不具备批量杀掉进程的功能)

在 Windows 里面还有一种程序，称为服务。这是系统启动的时候就在的，我们可以通过控制面板的服务管理启动和关闭它。

![img](./assets/f24f0f11bcb9a177861a4782ba1d82a6.png)

Linux 也有相应的服务，这就是**程序运行的第三种方式，以服务的方式运行**。例如常用的数据库 MySQL，就可以使用这种方式运行。

> 三种类型 前台 后台 服务

例如在 Ubuntu 中，我们可以通过 apt-get install mysql-server 的方式安装 MySQL，然后通过命令<font color="red">systemctl start mysql</font>启动 MySQL，通过<font color="red">systemctl enable mysql</font>设置开机启动。之所以成为服务并且能够开机启动，是因为在 /lib/systemd/system 目录下会创建一个 XXX.service 的配置文件，里面定义了如何启动、如何关闭。

在 CentOS 里有些特殊，MySQL被Oracle 收购后，因为担心授权问题，改为使用MariaDB，它是MySQL的一个分支。通过命令<font color="red">yum install mariadb-server mariadb</font>进行安装，命令<font color="red">systemctl start mariadb</font>启动，命令<font color="red">systemctl enable mariadb</font>设置开机启动。同理，会在/usr/lib/systemd/system目录下，创建一个XXX.service的配置文件，从而成为一个服务。

> 只有通过自动安装的方式如 yum install 才会在 /lib/systemd/system目录下生成对应的xxx.service，然后通过 systemctl enable xxx 就会将xxx.service 创建快捷链接复制到 /etc/systemd/system/multi-user.target.wants，这样系统在启动时会自动启动该目录下的程序 
>
> 以docker为例， 
>
> 执行 systemctl enable docker，会输出： Created symlink from /etc/systemd/system/multi-user.target.wants/docker.service to /usr/lib/systemd/system/docker.service 
>
> 执行systemctl disable docker，会输出： Removed symlink /etc/systemd/system/multi-user.target.wants/docker.service，也就表示在该目录下删除了docker.service

systemd 的机制十分复杂，这里咱们不讨论。如果有兴趣，你可以自己查看相关文档。
> 手册内容太多，阮一峰总结的很好，可以看看：http://www.ruanyifeng.com/blog/2016/03/systemd-tutorial-commands.html

最后咱们要学习的是如何关机和重启。这个就很简单啦。shutdown -h now是现在就关机，reboot就是重启。

## 总结时刻

好了，掌握这些基本命令足够你熟练操作 Linux 了。如果你是个初学者，这些命令估计看起来还是很多。我把今天这些基本的命令以及对应的操作总结了一下，方便你操作和查阅。

你不用刻意去死记硬背，按照我讲的这个步骤，从设置用户和密码、浏览文件、安装软件，最后到运行程序，自己去操作几遍，再自己整理一遍，手脑并用，加深理解，巩固记忆，效果可能会更好。

![img](./assets/8855bb645d8ecc35c80aa89cde5d16e5.jpg)

## 课堂练习

现在你应该已经学会了安装 JDK 和 MySQL，你可以尝试搭建一个基于 Java+MySQL 的服务端应用，上手使用一下。
