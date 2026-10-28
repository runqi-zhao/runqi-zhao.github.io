---
type: knowledge
status: active
moc:
  - "[[后端开发 MOC]]"
kind: tech
review_after: 2026-12-26
last_reviewed: 2026-09-27
---
作为一个应用开发者，你首先要做的，是制作容器的镜像。这一部分内容，请参见[[03_白话容器基础_深入理解容器镜像]]。

什么是kubernets项目能“认识”的方式呢？

编写对应的yaml文档。

下面是一个常见的yaml文档的示例：

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
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

像这样的一个 YAML 文件，对应到 Kubernetes 中，就是一个 API Object（API 对象）。

这个 YAML 文件中的 Kind 字段，指定了这个 API 对象的类型（Type），是一个 Deployment。所谓 Deployment，是一个定义多副本应用（即多个副本 Pod）的对象，在[[05_从容器到容器云：谈谈kubernetes的本质]]中已经讲解过。

那么，Pod的具体样子是什么样子呢？

**Pod 就是 Kubernetes 世界里的“应用”；而一个应用，可以由多个容器组成。**

像这样使用一种API对象（Deployment）管理另一种API对象（Pod）的方法，在kubernetes中，叫做“控制器”模式。

每个API对象都有一个叫做Metadata的字段，这个字段就是API对象的“标识”，即元数据，也是我们从k8s里面找到这个对象的主要依据。这其中最主要使用到的是Labels。

顾名思义，Labels 就是一组 key-value 格式的标签。而像 Deployment 这样的控制器对象，就可以通过这个 Labels 字段从 Kubernetes 中过滤出它所关心的被控制对象。

Metadata 中，还有一个与 Labels 格式、层级完全相同的字段叫 Annotations，它专门用来携带 key-value 格式的内部信息。所谓内部信息，指的是对这些信息感兴趣的，是 Kubernetes 组件本身，而不是用户。

一个 Kubernetes 的 API 对象的定义，大多可以分为 Metadata 和 Spec 两个部分。前者存放的是这个对象的元数据，对所有 API 对象来说，这一部分的字段和格式基本上是一样的；而后者存放的，则是属于这个对象独有的定义，用来描述它所要表达的功能。

在了解了上述 Kubernetes 配置文件的基本知识之后，我们现在就可以把这个 YAML 文件“运行”起来。

```shell
$ kubectl create -f nginx-deployment.yaml
```

然后，通过 kubectl get 命令检查这个 YAML 运行起来的状态是不是与我们预期的一致：

```shell
$ kubectl get pods -l app=nginx
NAME                                READY     STATUS    RESTARTS   AGE
nginx-deployment-67594d6bf6-9gdvr   1/1       Running   0          10m
nginx-deployment-67594d6bf6-v6j7w   1/1       Running   0          10m
```

**在命令行中，所有 key-value 格式的参数，都使用“=”而非“:”表示。**

接下来可以用使用kubectl describe命令，查看一个API的细节：

```shell

$ kubectl describe pod nginx-deployment-67594d6bf6-9gdvr
Name:               nginx-deployment-67594d6bf6-9gdvr
Namespace:          default
Priority:           0
PriorityClassName:  <none>
Node:               node-1/10.168.0.3
Start Time:         Thu, 16 Aug 2018 08:48:42 +0000
Labels:             app=nginx
                    pod-template-hash=2315082692
Annotations:        <none>
Status:             Running
IP:                 10.32.0.23
Controlled By:      ReplicaSet/nginx-deployment-67594d6bf6
...
Events:

  Type     Reason                  Age                From               Message

  ----     ------                  ----               ----               -------
  
  Normal   Scheduled               1m                 default-scheduler  Successfully assigned default/nginx-deployment-67594d6bf6-9gdvr to node-1
  Normal   Pulling                 25s                kubelet, node-1    pulling image "nginx:1.7.9"
  Normal   Pulled                  17s                kubelet, node-1    Successfully pulled image "nginx:1.7.9"
  Normal   Created                 17s                kubelet, node-1    Created container
  Normal   Started                 17s                kubelet, node-1    Started container
```

