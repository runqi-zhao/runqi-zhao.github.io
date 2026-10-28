---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
前面四节中，已经介绍了Docker运行的本质：

一个“容器”，实际上是一个由Linux Namespace、Linux Cgroups和rootffs三种技术偶见出来的进程的各路ii环境。

从这个结构中，不难看出，一个正在运行Linxux容器，可以被“一分为而”地看待：

1. 一个联合挂载在/var/lib/docker/aufs/mnt上地rootfs。这一部分我们成为“容器镜像”（Container Image），是容器地动态试图。
2. 一个由Namespace+Cgroups构建的隔离环境，这一部分我们称为“容器运行时”（Container Runtime），是容器地动态试图。

**容器从一个开发者手里地小工具，一跃成为了云计算领域的绝对主角；而能够定义容器组织和管理规范的“容器编排”技术，当仁不让地坐上了容器领域的“头把交椅”。**

k8s项目的里面基础主要是采用Google的Borg论文。

![](https://img-1312072469.cos.ap-nanjing.myqcloud.com/c7ed0043465bccff2efc1a1257e970bd.png)

k8s依照Google的技术架构，从刚开始就站上了别的企业无法企及的技术阶段：Kubernetes 项目在 Borg 体系的指导下，体现出了一种独有的“先进性”与“完备性”，而这些特质才是一个基础设施领域开源项目赖以生存的核心价值。

## Kubernetes项目解决的问题

> 实际上k8s已经逐渐演进为云原生下的”操作系统”。操作系统一般有存储，网络，进程管理，进程调度，系统调用api等功能。类似的k8s也提供了云原生下的存储，调度，网络，声明式api。而且还带有一些运维能力，比如备份，扩缩容，负载均衡等。 它解决的问题和所在的高度就和swarm，compose不一样。

对于大多数用户来说，他们希望 Kubernetes 项目带来的体验是确定的：现在我有了应用的容器镜像，请帮我在一个给定的集群上把这个应用运行起来。

更进一步地说，我还希望 Kubernetes 能给我提供路由网关、水平扩展、监控、备份、灾难恢复等一系列运维能力。

那么，kubernetes项目于PASS项目的区别是什么？

在定义核心功能的过程中，Kubernetes 项目正是依托着 Borg 项目的理论优势，才在短短几个月内迅速站稳了脚跟，进而确定了一个如下图所示的全局架构：

![img](https://img-1312072469.cos.ap-nanjing.myqcloud.com/8ee9f2fa987eccb490cfaa91c6484f67.png)

结构主要如下：

1. 控制节点（Master节点），由三个金木写作的独立组件组成而成，它们分别是
   1. 负责API服务的kube-apiserver；
   2. 负责调度的kube-scheduler；
   3. 以及负责容器编排的kube-controller-manager。整个集群的持久化数据，则由kube-apiserver处理后保存在Ectd中
2. 计算节点，计算节点中的关键部分，就是kubelet的组件。
   1. **在k8s上面，kubelet主要负责同容器运行时（比如Docker项目）打交道。**而这个交互所依赖的，是一个称作CRI（Container Runtime Interface）的远程调用接口，这个接口定义了容器运行时的各项核心操作：比如：启动一个容器需要的所有参数。
   2. 而具体的容器运行时，比如 Docker 项目，则一般通过 OCI 这个容器运行时规范同底层的 Linux 操作系统进行交互，即：把 CRI 请求翻译成对 Linux 操作系统的调用（操作 Linux Namespace 和 Cgroups 等）。
   3. **此外，kubelet 还通过 gRPC 协议同一个叫作 Device Plugin 的插件进行交互。**
   4. **kubelet 的另一个重要功能，则是调用网络插件和存储插件为容器配置网络和持久化存储。**
   5. kubelet 完全就是为了实现 Kubernetes 项目对容器的管理能力而重新实现的一个组件，与 Borg 之间并没有直接的传承关系。

## Borg对于Kuberbetes项目的指导作用

体现在Master节点上面。

虽然在 Master 节点的实现细节上 Borg 项目与 Kubernetes 项目不尽相同，但它们的出发点却高度一致，即：如何编排、管理、调度用户提交的作业？

**从一开始，Kubernetes 项目就没有像同时期的各种“容器云”项目那样，把 Docker 作为整个架构的核心，而仅仅把它作为最底层的一个容器运行时实现。**

而k8s要解决的问题，正式Borg研究人员在论文中一个重要的观点：

> 运行在大规模集群中的各种任务之间，实际上存在着各种各样的关系。这些关系的处理，才是作业编排和管理系统最困难的地方。

同属一个服务单位不同功能之间，完全可能存在不同服务的互相调用（比如说一个网页使用到一个数据库，一般情况下都是部署到不同的机器之上，那么在k8s之中，应该如何进行部署）？

**Kubernetes 项目最主要的设计思想是，从更宏观的角度，以统一的方式来定义任务之间的各种关系，并且为将来支持更多种类的关系留有余地。**

在常规环境下，这些应用往往会被直接部署在同一台机器上，通过 Localhost 通信，通过本地磁盘目录交换文件。而在 Kubernetes 项目中，这些容器则会被划分为一个“Pod”，Pod 里的容器共享同一个 Network Namespace、同一组数据卷，从而达到高效率交换信息的目的。

对于一个容器来说，它的 IP 地址等信息不是固定的，那么 Web 应用又怎么找到数据库容器的 Pod 呢？

所以，Kubernetes 项目的做法是给 Pod 绑定一个 Service 服务，而 Service 服务声明的 IP 地址等信息是“终生不变”的。这个 **Service 服务的主要作用，就是作为 Pod 的代理入口（Portal），从而代替 Pod 对外暴露一个固定的网络地址。**

围绕了Pod不断像真实地技术场景地拓展，我们就能摸索出如下所示地k8s项目核心功能地“全景图”：

![img](https://img-1312072469.cos.ap-nanjing.myqcloud.com/16c095d6efb8d8c226ad9b098689f306.png)

按照这幅图的线索，我们从容器这个最基础的概念出发，首先遇到了容器间“紧密协作”关系的难题，于是就扩展到了 Pod；有了 Pod 之后，我们希望能一次启动多个应用的实例，这样就需要 Deployment 这个 Pod 的多实例管理器；而有了这样一组相同的 Pod 后，我们又需要通过一个固定的 IP 地址和端口以负载均衡的方式访问它，于是就有了 Service。

**除了应用与应用之间的关系外，应用运行的形态是影响“如何容器化这个应用”的第二个重要因素。**

为此，k8s定义了新的、基于Pod改进后地对象。比如Job，用来描述一次性运行的Pod（比如大数据任务）。

如此种种，正式kubernetes项目定义容器关系和形态的主要方法。

相比之下，在kubernetes项目中，我们所推崇的使用方法是：

- 首先，通过一个“编排对象”，比如 Pod、Job、CronJob 等，来描述你试图管理的应用；
- 然后，再为它定义一些“服务对象”，比如 Service、Secret、Horizontal Pod Autoscaler（自动水平扩展器）等。这些对象，会负责具体的平台级功能。

**这种使用方法，就是所谓的“声明式 API”。这种 API 对应的“编排对象”和“服务对象”，都是 Kubernetes 项目中的 API 对象（API Object）。**

## kubernetes项目如何启动一个容器化任务

编写yaml文档进行启动：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.7.9
        ports:
        - containerPort: 80
```

在上面这个 YAML 文件中，我们定义了一个 Deployment 对象，它的主体部分（spec.template 部分）是一个使用 Nginx 镜像的 Pod，而这个 Pod 的副本数是 2（replicas=2）。

然后执行：

```shell
$ kubectl create -f nginx-deployment.yaml
```

这样，两个完全相同的 Nginx 容器副本就被启动了。

## 总结

容器可以分为两个部分：容器运行时和容器镜像。

了解k8s的结构。

k8s项目的本质，就是为用户提供一个具有普遍意义的容器编排工具。

## 思考题

1. 这今天的分享中，我介绍了 Kubernetes 项目的架构。你是否了解了 Docker Swarm（SwarmKit 项目）和 Kubernetes 在架构上和使用方法上的异同呢？

   > 从微服务架构来讲，多个独立功能内聚的服务带来了整体的灵活性，但是同时也带来了部署运维的复杂度提升，这时Docker配合Devops带来了不少的便利(轻量、隔离、一致性、CI、CD等)解决了不少问题，再配合compose，看起来一切都很美了，为什么还需要K8s？可以试着这样理解么？把微服务理解为人，那么服务治理其实就是人之间的沟通而已，人太多了就需要生存空间和沟通方式的优化，这就需要集群和编排了。Docker Compose，swarm，可以解决少数人之间的关系，比如把手机号给你，你就可以方便的找到我，但是如果手机号变更的时候就会麻烦，人多了也会麻烦。而k8s是站在上帝视角俯视芸芸众生后的高度抽象，他看到了大概有哪些类人(组织）以及不同组织有什么样的特点（Job、CornJob、Autoscaler、StatefulSet、DaemonSet...），不同组织之间交流可能需要什么（ConfigMap,Secret...）,这样比价紧密的人们在相同pod中，通过Service-不会变更的手机号，来和不同的组织进行沟通，Deployment、RC则可以帮组人们快速构建组织。Dokcer 后出的swarm mode，有类似的视角抽象（比如Service），不过相对来说并不完善。

2. 在 Kubernetes 之前，很多项目都没办法管理“有状态”的容器，即，不能从一台宿主机“迁移”到另一台宿主机上的容器。你是否能列举出，阻止这种“迁移”的原因都有哪些呢？