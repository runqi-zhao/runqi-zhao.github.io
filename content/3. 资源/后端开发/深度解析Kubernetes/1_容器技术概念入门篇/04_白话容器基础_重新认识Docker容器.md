---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
在前面的[[01_白话容器基础：从进程说起]] [[02_白话容器基础_限制与隔离]] [[03_白话容器基础_深入理解容器镜像]]，我分别从 Linux Namespace 的隔离能力、Linux Cgroups 的限制能力，以及基于 rootfs 的文件系统三个角度，为你剖析了一个 Linux 容器的核心实现原理。

## 从一个例子说起

下面 Docker 部署一个用 Python 编写的 Web 应用。这个应用的代码部分（app.py）非常简单：

```python
from flask import Flask
import socket
import os

app = Flask(__name__)

@app.route('/')
def hello():
    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>"           
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname())
    
if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```

这个应用的依赖，则被定义在了同目录下的 requirements.txt 文件里，内容如下所示：

```shell
$ cat requirements.txt
Flask
```

### 制作镜像

使用下面命令的Dockerfile文件制作镜像：

```dockerfile

# 使用官方提供的Python开发镜像作为基础镜像
FROM python:2.7-slim

# 将工作目录切换为/app
WORKDIR /app

# 将当前目录下的所有内容复制到/app下
ADD . /app

# 使用pip命令安装这个应用所需要的依赖
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# 允许外界访问容器的80端口
EXPOSE 80

# 设置环境变量
ENV NAME World

# 设置容器进程为：python app.py，即：这个Python应用的启动命令
CMD ["python", "app.py"]
```

文件不在详细赘述，使用这个文件的好处就是可以直接可以使用一个打包好的镜像，省得自己再去进行配置。

注意，最后一行的CMD的含义是 Dockerfile 指定 python app.py 为这个容器的进程。这里，app.py 的实际路径是 /app/app.py。所以，CMD ["python", "app.py"]等价于"docker run python app.py"。另外，在使用 Dockerfile 时，你可能还会看到一个叫作 ENTRYPOINT 的原语。

实际上，它和 CMD 都是 Docker 容器进程启动所必需的参数，完整执行格式是：“ENTRYPOINT CMD”。但是，默认情况下，Docker 会为你提供一个隐含的 ENTRYPOINT，即：/bin/sh -c。所以，在不指定 ENTRYPOINT 时，比如在我们这个例子里，实际上运行在容器里的完整进程是：/bin/sh -c "python app.py"，即 CMD 的内容就是 ENTRYPOINT 的参数。

接下来，将这个dockerfile打上标签，然后推送到自己的hubDocker之中，就能完成在任何机器上面进行下载了。

```shell
$ docker build -t helloworld .
$ docker run -p 4000:80 helloworld
$ docker tag helloworld geektime/helloworld:v1
```

接下来，我们可以使用docker commit指令和，把一个正在运行的容器，直接提交为一个镜像。好处是对于一个已经运行起来的容器，需要再次进行一些操作，并且把结果保存到镜像里面，比如：

```shell
$ docker exec -it 4ddf4638572d /bin/sh
# 在容器内部新建了一个文件
root@4ddf4638572d:/app# touch test.txt
root@4ddf4638572d:/app# exit

#将这个新建的文件提交到镜像中保存
$ docker commit 4ddf4638572d geektime/helloworld:v2
```

<font color="red">那么，docker  exec是怎么做到进入容器里面的呢?</font>

## docker exec

一个进程的NameSpace在宿主机上面确实存在的，并且是一个文件的方式存在。

```shell
$ docker inspect --format '{{ .State.Pid }}'  4ddf4638572d
25686
```

接下来，可以通过宿主机的proc文件，看到这个25686进程的所有Namespace对应的文件：

```shell
$ ls -l  /proc/25686/ns
total 0
lrwxrwxrwx 1 root root 0 Aug 13 14:05 cgroup -> cgroup:[4026531835]
lrwxrwxrwx 1 root root 0 Aug 13 14:05 ipc -> ipc:[4026532278]
lrwxrwxrwx 1 root root 0 Aug 13 14:05 mnt -> mnt:[4026532276]
lrwxrwxrwx 1 root root 0 Aug 13 14:05 net -> net:[4026532281]
lrwxrwxrwx 1 root root 0 Aug 13 14:05 pid -> pid:[4026532279]
lrwxrwxrwx 1 root root 0 Aug 13 14:05 pid_for_children -> pid:[4026532279]
lrwxrwxrwx 1 root root 0 Aug 13 14:05 user -> user:[4026531837]
lrwxrwxrwx 1 root root 0 Aug 13 14:05 uts -> uts:[4026532277]
```

