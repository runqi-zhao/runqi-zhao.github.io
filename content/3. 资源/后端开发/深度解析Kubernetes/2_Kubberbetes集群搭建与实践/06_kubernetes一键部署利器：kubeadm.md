---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
没有kubeadm的部署方法：使用SlatStack、Ansible等运维工具自动化地执行这些步骤。

kubeadm地作用：一键化地进行部署。

```shell
# 创建一个Master节点
$ kubeadm init

# 将一个Node节点加入到当前集群中
$ kubeadm join <Master节点的IP和端口>
```

Kubernetes 的功能那么多，这样一键部署出来的集群，能用于生产环境吗？

## kubeadm的工作原理

在[[05_从容器到容器云：谈谈kubernetes的本质]]中已经讲解Kubernetes的架构和它的组件。

**为什么不用容器部署kubernetes呢？**

事实上，在 Kubernetes 早期的部署脚本里，确实有一个脚本就是用 Docker 部署 Kubernetes 项目的，这个脚本相比于 SaltStack 等的部署方式，也的确简单了不少。

**但是，这样做会带来一个很麻烦的问题，即：如何容器化 kubelet。**

kublect是Kubernetes 项目用来操作 Docker 等容器运行时的核心组件。可是，除了跟容器运行时打交道外，kubelet 在配置容器网络、管理容器数据卷时，都需要直接操作宿主机。

kubelet容器可以通过不开启Network Namespace（即Docker的host nnetwork模式）的方式，直接共享宿主机的网络栈。可是，要让 kubelet 隔着容器的 Mount Namespace 和文件系统，操作宿主机的文件系统，就有点儿困难了。

到目前为止，在容器中运行kubelect并没有很好的解决方案，此时kubeadm采用了一种托这些的方案：

**把kubelet直接运行在宿主机上，然后使用容器部署其他的kubernetes组件**

可以使用下面命令安装kubeadm：

```shell
$ apt-get install kubeadm
```

接下来，你就可以使用“kubeadm init”部署 Master 节点了。

## kubeadm init 工作原理

1. 首先需要做的是一系列的检查工作，以确定这条在机器可以用来部署kubernetes。 这一步检查，称之为“Preflight Checks”。

   其实，Preflight Checks 包括了很多方面，比如：

   - Linux 内核的版本必须是否是 3.10 以上？Linux Cgroups 模块是否可用？
   - 机器的 hostname 是否标准？在 Kubernetes 项目里，机器的名字以及一切存储在 Etcd 中的 API 对象，都必须使用标准的 DNS 命名（RFC 1123）。
   - 用户安装的 kubeadm 和 kubelet 的版本是否匹配？
   - 机器上是不是已经安装了 Kubernetes 的二进制文件？
   - Kubernetes 的工作端口 10250/10251/10252 端口是不是已经被占用？
   - ip、mount 等 Linux 指令是否存在？Docker 是否已经安装？
   - ……

2. **在通过了 Preflight Checks 之后，kubeadm 要为你做的，是生成 Kubernetes 对外提供服务所需的各种证书和对应的目录。**

   1. kubernetes对外提供服务时，除非专门开启“不安全模式”，否则都要通过HTTPS才能访问kube-apiserver。这时就需要为kubernetes集群配置好证书文件。

   2. kubeadm 为 Kubernetes 项目生成的证书文件都放在 Master 节点的 /etc/kubernetes/pki 目录下。在这个目录下，最主要的证书文件是 ca.crt 和对应的私钥 ca.key。

   3. 用户使用 kubectl 获取容器日志等 streaming 操作时，需要通过 kube-apiserver 向 kubelet 发起请求，这个连接也必须是安全的。kubeadm 为这一步生成的是 apiserver-kubelet-client.crt 文件，对应的私钥是 apiserver-kubelet-client.key。

   4. Kubernetes 集群中还有 Aggregate APIServer 等特性，也需要用到专门的证书

   5. 可以选择不生成证书，而是拷贝现有的证书到如下证书的目录中：

      ```shell
      /etc/kubernetes/pki/ca.{crt,key}
      ```

3. 证书生成后，kubeadm 接下来会为其他组件生成访问 kube-apiserver 所需的配置文件。这些文件保存的路径的是：/etc/kubernetes/xxx.conf。

   这些文件里面记录的是，当前这个 Master 节点的服务器地址、监听端口、证书目录等信息。这样，对应的客户端（比如 scheduler，kubelet 等），可以直接加载相应的文件，使用里面的信息与 kube-apiserver 建立安全连接。

4. **接下来，kubeadm 会为 Master 组件生成 Pod 配置文件。**

   我已经在[[05_从容器到容器云：谈谈kubernetes的本质]]中和你介绍过 Kubernetes 有三个 Master 组件 kube-apiserver、kube-controller-manager、kube-scheduler，而它们都会被使用 Pod 的方式部署起来。

   提问：kubernetes集群上不存在，难道kubeadm会直接执行docker run这些容器吗？答案是否定的。

   - 在 Kubernetes 中，有一种特殊的容器启动方法叫做“Static Pod”。它允许你把要部署的 Pod 的 YAML 文件放在一个指定的目录里。这样，当这台机器上的 kubelet 启动时，它会自动检查这个目录，加载所有的 Pod YAML 文件，然后在这台机器上启动它们。
   - 在 kubeadm 中，Master 组件的 YAML 文件会被生成在 /etc/kubernetes/manifests 路径下。

