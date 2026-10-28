---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
为什么 Kubernetes 项目又突然搞出一个 Pod 来呢？

容器的本质是什么？

容器的本质是进程。

kubernetes是操作系统。

那么，现在登录到一台Linux机器里，执行一条如下所示的命令：

```shell
$ pstree -g
```

这条命令的作用是展示当前系统中正在运行的进程的树状结构，返回结果如下：

```shell

systemd(1)-+-accounts-daemon(1984)-+-{gdbus}(1984)
           | `-{gmain}(1984)
           |-acpid(2044)
          ...      
           |-lxcfs(1936)-+-{lxcfs}(1936)
           | `-{lxcfs}(1936)
           |-mdadm(2135)
           |-ntpd(2358)
           |-polkitd(2128)-+-{gdbus}(2128)
           | `-{gmain}(2128)
           |-rsyslogd(1632)-+-{in:imklog}(1632)
           |  |-{in:imuxsock) S 1(1632)
           | `-{rs:main Q:Reg}(1632)
           |-snapd(1942)-+-{snapd}(1942)
           |  |-{snapd}(1942)
           |  |-{snapd}(1942)
           |  |-{snapd}(1942)
           |  |-{snapd}(1942)
```

所有进程之间都是相互合作的。

kubernetes项目所做的，就是将“进程组”的概念映射到容器技术中，并使其成为了这个云计算“操作系统”里的“一等公民”。

Kubernetes 项目之所以要这么做的原因，我在前面介绍 Kubernetes 和 Borg 的关系时曾经提到过：在 Borg 项目的开发和实践过程中，Google 公司的工程师们发现，他们部署的应用，往往都存在着类似于“进程和进程组”的关系。更具体地说，就是这些应用之间有着密切的协作关系，使得它们必须部署在同一台机器上。

“单进程模型”并不是只容器里只能运行“一个”进程，而是指容器没有管理多个线程的能力。

Kubernetes 项目的调度器，是统一按照 Pod 而非容器的资源需求进行计算的。

而如果 Pod 的设计只是出于调度上的考虑，那么 Kubernetes 项目似乎完全没有必要非得把 Pod 作为“一等公民”吧？这不是故意增加用户的学习门槛吗？

Pod在k8s项目里面还有更重要的意义，那就是：容器设计模式。
