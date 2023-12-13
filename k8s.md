# Docker #
#### $ docker build " 我的镜像 "
#### $ docker run " 我的镜像 "
## Docker 项目在短时间内迅速崛起的三个重要原因 ##
如何让开发者把应用部署在我的项目上。
1. Docker 镜像通过技术手段解决了 PaaS 的根本性问题：打包应用改成打包镜像
2. Docker 容器同开发者之间有着与生俱来的密切关系；
3. PaaS 概念已经深入人心的完美契机。
## 容器 ##
容器技术的兴起源于 PaaS 技术的普及；
Docker 公司发布的 Docker 项目具有里程碑式的意义；
Docker 项目通过“容器镜像”，解决了应用打包这个根本性难题。
容器其实是一种沙盒技术。顾名思义，沙盒就是能够像一个集装箱一样，把你的应用“装”起来的技术。这
样，应用与应用之间，就因为有了边界而不至于相互干扰；而被装进集装箱的应用，也可以
被方便地搬来搬去，这不就是 PaaS 最理想的状态嘛。

##### 容器技术的核心功能，就是通过约束和修改进程的动态表现，从而为其创造出一个“边界”。
Docker 容器这个听起来玄而又玄的概念，实际上是在创建容器进程时，指定了这个进程所需要启用的一组 Namespace 参数。这样，容器就只能“看”到当前 Namespace
所限定的资源、文件、设备、状态，或者配置。而对于宿主机以及其他不相关的程序，它就完全看不到了
#### 所以 容器，其实是一种特殊的进程而已。

容器技术的核心功能，就是通过约束和修改进程的动态表现，从而为其创造出一个“边界”。

“敏捷”和“高性能”是容器相较于虚拟机最大的优势，也是它能够在 PaaS 这种更细粒度的资源管理平台上大行其道的重要原因。

Cgroups 技术是用来制造约束的主要手段
#### Namespace 技术则是用来修改进程视图的主要方法。（比如：被隔离应用的进程空间做了手脚，使得这些进程只能看到重新计算过
的进程编号，比如 PID=1。可实际上，他们在宿主机的操作系统里，还是原来的第 100 号进程。）

#### Linux Cgroups 的全称是 Linux Control Group。它最主要的作用，就是限制一个进程组能够使用的资源上限，包括 CPU、内存、磁盘、网络带宽等等。

这是因为容器本身的设计，就是希望####容器和应用能够同生命周期

一个正在运行的 Docker 容器，其实就是一个启用了多个 Linux Namespace 的应用进程，而这个进程能够使用的资源量，则受 Cgroups 配置的限制，容器是一个“单进程”模型。

Linux Cgroups 的设计还是比较易用的，简单粗暴地理解呢，它就是一个子系统目录加上一组资源限制文件的组合。

容器本身的设计，就是希望容器和应用能够同生命周期

#### 这个挂载在容器根目录上、用来为容器进程提供隔离后执行环境的文件系统，就是所谓的“容器镜像”。它还有一个更为专业的名字，叫作：rootfs（根文件系统）。
所以，一个最常见的 rootfs，或者说容器镜像，会包括如下所示的一些目录和文件，比如
/bin，/etc，/proc 等等：
 复制代码
1 $ ls /
2 bin dev etc home lib lib64 mnt opt proc root run sbin sys tmp usr var
而你进入容器之后执行的 /bin/bash，就是 /bin 目录下的可执行文件，与宿主机的
/bin/bash 完全不同。
现在，你应该可以理解，对 Docker 项目来说，它最核心的原理实际上就是为待创建的用户进程：
1. 启用 Linux Namespace 配置；
2. 设置指定的 Cgroups 参数；
3. 切换进程的根目录（Change Root）。

正是由于 rootfs 的存在，容器才有了一个被反复宣传至今的重要特性： 一致性。
由于 rootfs 里打包的不只是应用，而是整个操作系统的文件和目录，也就意味着，应用以及它运行所需要的所有依赖，都被封装在了一起。

需要明确的是，rootfs 只是一个操作系统所包含的文件、配置和目录，并不包括操作系统内核。在 Linux 操作系统中，这两部分是分开存放的，操作系统只有在开机启动时
才会加载指定版本的内核镜像。