5. 在这一步完成后，kubeadm 还会再生成一个 Etcd 的 Pod YAML 文件，用来通过同样的 Static Pod 的方式启动 Etcd。所以，最后 Master 组件的 Pod YAML 文件如下所示：

```shell
$ ls /etc/kubernetes/manifests/
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
```

6. 然后，kubeadm就会为集群生成一个bootstrap token，在后面，只要持有这个token，任何一个安装了kubelet和kubadm的节点，都可以kubeadm join加入大这个集群之中。
7. **在 token 生成之后，kubeadm 会将 ca.crt 等 Master 节点的重要信息，通过 ConfigMap 的方式保存在 Etcd 当中，供后续部署 Node 节点使用。**
8. **kubeadm init 的最后一步，就是安装默认插件。**Kubernetes 默认 kube-proxy 和 DNS 这两个插件是必须安装的。它们分别用来提供整个集群的服务发现和 DNS 功能。

## kubeadm join的工作流程

这个流程很简单，kubeadm init生成bootstrap token之后，在任意一台安装了 kubelet 和 kubeadm 的机器上执行 kubeadm join 了。

可是，为什么执行 kubeadm join 需要这样一个 token 呢？

因为，任何一台机器想要成为 Kubernetes 集群中的一个节点，就必须在集群的 kube-apiserver 上注册。可是，要想跟 apiserver 打交道，这台机器就必须要获取到相应的证书文件（CA 文件）。可是，为了能够一键安装，我们就不能让用户去 Master 节点上手动拷贝这些文件。

所以，kubeadm 至少需要发起一次“不安全模式”的访问到 kube-apiserver，从而拿到保存在 ConfigMap 中的 cluster-info（它保存了 APIServer 的授权信息）。而 bootstrap token，扮演的就是这个过程中的安全验证的角色。

只要有了 cluster-info 里的 kube-apiserver 的地址、端口、证书，kubelet 就可以以“安全模式”连接到 apiserver 上，这样一个新的节点就部署完成了。

## 配置kubeadm的参数

kubeadm 确实简单易用，可是我又该如何定制我的集群组件参数呢？

比如，我要指定 kube-apiserver 的启动参数，该怎么办？

这里推荐使用kubeadm init部署Master节点是，使用下面这条命令：

```shell
$ kubeadm init --config kubeadm.yaml
```

这是，你可以提供一个yaml文件（比如，kubeadm,yaml），所展示的内容如下：

```yaml
apiVersion: kubeadm.k8s.io/v1alpha2
kind: MasterConfiguration
kubernetesVersion: v1.11.0
api:
  advertiseAddress: 192.168.0.102
  bindPort: 6443
  ...
etcd:
  local:
    dataDir: /var/lib/etcd
    image: ""
imageRepository: k8s.gcr.io
kubeProxy:
  config:
    bindAddress: 0.0.0.0
    ...
kubeletConfiguration:
  baseConfig:
    address: 0.0.0.0
    ...
networking:
  dnsDomain: cluster.local
  podSubnet: ""
  serviceSubnet: 10.96.0.0/12
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  ...
```

通过制定这样一个部署参数配置文件，你就可以很方便地在这个文件里填写各种自定义的部署参数了。比如，我现在要指定 kube-apiserver 的参数，那么我只要在这个文件里加上这样一段信息：

```yaml
...
apiServerExtraArgs:
  advertise-address: 192.168.0.103
  anonymous-auth: false
  enable-admission-plugins: AlwaysPullImages,DefaultStorageClass
  audit-log-path: /home/johndoe/audit.log
```

然后，kubeadm 就会使用上面这些信息替换 /etc/kubernetes/manifests/kube-apiserver.yaml 里的 command 字段里的参数了。

而这个 YAML 文件提供的可配置项远不止这些。比如，你还可以修改 kubelet 和 kube-proxy 的配置，修改 Kubernetes 使用的基础镜像的 URL（默认的k8s.gcr.io/xxx镜像 URL 在国内访问是有困难的），指定自己的证书文件，指定特殊的容器运行时等等。

## 总结

总体层面上介绍了kubeadm的工作原理。

kubeadm 不能够用于生产环境。

## 思考题

1. 在 Linux 上为一个类似 kube-apiserver 的 Web Server 制作证书，你知道可以用哪些工具实现吗？
2. 回忆一下我在前面文章中分享的 Kubernetes 架构，你能够说出 Kubernetes 各个功能组件之间（包含 Etcd），都有哪些建立连接或者调用的方式吗？（比如：HTTP/HTTPS，远程调用等等）