**这也就意味着，一个进程，可以选择加入到某个进程已有的Namespace当中，从而达到“进入”这个进程所在容器的目的，这就是docker exec的原理。**

这个操作所依赖的，乃是一个名叫sents()的Linux系统调用。下面一段程序展示了这个例子：

```c
#define _GNU_SOURCE
#include <fcntl.h>
#include <sched.h>
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

#define errExit(msg) do { perror(msg); exit(EXIT_FAILURE);} while (0)

int main(int argc, char *argv[]) {
    int fd;
    
    fd = open(argv[1], O_RDONLY);
    if (setns(fd, 0) == -1) {
        errExit("setns");
    }
    execvp(argv[2], &argv[2]); 
    errExit("execvp");
}
```

这段代码功能非常简单：它一共接收两个参数，第一个参数是 argv[1]，即当前进程要加入的 Namespace 文件的路径，比如 /proc/25686/ns/net；而第二个参数，则是你要在这个 Namespace 里运行的进程，比如 /bin/bash。

这段代码的核心操作，则是通过 open() 系统调用打开了指定的 Namespace 文件，并把这个文件的描述符 fd 交给 setns() 使用。在 setns() 执行后，当前进程就加入了这个文件对应的 Linux Namespace 当中了。现在，你可以编译执行一下这个程序，加入到容器进程（PID=25686）的 Network Namespace 中：