容器的 rootfs 由如下图所示的三部分组成：
![image](https://github.com/rita-zu/k8s/assets/153474666/19620014-233b-44c6-93ca-c9f164a95a7b)

## DockerFile ##
Dockerfile 中的每个原语执行后，都会生成一个对应的镜像层
我们后面会统一称 Docker 容器的启动进程为 ENTRYPOINT，而不是 CMD。

一个进程，可以选择加入到某个进程已有的 Namespace 当中，从而达到“进入”这个进程所在容器的目的，这正是 docker exec 的实现原理。
## Volume ##
容器里进程新建的文件，怎么才能让宿主机获取到？

宿主机上的文件和目录，怎么才能让容器里的进程访问到？

### Volume 机制，允许你将宿主机上指定的目录或者文件，挂载到容器里面进行读取和修改操作。

![image](https://github.com/rita-zu/k8s/assets/153474666/f8fcd735-1d15-4f3f-aa1b-c5baece05a84)

这个容器进程“python app.py”，运行在由 Linux Namespace 和 Cgroups 构成的隔离环境里；而它运行所需要的各种文件，比如 python，app.py，以及整个操作系统文件，则由多个联合挂载在一起的 rootfs 层提供。

这些 rootfs 层的最下层，是来自 Docker 镜像的只读层。

在只读层之上，是 Docker 自己添加的 Init 层，用来存放被临时修改过的 /etc/hosts 等文件。

### 而 rootfs 的最上层是一个可读写层，它以 Copy-on-Write 的方式存放任何对只读层的修改，容器声明的 Volume 的挂载点，也出现在这一层。

### 一个“容器”，实际上是一个由 Linux Namespace、Linux Cgroups 和 rootfs 三种技术构建出来的进程的隔离环境。
1. 一组联合挂载在 /var/lib/docker/aufs/mnt 上的 rootfs，这一部分我们称为“容器镜像”（Container Image），是容器的静态视图；

2. 一个由 Namespace+Cgroups 构成的隔离环境，这一部分我们称为“容器运行时”（Container Runtime），是容器的动态视图。
   
# Kubernetes #
![image](https://github.com/rita-zu/k8s/assets/153474666/c66b37f3-3c39-4c59-aebf-bf6ecb9b3338)

Kubernetes 项目的架构，跟它的原型项目 Borg 非常类似，都由 Master
和 Node 两种节点组成，而这两种角色分别对应着控制节点和计算节点。
其中，控制节点，即 Master 节点，由三个紧密协作的独立组件组合而成，它们分别是负责
API 服务的 kube-apiserver、负责调度的 kube-scheduler，以及负责容器编排的 kubecontroller-manager。整个集群的持久化数据，则由 kube-apiserver 处理后保存在 Etcd中。
而计算节点上最核心的部分，则是一个叫作 kubelet 的组件。

在 Kubernetes 项目中，kubelet 主要负责同容器运行时（比如 Docker 项目）打交道。而这个交互所依赖的，是一个称作 CRI（Container Runtime Interface）的远程调用接
口，这个接口定义了容器运行时的各项核心操作，比如：启动一个容器需要的所有参数

Kubernetes 项目并不关心你部署的是什么容器运行时、使用的什么技术实现，只要你的的这个容器运行时能够运行标准的容器镜像，它就可以通过实现 CRI 接入到Kubernetes 项目当中。
而具体的容器运行时，比如 Docker 项目，则一般通过 OCI 这个容器运行时规范同底层的Linux 操作系统进行交互，即：把 CRI 请求翻译成对 Linux 操作系统的调用（操作 Linux
Namespace 和 Cgroups 等）。
此外，kubelet 还通过 gRPC 协议同一个叫作 Device Plugin 的插件进行交互。这个插件，是 Kubernetes 项目用来管理 GPU 等宿主机物理设备的主要组件，也是基于
Kubernetes 项目进行机器学习训练、高性能作业支持等工作必须关注的功能。
而kubelet 的另一个重要功能，则是调用网络插件和存储插件为容器配置网络和持久化存储。这两个插件与 kubelet 进行交互的接口，分别是 CNI（Container Networking
Interface）和 CSI（Container Storage Interface）。

#### Kubernetes 项目最主要的设计思想是，从更宏观的角度，以统一的方式来定义任务之间的各种关系，并且为将来支持更多种类的关系留有余地。

Kubernetes 项目的做法是给 Pod 绑定一个 Service 服务，而 Service 服务声明的IP 地址等信息是“终生不变”的。这个Service 服务的主要作用，就是作为 Pod 的代理入口（Portal），从而代替 Pod 对外暴露一个固定的网络地址。

![image](https://github.com/rita-zu/k8s/assets/153474666/b2f23c63-d180-45fc-9c7f-a0e2ee4ffd3c)

按照这幅图的线索，我们从容器这个最基础的概念出发，首先遇到了容器间“紧密协作”关系的难题，于是就扩展到了 Pod；有了 Pod 之后，我们希望能一次启动多个应用的实例，
这样就需要 Deployment 这个 Pod 的多实例管理器；而有了这样一组相同的 Pod 后，我们又需要通过一个固定的 IP 地址和端口以负载均衡的方式访问它，于是就有了 Service。

可是，如果现在两个不同 Pod 之间不仅有“访问关系”，还要求在发起时加上授权信息。最典型的例子就是 Web 应用对数据库访问时需要 Credential（数据库的用户名和密码）
信息。那么，在 Kubernetes 中这样的关系又如何处理呢？
Kubernetes 项目提供了一种叫作 Secret 的对象，它其实是一个保存在 Etcd 里的键值对数据。这样，你把 Credential 信息以 Secret 的方式存在 Etcd 里，Kubernetes 就会在你指
定的 Pod（比如，Web 应用的 Pod）启动时，自动把 Secret 里的数据以 Volume 的方式挂载到容器里。这样，这个 Web 应用就可以访问数据库了。
除了应用与应用之间的关系外，应用运行的形态是影响“如何容器化这个应用”的第二个重要因素。为此，Kubernetes 定义了新的、基于 Pod 改进后的对象。比如 Job，用来描述一次性运
行的 Pod（比如，大数据任务）；再比如 DaemonSet，用来描述每个宿主机上必须且只能运行一个副本的守护进程服务；又比如 CronJob，则用于描述定时任务等等。

相比之下，在 Kubernetes 项目中，我们所推崇的使用方法是：首先，通过一个“编排对象”，比如 Pod、Job、CronJob 等，来描述你试图管理的应用；
然后，再为它定义一些“服务对象”，比如 Service、Secret、Horizontal Pod Autoscaler（自动水平扩展器）等。这些对象，会负责具体的平台级功能。
这种使用方法，就是所谓的“声明式 API”。这种 API 对应的“编排对象”和“服务对象”，都是 Kubernetes 项目中的 API 对象（API Object）。

# kubeadm 的工作原理

把 kubelet 直接运行在宿主机上，然后使用容器部署其他的 Kubernetes 组件

所以，你使用 kubeadm 的第一步，是在机器上手动安装 kubeadm、kubelet 和 kubectl这三个二进制文件。当然，kubeadm 的作者已经为各个发行版的 Linux 准备好了安装包，
所以你只需要执行：
1 $ apt-get install kubeadm 就可以了。
接下来，你就可以使用“kubeadm init”部署 Master 节点了。

kubeadm 部署 Kubernetes 集群最关键的两个步骤，kubeadm init 和 kubeadm join。

有一种特殊的容器启动方法叫做“Static Pod”。它允许你把要部署的
Pod 的 YAML 文件放在一个指定的目录里。这样，当这台机器上的 kubelet 启动时，它会自动检查这个目录，加载所有的 Pod YAML 文件，然后在这台机器上启动它们。
1 $ ls /etc/kubernetes/manifests/
2 etcd.yaml kube-apiserver.yaml kube-controller-manager.yaml kube-scheduler.yaml

而一旦这些 YAML 文件出现在被 kubelet 监视的 /etc/kubernetes/manifests 目录下，kubelet 就会自动创建这些 YAML 文件中定义的 Pod，即 Master 组件的容器。