其中：需要特别关注的是Events。

在 Kubernetes 执行的过程中，对 API 对象的所有重要操作，都会被记录在这个对象的 Events 里，并且显示在 kubectl describe 指令返回的结果中。

**如果有异常发生，你一定要第一时间查看这些 Events，往往可以看到非常详细的错误信息。**

**接下来，如果我们要对这个 Nginx 服务进行升级，把它的镜像版本从 1.7.9 升级为 1.8，要怎么做呢？**

直接修改yaml文档并且运行对应的apply命令。

推荐使用apply而不是replace。它有助于帮助开发和运维人员，围绕着可以版本化管理的 YAML 文件，而不是“行踪不定”的命令行进行协作，从而大大降低开发人员和运维人员之间的沟通成本。

**而当应用本身发生变化时，开发人员和运维人员可以依靠容器镜像来进行同步；当应用部署参数发生变化时，这些 YAML 文件就是他们相互沟通和信任的媒介。**

最后，说明如何装在对应Volume。

两种方式：

```shell

apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.8
        ports:
        - containerPort: 80
        volumeMounts:
        - mountPath: "/usr/share/nginx/html"
          name: nginx-vol
      volumes:
      - name: nginx-vol
        emptyDir: {}
```

可以看到，我们在 Deployment 的 Pod 模板部分添加了一个 volumes 字段，定义了这个 Pod 声明的所有 Volume。它的名字叫作 nginx-vol，类型是 emptyDir。

那什么是 emptyDir 类型呢？

它其实就等同于我们之前讲过的 Docker 的隐式 Volume 参数，即：不显式声明宿主机目录的 Volume。所以，Kubernetes 也会在宿主机上创建一个临时目录，这个目录将来就会被绑定挂载到容器所声明的 Volume 目录上。

> 备注：不难看到，Kubernetes 的 emptyDir 类型，只是把 Kubernetes 创建的临时目录作为 Volume 的宿主机目录，交给了 Docker。这么做的原因，是 Kubernetes 不想依赖 Docker 自己创建的那个 _data 目录。

当然，Kubernetes 也提供了显式的 Volume 定义，它叫作 hostPath。

```yaml
	...   
    volumes:
      - name: nginx-vol
        hostPath: 
          path:  " /var/data"
```

在上述修改完成后，我们还是使用 kubectl apply 指令，更新这个 Deployment:

```shell
$ kubectl apply -f nginx-deployment.yaml
```

接下来，你可以通过 kubectl get 指令，查看两个 Pod 被逐一更新的过程：

```shell
$ kubectl get pods
NAME                                READY     STATUS              RESTARTS   AGE
nginx-deployment-5c678cfb6d-v5dlh   0/1       ContainerCreating   0          4s
nginx-deployment-67594d6bf6-9gdvr   1/1       Running             0          10m
nginx-deployment-67594d6bf6-v6j7w   1/1       Running             0          10m
$ kubectl get pods
NAME                                READY     STATUS    RESTARTS   AGE
nginx-deployment-5c678cfb6d-lg9lw   1/1       Running   0          8s
nginx-deployment-5c678cfb6d-v5dlh   1/1       Running   0          19s
```

最后，你还可以使用 kubectl exec 指令，进入到这个 Pod 当中（即容器的 Namespace 中）查看这个 Volume 目录：

```shell
$ kubectl exec -it nginx-deployment-5c678cfb6d-lg9lw -- /bin/bash
# ls /usr/share/nginx/html
```

此外，你想要从 Kubernetes 集群中删除这个 Nginx Deployment 的话，直接执行：

```shell
$ kubectl delete -f nginx-deployment.yaml
```

## 总结

所以，如果你想要快速熟悉 Kubernetes，请按照下面的流程进行练习：

- 首先，在本地通过 Docker 测试代码，制作镜像；
- 然后，选择合适的 Kubernetes API 对象，编写对应 YAML 文件（比如，Pod，Deployment）；
- 最后，在 Kubernetes 上部署这个 YAML 文件。

以后计量少直接使用镜像，直接使用yaml文档。