```shell

$ gcc -o set_ns set_ns.c 
$ ./set_ns /proc/25686/ns/net /bin/bash 
$ ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:ac:11:00:02  
          inet addr:172.17.0.2  Bcast:0.0.0.0  Mask:255.255.0.0
          inet6 addr: fe80::42:acff:fe11:2/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:12 errors:0 dropped:0 overruns:0 frame:0
          TX packets:10 errors:0 dropped:0 overruns:0 carrier:0
     collisions:0 txqueuelen:0 
          RX bytes:976 (976.0 B)  TX bytes:796 (796.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          inet6 addr: ::1/128 Scope:Host
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
    collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

那么，为什么只能看到两个网卡呢，讲道理，宿主机上面应该有很多个网卡？

实际上，在 setns() 之后我看到的这两个网卡，正是我在前面启动的 Docker 容器里的网卡。也就是说，我新创建的这个 /bin/bash 进程，由于加入了该容器进程（PID=25686）的 Network Namepace，它看到的网络设备与这个容器里是一样的，即：/bin/bash 进程的网络设备视图，也被修改了。

而一旦一个进程加入到了另一个NameSpace当中，在宿主机的Namespace文件上，也会有所体现。

在宿主机上，可以使用ps指令找到set_ns程序执行的/bin/bash进程，真实的PID是28499：

```shell
# 在宿主机上
ps aux | grep /bin/bash
root     28499  0.0  0.0 19944  3612 pts/0    S    14:15   0:00 /bin/bash
```

这时，如果按照前面介绍过的方法，查看一下这个 PID=28499 的进程的 Namespace，你就会发现这样一个事实：

```shell
$ ls -l /proc/28499/ns/net
lrwxrwxrwx 1 root root 0 Aug 13 14:18 /proc/28499/ns/net -> net:[4026532281]
$ ls -l  /proc/25686/ns/net
lrwxrwxrwx 1 root root 0 Aug 13 14:05 /proc/25686/ns/net -> net:[4026532281]
```

在 /proc/[PID]/ns/net 目录下，这个 PID=28499 进程，与我们前面的 Docker 容器进程（PID=25686）指向的 Network Namespace 文件完全一样。这说明这两个进程，共享了这个名叫 net:[4026532281]的 Network Namespace。

此外，Docker 还专门提供了一个参数，可以让你启动一个容器并“加入”到另一个容器的 Network Namespace 里，这个参数就是 -net，比如:

```shell
$ docker run -it --net container:4ddf4638572d busybox ifconfig
```

当碰到奇怪问题的使用，可以使用--net=host进行指定。

<font color="red">说了这么多，其实就是想要表达在docker exec这个操作之后，Linux NameSpace更具体的工作原理</font>

## docker commit

docker commit实际上就是在容器运行起来后，把最上层的“可读写层”，加上原先容器镜像的只读层，打包组成了一个新的镜像。

而由于使用了联合文件系统，你在容器里对镜像 rootfs 所做的任何修改，都会被操作系统先复制到这个可读写层，然后再修改。这就是所谓的：Copy-on-Write。

## Volume（数据卷）

容器技术使用了rootfs机制和Mount Namespace。构建出了一个同宿柱苣完全隔离开的文件系统环境，此时，就需要考虑两个问题：

1. 容器里进程新建的文件，怎么才能让宿主机获取到？
2. 宿主机上的文件和目录，怎么才能让容器里的进程访问到？

这正是Docker Volume要解决的问题：**Volume机制，允许你将宿主机上指定的目录或者文件，挂载到容器里面进行读取和修改操作。**

Docker项目中，指令Volume的两种声明方式：

```shell
$ docker run -v /test ...
$ docker run -v /home:/test ...
```

而这两种声明方式的本质，实际上是相同的：都是把一个宿主机的目录挂载进了容器的 /test 目录。

Docker是如何做到把一个宿主机上的目录或者文件，挂载到容器里面去呢？

在[[03_白话容器基础_深入理解容器镜像]]中已经介绍过，当容器创建之后，尽管开启了Mount Namespace，但是在执行chroot（或者pivot_root）之前，容器进程可以看到宿主机上的整个文件系统。

> 这就是真正的操作，在 Mount Namespace之后， chroot之前，把需要的目录挂载进来

因此，真实的流程可以这么理解：

1. 启动容器进程
2. 准备rootfs：将容器镜像的各个文件挂载到/var/lin/docker/aufs/mnt/[可读写层ID]/test目录上
3. 挂载volume目录：将生命的volume目录挂载到上述的目录中
4. 执行chroot，是文件系统隔离生效。

执行这个挂在操作的时候，“容器进程”已经创建了，也就意味着此时Mount Namespace已经开启了。所以，整个挂载时间只有在整个容器中是可见的。在宿主上，咯不见宿主机上容器内部这个挂载点。**保证了容器的隔离性不会被Volume打破。**

> 注意：这里提到的"容器进程"，是 Docker 创建的一个容器初始化进程 (dockerinit)，而不是应用进程 (ENTRYPOINT + CMD)。dockerinit 会负责完成根目录的准备、挂载设备和目录、配置 hostname 等一系列需要在容器内进行的初始化操作。最后，它通过 execv() 系统调用，让应用进程取代自己，成为容器里的 PID=1 的进程。

使用到挂载技术，实际上Linux的**绑定挂载（bind mount）机制**。主要作用是，允许将一个目录或者文件，而不是整个设备，挂载到指定的目录上。并且，这是你在挂载点上进行的任何操作，只是发生在被挂载的目录或者文件上，而原挂载点的内容则会被隐藏不受影响。

![img](https://img-1312072469.cos.ap-nanjing.myqcloud.com/95c957b3c2813bb70eb784b8d1daedc6.png)

如上图所示，表达的很清楚，不在详细赘述。

**在一个正确的时机，进行一次绑定挂载，Docker 就可以成功地将一个宿主机上的目录或文件，不动声色地挂载到容器中。**

接下来，修改这个test信息，则test信息本身不会修改，只会修改/home目录。

## 总结

Docker容器的全景图如下图所示：

![img](https://img-1312072469.cos.ap-nanjing.myqcloud.com/3116751445d182687ce496f2825117e5.jpg)

这个容器进程“python app.py”，运行在由 Linux Namespace 和 Cgroups 构成的隔离环境里；而它运行所需要的各种文件，比如 python，app.py，以及整个操作系统文件，则由多个联合挂载在一起的 rootfs 层提供。

这些 rootfs 层的最下层，是来自 Docker 镜像的只读层。

在只读层之上，是 Docker 自己添加的 Init 层，用来存放被临时修改过的 /etc/hosts 等文件。

而 rootfs 的最上层是一个可读写层，它以 Copy-on-Write 的方式存放任何对只读层的修改，容器声明的 Volume 的挂载点，也出现在这一层。通

## 思考题

1. 你在查看 Docker 容器的 Namespace 时，是否注意到有一个叫 cgroup 的 Namespace？它是 Linux 4.6 之后新增加的一个 Namespace，你知道它的作用吗？

2. 如果你执行 docker run -v /home:/test 的时候，容器镜像里的 /test 目录下本来就有内容的话，你会发现，在宿主机的 /home 目录下，也会出现这些内容。这是怎么回事？为什么它们没有被绑定挂载隐藏起来呢？（提示：Docker 的“copyData”功能）

   docker的copyData功能可以使挂载点文件和挂载目录下的文件同时存在

3. 请尝试给这个 Python 应用加上 CPU 和 Memory 限制，然后启动它。根据我们前面介绍的 Cgroups 的知识，请你查看一下这个容器的 Cgroups 文件系统的设置，是不是跟我前面的讲解一致。

   docker run -it --cpuset-cpus="0-2" -m 500M helloworld 限制cpu只能用 0，1，2 内存500MB