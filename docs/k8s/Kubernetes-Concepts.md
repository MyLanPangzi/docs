# K8S概念

[TOC]

## 大纲

### 组件

#### Master组件

1. api server：Kubernetes API的实现
2. etcd ：存储集群数据
3. scheduler：负责Pods的调度，运行在哪些节点上
4. controler manager：控制器集合
   1. Node Controller：当节点下线时，负责通知以及响应。
   2. Replication Controller：维护Pods的正确数量
   3. Endpoints Controller：填充终端对象
   4. Service Account & Token Controller：负责新命名空间的账号创建以及API访问Token

#### Node组件

1. kubelet：运行在每个节点上，负责容器的创建以及健康检查。
2. kubeproxy：网络代理，负责Pods与内部或外部的通信。
3. 容器运行时：运行容器的软件

#### 插件

1. DNS
2. Dashboard
3. 监控
4. 集群日志

### API

### K8S对象

**是一些持久化实体对象，用来描述集群状态。**

1. 哪些程序运行在哪些节点上。
2. 这些程序可以使用哪些资源。
3. 以及这些程序的运行策略，例如，重启，升级，容错等。

**一个K8S对象是一个“意图记录”，描述了集群的期望状态。**一旦创建就会永久运行。

**每个对象包括2个内嵌对象，管理着对象的配置。**

1. spec：用户提供，描述了对象的期望状态。
2. status：K8S提供，描述了对象的实际状态。

**K8S永远管理着这些对象的实际状态，以匹配用户的期望状态。**

描述一个K8S对象：

```yml
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
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

#### 必须字段

1. apiVersion：使用哪个版本的API。
2. kind：对象类型。
3. matadata：对象元数据，用于标志对象的唯一性。
4. spec：描述对象的期望状态。

### K8S对象管理

一个Kubernetes对象应只用一种管理技术，混合使用会出现不可预料的结果。

#### 编程式命令

```shell
kubectl run nginx --image=nginx #会在未来移除kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
kubectl create deployment nginx --image=nginx
```

**对比对象配置的优势：**

1. 简单，易于学习，易于记忆。
2. 只需一个步骤就能对集群做出改变。

**对比对象配置的劣势：**

1. 不能与审查过程集成。
2. 不能提供审计轨迹跟踪。
3. 不能提供记录源。
4. 不能提供对象创建模板。

#### 编程式对象配置

注意：不适用独立于配置文件更新的资源对象，也就是不能用配置文件进行静态更新的资源。

例如：负载均衡器。

```shell
kubectl create -f nginx.yml
kubectl delete -f nginx.yml -f redis.yml
kubectl replace -f nginx.yml
```

**对比编程式命令的优势：**

1. 对象配置能使用版本控制保存。
2. 能集成review以及audit过程。
3. 提供了对象模板。

**对比编程式命令的劣势：**

1. 需要理解基础的对象模式。
2. 需要编写额外的yml文件。

**对比声明式对象配置的优势：**

1. 简单，易于理解。
2. 表现的更自然。

**对比声明式对象配置的劣势：**

1. 只能在文件上工作，不能在目录上工作。
2. 对象的更新只能反映的配置文件中，下次替换会丢失。

#### 声明式对象配置

```shell
kubectl diff -f .
kubectl apply -f .
kubectl diff -R -f .
kubectl apply -R -f .
```

**对比对象配置的优势：**

1. 保留变更历史，即使文件未合并。
2. 更好的支持目录，以及自动检测操作类型。

**对比对象配置的劣势：**

1. 难以调试以及理解。（非常难。。。。）
2. 部分更新会变得复杂。

### 名称

1. 名称：每个命名空间中的每种资源下每个对象的名字唯一。由客户命名。namespace/kind/name
2. UID：集群中每个对象拥有唯一的UID。由系统生成。用于区分同一对象的历史纪录。

### 命名空间

**同一物理集群下的不同虚拟集群称之为命名空间。**

#### 何时使用

多用户跨多个团队或者项目时使用，几个或者几十个的不要考虑。

命名空间提供了名称作用域，命名空间不能嵌套。

在多用户下，命名空间用于划分集群资源。

未来版本中，命名空间会统一对象的访问控制策略。

资源的名称在命名空间中需要唯一，每个资源只能在一个命名空间下，区分资源优先考虑标签。

#### 使用命名空间

```shell
kubectl get namespaces
kubectl create namespace my-namespace
kubectl run nginx --namespace=my-namespace --image=nginx
kubectl create deployment nginx --image nginx --namespace=my-namespace
kubectl get deployment -n my-namespace
kubectl get pods -n my-namespace
kubectl delete namespace my-namespace

kubectl create namespace hiscat
kubectl config set-context --current --namespace=hiscat
kubectl config view --minify

```

#### 默认命名空间

1. default：所有没有命名空间的对象都在这里。
2. kube-system：kubernetes系统创建的对象都在这里。
3. kube-pulibc：整个集群中可见的公共资源都放这里，只是一个通俗的约定并不是强制性。

#### 命名空间与DNS

创建服务时会默认创建一个DNS入口。DNS：服务名.命名空间.svc.cluster.local

意味着，如果容器使用服务名，将会解析到本地命名空间下的服务。

如果要跨多命名空间使用相同的配置则需要全限定域名（FQDN）。

#### 并不是所有的对象都有命名空间

大多数kubernetes对象都有命名空间，例如service，pod，deployment，replication，controller等。

但是命名空间本身不属于命名空间，以及节点，持久化卷都没有命名空间。

```shell
kubectl api-resources --namespaced=true
kubectl api-resources --namesapced=false
```



### 标签以及选择器

#### 标签是什么

metadata.labels节点下的键值对，用于指定识别属性，有意义且与用户相关，并不意味着核心系统的语义。

用于组织以及选择一组子对象。可以在对象创建时附加也可以后续修改。每个key必须唯一。

标签允许高效的查询和监听，最适合UI以及命令行使用。非识别信息应使用注释。

#### 动机

1. 以松散耦合的方式映射自己特有的组织结构到对象上，而不需要客户端存储这些映射。
2. 跨多维度实体操作。

#### 语法

**prefix/label**：

prefix:可选前缀，如果有必须是DNS子域名，总长不能超过253个字符，用点拆分子域名。

label:  63及位以内数字字母开头或结尾，中间可用数字字母，下划线，横杠，点

如果省略前缀，则默认key为用户私有。

自动化系统组件或第三方插件必须指定前缀。

kubernetes.io/或者k8s.io保留给K8S核心组件。

**value:**

63个字符及以内，数字字母开头或结尾，中间可用-_.

#### 选择器

通过标签选择器可以识别一组对象。标签选择器是kubernetes的核心分组原语。

目前支持2种选择器：等价性选择器，集合选择器。

一个选择器可以由多个条件组成，用逗号分隔。每个逗号相当于逻辑与运算符。

空的选择器或未指定的选择器取决于上下文，使用选择器的API类型，应当记录他们的有效性以及含义。

某些选择器，选择的资源不能重叠，例如ReplicaSet。

#### 相等选择器

=，==，!=

```shell
kubectl get pods -l app=nginx
kubectl get pods -l app!=nginx
```

#### 集合选择器

in，notin，exists

```shell
kubectl get pods -l 'app in (nginx,redis)'
kubectl get pods -l 'app notin (redis)'
kubectl get pods -l 'app'
```

#### 在API对象中设置引用

Service以及ReplicationController可以在selector自动引用其他资源，但只支持等价性选择器。

```yml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  type: LoadBalancer
  ports:
    - port: 80
      targetPort: 80
```

**Job，Deployment，Replica Set，Daemon支持集合选择器。**

**支持等价性选择器。** 

  selector:
    matchLabels:
      enviroment: development

**支持集合选择器，操作符有In，NotIn，Exists，DoesNotExist**

    matchExpressions:
      - key: tier
        operator: In
        values:
          - frontend
```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      enviroment: development
    matchExpressions:
      - key: tier
        operator: In
        values:
          - frontend
  template:
    metadata:
      labels:
        enviroment: development
        tier: frontend
    spec:
      containers:
        - name: nginx-dev
          image: nginx

```

**节点选择器**

### 注释

#### 是什么

**也是对象的元数据，用于附加非识别性数据。**客户端以及工具可以检索此数据。

#### 何时使用

1. 配置默认值。
2. 构建信息，版本号，时间戳，分支号，镜像哈希，镜像注册地址等。
3. 日志，监控，分析或审计的仓库地址。
4. 用于工具或客户端的调试信息。
5. 引用信息（引用地址），引用了其他生态系统的组件等。
6. 轻量级滚动升级工具的元数据。
7. 负责人的手机号，座机号，或者团队的网站等信息。
8. 使用了非标准特性或修改了用户行为的指令。

#### 语法

1. 键值对
2. 键：prefix/name。跟标签一样的规则。
3. 值：value。没有限定。

### 字段选择器

#### 是什么

**通过资源的字段名进行过滤。**

**所有的资源都支持metadata.name以及metadata.namespace字段选择器。**

**如果使用不支持的字段选择器会报错。**

```shell
kubectl get pods --all-namspaces --field-selector status.phase=Running
kubectl get pods --field-selector ''
```

**支持的操作符：=，!=，==。**

**可以链式选择，用逗号分隔。**

**可以选择多资源类型。**

```shell
kubectl get pods --all-namespace --field-selector status.phase=Running,spec.restartPolicy=Always
kubectl get statefulsets,svc --all-namespace --field-selector metadata.namespace=default

```

### 建议标签

**使用一组常见的标签使得工具具有更好的互操作性，并且易于理解。**

**建议化的标签能更好的查询应用。**

元数据是围绕应用程序的概念进行组织的，K8S不提供也不强制实施应用程序的概念。

应用是非正式且以元数据进行描述的，应用程序内容的定义是松散的。

#### 常用标签

| Key                          | 描述               | 案例          | 类型   |
| ---------------------------- | ------------------ | ------------- | ------ |
| app.kubernetes.io/name       | 应用名称           | mysql         | string |
| app.kubernetes.io/instance   | 唯一的实例名称     | rbac-10.2.2.3 | string |
| app.kubernetes.io/version    | 版本号             | 1.0.0         | string |
| app.kubernetes.io/component  | 架构组件           | database      | string |
| app.kubernetes.io/part-of    | 所属高层应用名     | hiscat        | string |
| app.kubernetes.io/managed-by | 管理应用操作的工具 | helm          | string |

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app.kubernetes.io/name: disvoery
    app.kubernetes.io/instance: discovery-deployment
    app.kubernetes.io/version: 1.0.0
    app.kubernetes.io/component: discovery
    app.kubernetes.io/part-of: hiscat-blog
    
```

#### 应用程序以及应用程序实例

在集群中一个应用能安装多次，某些情况下是在同一个命名空间中。

应用程序名以及实例名是单独记录的。每个实例的实例名必须唯一。

## 集群架构

### 节点

K8S中一个节点是一台工作机，可以是虚拟机也可以是物理机。

每个节点包含着运行Pods的必要服务以及受Master组件管理。

节点服务包括容器运行时，kubelet，kubeproxy。

#### 节点状态

一个节点的状态包含四个信息。

- 地址
- 条件
- 容量以及可分配资源
- 通用信息

```shell
kubectl describe node nodeName
Name:               k8s-server
Roles:              master
Labels:             beta.kubernetes.io/arch=amd64
                    beta.kubernetes.io/os=linux
                    kubernetes.io/arch=amd64
                    kubernetes.io/hostname=k8s-server
                    kubernetes.io/os=linux
                    node-role.kubernetes.io/master=
Annotations:        kubeadm.alpha.kubernetes.io/cri-socket: /var/run/dockershim.sock
                    node.alpha.kubernetes.io/ttl: 0
                    projectcalico.org/IPv4Address: 192.168.2.10/24
                    projectcalico.org/IPv4IPIPTunnelAddr: 192.168.185.128
                    volumes.kubernetes.io/controller-managed-attach-detach: true
CreationTimestamp:  Mon, 06 Jan 2020 07:29:42 +0800
Taints:             node-role.kubernetes.io/master:NoSchedule
Unschedulable:      false
Lease:
  HolderIdentity:  k8s-server
  AcquireTime:     <unset>
  RenewTime:       Thu, 09 Jan 2020 06:31:02 +0800
Conditions: #条件信息
  Type                 Status  LastHeartbeatTime                 LastTransitionTime                Reason                       Message
  ----                 ------  -----------------                 ------------------                ------                       -------
  NetworkUnavailable   False   Mon, 06 Jan 2020 07:51:44 +0800   Mon, 06 Jan 2020 07:51:44 +0800   CalicoIsUp                   Calico is running on this node
  MemoryPressure       False   Thu, 09 Jan 2020 06:27:34 +0800   Mon, 06 Jan 2020 07:29:38 +0800   KubeletHasSufficientMemory   kubelet has sufficient memory available
  DiskPressure         False   Thu, 09 Jan 2020 06:27:34 +0800   Mon, 06 Jan 2020 07:29:38 +0800   KubeletHasNoDiskPressure     kubelet has no disk pressure
  PIDPressure          False   Thu, 09 Jan 2020 06:27:34 +0800   Mon, 06 Jan 2020 07:29:38 +0800   KubeletHasSufficientPID      kubelet has sufficient PID available
  Ready                True    Thu, 09 Jan 2020 06:27:34 +0800   Mon, 06 Jan 2020 07:51:27 +0800   KubeletReady                 kubelet is posting ready status. AppArmor enabled
Addresses: #地址信息
  InternalIP:  192.168.2.10
  Hostname:    k8s-server
Capacity:#容量信息
  cpu:                2
  ephemeral-storage:  19540624Ki
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             2017840Ki
  pods:               110
Allocatable:#可分配信息
  cpu:                2
  ephemeral-storage:  18008639049
  hugepages-1Gi:      0
  hugepages-2Mi:      0
  memory:             1915440Ki
  pods:               110
System Info:#通用信息
  Machine ID:                 a7844a3135874a6bb66b0207bfabc345
  System UUID:                08914D56-5EF1-70CB-B6F2-5AFD65696049
  Boot ID:                    7cb1182a-1020-42a4-bd35-6c87288135fd
  Kernel Version:             4.15.0-72-generic
  OS Image:                   Ubuntu 18.04.3 LTS
  Operating System:           linux
  Architecture:               amd64
  Container Runtime Version:  docker://18.9.7
  Kubelet Version:            v1.17.0
  Kube-Proxy Version:         v1.17.0
PodCIDR:                      10.244.0.0/24
PodCIDRs:                     10.244.0.0/24
Non-terminated Pods:          (6 in total)
  Namespace                   Name                                  CPU Requests  CPU Limits  Memory Requests  Memory Limits  AGE
  ---------                   ----                                  ------------  ----------  ---------------  -------------  ---
  kube-system                 calico-node-ds2cz                     250m (12%)    0 (0%)      0 (0%)           0 (0%)         2d22h
  kube-system                 etcd-k8s-server                       0 (0%)        0 (0%)      0 (0%)           0 (0%)         2d23h
  kube-system                 kube-apiserver-k8s-server             250m (12%)    0 (0%)      0 (0%)           0 (0%)         2d23h
  kube-system                 kube-controller-manager-k8s-server    200m (10%)    0 (0%)      0 (0%)           0 (0%)         2d23h
  kube-system                 kube-proxy-hcnkk                      0 (0%)        0 (0%)      0 (0%)           0 (0%)         2d23h
  kube-system                 kube-scheduler-k8s-server             100m (5%)     0 (0%)      0 (0%)           0 (0%)         2d23h
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  Resource           Requests    Limits
  --------           --------    ------
  cpu                800m (40%)  0 (0%)
  memory             0 (0%)      0 (0%)
  ephemeral-storage  0 (0%)      0 (0%)
Events:              <none>
```

##### 地址

- HostName：主机名，由节点内核显示。--hostname-override参数重写。
- ExternalIP：外部IP，由外部路由分配。
- InternalIP：内部IP，由K8S集群分配。

每个字段的使用取决于裸机配置或云厂商。

##### 条件

描述了所有运行节点的状态。

| 节点条件           | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| Ready              | True，表示节点健康且准备接受Pods。Unknow表示节点控制器没有收到此节点最后一个周期的心跳（默认40S）。node-monitor-grace-period |
| MemoryPressure     | True，表示可用内存过低。                                     |
| PIDPressure        | True，表示节点进程过多。                                     |
| DiskPressure       | True，表示节点可用磁盘容量过低。                             |
| NetworkUnavailable | True，表示节点网络配置不正确。                               |

```json
"conditions": [
  {
    "type": "Ready",
    "status": "True",
    "reason": "KubeletReady",
    "message": "kubelet is posting ready status",
    "lastHeartbeatTime": "2019-06-05T18:38:35Z",
    "lastTransitionTime": "2019-06-05T11:41:27Z"
  }
]
```

**如果Ready类型下的status字段是False或者Unknown的状态持续超过pod-eviction-timeout，则此节点上的Pods会被节点控制器删除。默认是五分钟。**

当节点与主节点不能通讯时，Pods仍然可以运行在Node上，直到与Master恢复通讯。

1.5以前的版本Node Controller会删除不可达的Pods。

1.5以后的版本，不会删除，直到确认已经停止。此时Pods状态为Terminating或者Unknown。

当节点永久的离开集群时，K8S无法推断节点不可达，需要管理员手动删除节点对象，此时会删除节点上所有的Pods，以及释放它们的名字。

The node lifecycle controller automatically creates [taints](https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/) that represent conditions. When the scheduler is assigning a Pod to a Node, the scheduler takes the Node’s taints into account, except for any taints that the Pod tolerates.

##### 容量以及可分配资源

描述了节点的可用资源，包括cpu，内存，最大数量Pods。

capacity块描述了节点拥有资源的总数，allocatable块描述了节点可被Pods消费的资源。

##### 通用信息

描述了通用信息，包括内核版本，K8S版本，Docker版本，操作系统名称等。

#### 管理

Node是由外部创建的。metadata.name字段是节点IP。K8S会周期性的通过name字段进行健康检查。直到节点有效或者Node对象删除。

```json
{
  "kind": "Node",
  "apiVersion": "v1",
  "metadata": {
    "name": "10.240.79.157",
    "labels": {
      "name": "my-first-k8s-node"
    }
  }
}
```

##### 节点控制器

Master组件，管理节点的多个方面。

拥有多个角色，在节点的生命周期中。

- 当节点注册时，赋予CIDR块给节点（如果CIDR块赋值开关打开）
- 保持内部节点列表与云提供商的可用机器一致。
- 监控节点健康。负责更新的NodeStatus的NodeReady条件。每--node-monitor-period周期检查节点状态，默认是40S。

###### 心跳

两种心跳：更新NodeStatus的心跳以及Lease Object的心态（租约对象）。

每个节点在kube-node-lease命名空间中关联了一个lease对象。lease是一个轻量级的资源，随着节点的伸缩改善了节点的心跳性能。

kublet负责创建以及更新NodeStatus和Lease对象。

默认五分钟更新一次NodeStatus对象。

默认十秒钟更新一次Lease Object。Lease的更新独立于NodeStatus。

###### 可靠性

从1.4开始，当Node Controller决定驱逐Pods时会检查所有节点的状态。

大多数情况下，驱逐比率是--node-eviction-rate（0.1），每十秒驱逐一个。

当node在一个给定zone变得不健康时，evication行为将会发生改变，node controller会检查坏死率。默认是阈值是--unhealthly-zon-threshold（0.55），如果大于该阈值，则会减少eviction rate。

如果集群数量小于--large-cluster-size-threshold（50），eviction会停止，否则eviction rate会减少至--secondary-node-eviction-rate（0.01）也就是100秒一个。

主要是应对跨zone的集群，增加可用性。当某个zone不可用时，可以将负载转移到健康zone。

当所有node都在一个zone时，--node-eviction-rate是正常的0.1.

极端情况，所有zone都不可用时，node controller会假设master发生了连接性问题，然后停止eviction直到连接恢复。

从1.6开始，当Pods不能tolerate NoExcute taints时，node controller同样也负责evicting 带有NoExcute taints节点的Pods。

从1.8开始负责创建描述节点条件的taints。

#### 节点的自我注册

kubelet的--register-node默认为true，会自动注册到api server。自我注册选项

- --kubeconfig。注册凭证以及服务路径。Path to credentials to authenticate itself to the apiserver.
- --cloud-provider。读取自身元数据，以及与云提供商会话。
- --register-node。自动注册到api server。
- --register-with-taints。注册taints
- --node-ip。节点IP
- --node-lables。节点标签，用于Pods的标签选择器。
- --node-status-update-frequency。制定多久汇报一次节点状态给master。

### 主节点通信

分类Master与集群的通信路径，加强网络配置，以便在不受信任的网络部署集群。

#### Cluster与Master通信

集群中所有的通信路径都终止于api server。api server不对外暴露服务，内部配置监听443端口，启用多种认证与授权方式，特别是匿名请求以及服务账号token被允许时。

集群中的节点应当被提供公共根证书，以便用于安全的连接到api server。

Pods使用服务账号安全的连接到api server。在实例化的时候K8S会自动的注入证书以及token。

kubernetes服务对外暴露了一个虚拟IP用于重定向至HTTPS的终端。

master组件同样也可以通过HTTPS端口与api server通信。

节点与master的通信默认是安全的，能够运行在不安全的网络中。

#### Master与Cluster通信

1. 通过kubelet进行RPC通信。
   1. 用于抓取Pods日志。
   2. 附加到运行的Pods。
   3. 提供端口转发功能。
2. 通过kubeproxy进行通信。

连接终止于kubelet的HTTPS端口。默认的，api server不校验kubelet证书，这使得连接容易遭受中间人攻击，不能安全的运行在不受信任的网络中。

使用--kubelet-certificate-authority标志，并为apiserver提供根证书包来验证证书。

如果不能提供，则可以使用SSH隧道进行安全通信。（但以标记过期。。。）

apiserver到node,pods,service的连接是plain HTTP且未加密，未认证。

目前master与节点的通信，是不安全的，不能对外暴露。

### 控制器

**控制器是一个非终止回路，调节系统的状态与期望状态匹配。**

#### 控制器模式

一个控制器至少跟踪了一种资源类型。对象的spec字段描述了期望状态，控制器负责控制实际状态以匹配期望状态。

控制器通过apiserver来管理状态。

某些控制器需要对集群外部的资源做出改变。例如节点控制器。

#### 期望与现实

集群会随着工作的发生以及控制回路的自动修复而发生改变，这意味着，集群永远不会达到一个稳定状态。

只要控制器能做出有效改变，稳不稳定不重要。

#### 设计

K8S的设计原则，每个控制器管理一个特殊的集群状态。大多数控制器使用一种资源来描述期望状态，并使用另一种资源来做出改变以匹配期望状态。控制器被设计为允许失败。

控制器之间可能创建或更新同一种资源，但只更新自己关心的资源，不会更新到其他控制器的资源。

#### 运行controller的方式

通过kube-contolelr-manager运行，内部会自动管理失败的控制器。

### CCM（云控制器管理器）

目的是为了K8S能独立的演化，不涉及特定云厂商的代码。CCM与Master组件一起运行，能作为插件启动，运行在K8S之上。

CCM基于插件机制进行设计，运行新的云提供商以插件的方式轻松集成K8S。

未使用CCM的架构

![](../images\pre-ccm-arch.png)

#### 设计

未使用CCM的架构通过：

1. kubelet
2. apiserver
3. contoller manager

与云提供商进行集成。

使用CCM的架构：

![](../images\post-ccm-arch.png)

#### CCM组件

KCM云依赖组件：

- Node Controller
- Volume Controller
- Route Contoller
- Service Controller

CCM运行的控制器：

- Node Controller
- Route Controller
- Service Controller

Volume Controller打算使用CSI（容器存储接口）替代。

#### CCM功能

CCM继承了KCM的功能

**节点控制器**：

**从云提供商获取节点信息，用于节点的初始化。**

1. 用云特定的Zone/Region 标签初始化节点。
2. 使用特定于云实例的详细信息初始化节点，例如类型，大小。
3. 获取IP以及主机名。
4. 当节点无法响应时，检查cloud是否删除此实例，如果删除，则删除K8S Node对象。

**路由控制器：**

负责节点路由，跨节点容器通信，目前只有谷歌支持。。。

**服务控制器：**

监听服务的CRUD事件。基于当前状态，配置LB反应服务状态。确保LB是最新的。

**kubelet**

初始化无关于cloud provider的节点信息，并给节点打上污点标记，直到CCM初始化完特定云信息。

#### 插件机制

CCM使用Go接口，云厂商可自定义实现。

#### 授权

**Node Controller**

只操作节点对象， It requires full access to get, list, create, update, patch, watch, and delete Node objects.

**Route Controller**

监听节点的创建，配置合适的路由。 It requires get access to Node objects.

**Service Controller**

监听Service Object的创建，更新，删除事件，然后配置合适的终端对象。

v1/Service:

- List
- Get
- Watch
- Patch
- Update

**Other**

CCM核心实现要求能够创建事件，以及服务账号的创建。

v1/Event:

- Create
- Patch
- Update

v1/ServiceAccount:

- Create

#### 厂商实现

- AWS
- Azure
- OpenStack

#### 集群管理

## 容器

#### 镜像

跟docker的语法一样。

##### 更新镜像

镜像默认更新策略是IfNotPresent，如果镜像存在则不会pull新镜像。

- 设置imagePullPolicy为Always则永远pull镜像
- 如果使用latest标签，则永远pull image
- 省略imagePullPolicy以及tag，则永远pull image
- 启用AlwaysPullImage 管理控制器。

镜像应当避免使用:latest标签。 

##### 使用清单构建多架构镜像

Docker CLI现在支持docker manifest构建镜像。可以打包多个镜像到一个镜像。

https://docs.docker.com/engine/reference/commandline/manifest/

```shell
docker manifest inspect imageName
docker manifest create 
docker manifest push
docker manifest annotate
docker manifest create 45.55.81.106:5000/coolapp:v1 \
    45.55.81.106:5000/coolapp-ppc64le-linux:v1 \
    45.55.81.106:5000/coolapp-arm-linux:v1 \
    45.55.81.106:5000/coolapp-amd64-linux:v1 \
    45.55.81.106:5000/coolapp-amd64-windows:v1
Created manifest list 45.55.81.106:5000/coolapp:v1
```

这些命令只能在命令行使用。

##### 使用私有注册表

私有注册表可能需要密钥，在pull 镜像的时候。

- Configuring Nodes to Authenticate to a Private Registry
  - all pods can read any configured private registries
  - requires node configuration by cluster administrator
- Pre-pulled Images
  - all pods can use any images cached on a node
  - requires root access to all nodes to setup
- Specifying ImagePullSecrets on a Pod
  - only pods which provide own keys can access the private registry

Docker存储私有化注册表在$HOME/.dockercfg或者$HOME/.docker/config.json文件中。

kubelet拉取镜像时会读取这些配置用作凭证。

- `{--root-dir:-/var/lib/kubelet}/config.json`
- `{cwd of kubelet}/config.json`
- `${HOME}/.docker/config.json`
- `/.docker/config.json`
- `{--root-dir:-/var/lib/kubelet}/.dockercfg`
- `{cwd of kubelet}/.dockercfg`
- `${HOME}/.dockercfg`
- `/.dockercfg`

配置私有化注册表建议步骤：

1. ```shell
   docker login [server]#在你想要使用凭证的节点上登录
   #这会更新$HOME/.docker/config.json
   ```

2. 查看$HOME/.docker/config.json，确保已经认证。

3. 获取节点列表

   1. 获取节点名字 

      ```shell
      nodes=$(kubectl get nodes -o jsonpath='{range.items[*].metadata}{.name} {end}')
      ```

   2. 获取节点IP

      ```shell
      nodes=$(kubectl get nodes -o jsonpath='{range .items[*].status.addresses[?(@.type=="ExternalIP")]}{.address} {end}')
      ```

4. 拷贝config.json文件到上述路径中

   1. ```
      for n in $nodes; do scp ~/.docker/config.json root@$n:/var/lib/kubelet/config.json; done
      ```

5. 验证

   ```shell
   kubectl apply -f - <<EOF
   apiVersion: v1
   kind: Pod
   metadata:
     name: private-image-test-1
   spec:
     containers:
       - name: uses-private-image
         image: $PRIVATE_IMAGE_NAME
         imagePullPolicy: Always
         command: [ "echo", "SUCCESS" ]
   EOF
   #pod/private-image-test-1 created
   kubectl logs private-image-test-1
   kubectl describe pods/private-image-test-1 | grep "Failed"
   
   ```

   **确保每个节点都有同样的配置。**

可以预拉取镜像，这样可以绕过认证，必须设置imagePullPolicy为IfNotPresent或者Never。

确保所有节点拉取了镜像。（K8S集群搭建的时候可以使用阿里的镜像或者自己拉取镜像）。

**创建密钥与Docker Config文件。**只能在没有证书的时候创建，有证书的时候，参考：https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/#registry-secret-existing-credentials

```shell
kubectl create secret docker-registry <name> --docker-server=DOCKER_REGISTRY_SERVER --docker-username=DOCKER_USER --docker-password=DOCKER_PASSWORD --docker-email=DOCKER_EMAIL

```

**在Pods中引用密钥**

```shell 
cat <<EOF > pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: foo
  namespace: awesomeapps
spec:
  containers:
    - name: foo
      image: janedoe/awesomeapp:v1
  imagePullSecrets:
    - name: myregistrykey
EOF

cat <<EOF >> ./kustomization.yaml
resources:
- pod.yaml
EOF
```

**用例：**

1. 只运行在非专有镜像中，不需要隐藏镜像。无需配置。
2. 某些镜像需要隐藏时，但对所有集群用户可见，可以搭建私有镜像仓库，在每个节点上配置密钥。
3. 集群运行在专有镜像上，且有访问控制。开启镜像拉取控制器，控制敏感数据。
4. 运行在多租户环境中，每个租户需要一个私有镜像。开启镜像拉取控制器，运行私有仓库，每个租户生成不同的凭证，并为每个租户的命名空间设置密钥，租户添加密钥至命名空间的imagePullSecrets。
5. 如果需要访问多仓库，则每个仓库创建一个密钥。kubelet会合并这些密钥至.docker/config.json中。

#### 容器环境变量

K8S容器环境提供了几个重要的资源：

- 一个文件系统，这是一个镜像与一个或多个卷的组合。
- 关于容器的信息。
- 集群中其他节点的信息。

##### 容器信息

容器的hostname就是Pod的名字。

Pods名以及namespace作为环境变量传递至容器。

用户自定义的环境变量同样可用。

##### 集群信息

当容器创建时，集群中所有正在运行的服务作为环境变量传递至容器。环境变量匹配docker 的--link语法。

容器内可通过dns访问外部服务，如果DNS插件启用。

#### Runtime Class

**Runtime class**

RuntimeClass用于选择容器运行时，是一种配置。

**动机**

不同的Pods可以运行不同的RuntimeClass以提供性能以及安全的平衡。

不同的Pods可以运行相同的RuntimeClass但可以有不同的配置。

**会有额外的硬件开销。但保证了安全性隔离**

**设置**

1. 在节点上配置CRI实现。

2. 创建相应的RuntimeClass资源。

   ```yml
   apiVersion: node.k8s.io/v1beta1  # RuntimeClass is defined in the node.k8s.io API group
   kind: RuntimeClass
   metadata:
     name: myclass  # The name the RuntimeClass will be referenced by
     # RuntimeClass is a non-namespaced resource
   handler: myconfiguration  # The name of the corresponding CRI configuration
   ---
   apiVersion: v1
   kind: Pod
   metadata:
     name: mypod
   spec:
     runtimeClassName: myclass
     # ...
   ```

   

#### 容器生命周期钩子

尽可能的保持钩子轻量级。钩子函数可能会调用多次，确保可重复执行。

用于预设置资源，以及清除资源。比如密钥，token等。

容器初始化前后有2个钩子函数：

1. PostStart：无参，在容器创建后执行，同步阻塞，无法达到Running状态，超时会杀死容器。
2. PreStop：无参，在容器销毁前执行，同步阻塞，无法到达Complete状态，超过宽限期会强制杀死容器。

钩子实现：

1. Exec：执行指定的命令，资源消耗统计在容器中。
2. HTTP：执行HTTP请求。

## 负载

### Pods

#### 大纲

##### 理解Pods

K8S中，一个Pod是最基础的执行单元，最小最简单的可部署单元。

Pod即是集群中的进程。封装了应用的容器，存储，网络，以及运行选项，也是部署单元，应用程序实例。Pod由一个或多个紧耦合的容器组成，容器间共享资源。

Pods的2种表现方式:

-  **单容器Pod**
- **多容器Pod**.封了多个紧耦合需共享资源的容器。![example pod diagram](https://d33wubrfki0l68.cloudfront.net/aecab1f649bc640ebef1f05581bfcc91a48038c4/728d6/images/docs/pod.svg)
- 参考：
  - [The Distributed System Toolkit: Patterns for Composite Containers](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns)
  - [Container Design Patterns](https://kubernetes.io/blog/2016/06/container-design-patterns)

每个Pod对应一个应用实例，如需伸缩，可部署多个Pod。通常使用Controller进行伸缩。

**Pods如何管理多个容器**：

Pods被设计为协作多个进程组成一个紧密合作的服务单元。Pod中的容器自动的放置在一起并在调度在同一物理机或虚拟机上。这些容器共享资源以及依赖，可以与其他容器通讯以及协调他们何时以及如何终止。

只有紧耦合共享资源的Pods才考虑放置在同一个Pod中，例如上图。

某些Pods有初始容器以及应用容器，初始容器在应用容器启动之前运行并且完全成功退出。

Pods提供了2种共享的资源，网络以及存储。

**网络**

每个Pod拥有一个唯一的IP，Pod中的每个容器共享同一网络命名空间包括IP地址以及端口。同一Pod内的容器通讯通过localhost。Pod之间的通讯通过CNI插件进行协调，例如calico，flannel。

**存储**

一个Pod能指定一组共享的卷。Pod中的容器通过卷共享数据。卷能持久化数据，在容器重启后依然可以访问。

##### 使用Pods

极少手动创建单独的Pod，因为Pod被设计为一次性实体，用完即删。一旦创建就会被分配至一个节点，直至进程终止，pod对象删除，Pod被驱逐，节点失败。

> **Note:** 不要混淆重启容器与重启Pod。Pod不能运行，只是容器运行的环境。Pod的重启将被分配到其他节点，Pod内容器共享的卷将会被删除。

Pod不能自愈。如果调度到一个失败的节点或调度失败，则Pod会被删除。Pod不能在驱逐中以及节点维护时存活。K8S使用了高层抽象（控制器）来处理Pod的自愈以及伸缩。

**控制器能创建以及管理多个Pod，拥有复制，滚动更新，自愈等能力。**

常见控制器:

- [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
- [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/)
- [DaemonSet](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

控制器使用模板来创建Pod。

##### Pods模板

Pod模板是规范，包括在其他对象中，例如RC，JOB，DS。控制器使用模板来构建真实的Pod。模板如下

```yml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox
    command: ['sh', '-c', 'echo Hello Kubernetes! && sleep 3600']
```

Pod模板不描述期望状态，不影响已创建的Pod，能方便的更新。

Pod拥有状态，指定了所有容器的期望状态。

#### Pods

 Pod是最小可部署的计算单元，能被K8S创建以及管理。

##### 是什么

Pod是一组容器，他们共享存储，网络，规范，他们放置在同一位置，同时调度，共享同一上下文。

Pod构建了一个逻辑主机，这个逻辑主机放置了一组紧耦合的容器。

Pod的共享上下文是一Linux命名空间，cgroups，可能是与Docker容器类似的隔离。

在Pod的上下文中，应用可能有更多的子隔离。

Pod中的容器共享同一IP以及端口范围，通过localhost找到对方。Pod间可以通过IPC（中间进程通讯）通讯。不同Pod之间的容器有不同的IP，不能直接通过IPC通讯，需通过CNI插件。

Pod中的容器共享卷。

Pod被认为是相对临时的实体。一旦创建（赋予一个UID）会被调度到节点上，直至终止或删除。如果节点死亡，Pod被调度为删除，超时后，会创建新的Pod调度到新的节点（不同的UID）。

与Pod同生共死的东西，将会在Pod删除时被删除。重新分配的Pod不会再拥有相同的内容。

##### 动机

Management

Pod是一个模型，协调多进程组合成一个紧密的服务单元。Pod简化了应用的部署以及管理。Pod是部署，水平伸缩，复制的单元。Pod中容器的调度，命运（同生共死），协调复制，资源共享，以及依赖管理是自动处理的。

Pod内的应用共享IP以及端口（注意端口的分配），通过localhost与彼此通讯。在共享的扁平网络空间中，每个Pod拥有一个IP，可以跨网络进行通讯。Pod名即是容器主机名。除了共享网络，Pod中的应用还共享卷。

##### 用例

Pod可以用于托管垂直集成的应用程序栈，但主要动机是支持协同放置，协同管理的帮助程序。:

-  内容管理系统，文件，数据加载器，本地缓存管理器。
- 日志，检查点备份，压缩，自旋，快照等。
- 数据监听器，日志收集器，日志以及监控适配器，事件发布器等。
- 代理，桥接，适配器。
- 控制器，管理器，配置器，更新器等。

单独个Pod不打算用于运行多个相同的应用程序实例。

For a longer explanation, see [The Distributed System ToolKit: Patterns for Composite Containers](https://kubernetes.io/blog/2015/06/the-distributed-system-toolkit-patterns).

##### 替换考虑

为什么不要在单个容器中运行多个应用？：

1. 透明性。使得底层基础设施可以为容器提供服务，为用户提供便利。

2. 解耦软件依赖。单容器可以独立的被版本化，重建以及重部署。

3. 易于管理

4. 高效。基础设施承担了大量的工作，容器可以变得轻量级。

*Why not support affinity-based co-scheduling of containers?*

That approach would provide co-location, but would not provide most of the benefits of Pods, such as resource sharing, IPC, guaranteed fate sharing, and simplified management.

##### 持久性

Pods不打算用作持久化实体，不会再调度失败，节点失败，驱逐或节点维护中幸存。

用户不应当直接创建Pods。应使用高层抽象（控制器）来创建Pod。控制器提供了集群范围的自愈，复制，滚动更新。

Pod 被暴露为原语，以便于:

- 调度器以及控制器的可插拔性。
- 支持Pod级别的操作，无需控制器API代理。
- 从控制器的生命期中解耦。
- 解耦控制器以及服务。终端控制器只需观察Pods。
- kubelet级别功能与集群级别功能的清晰组合。kubelet是高效的pod控制器。
- HA应用希望Pod终止时以及删除时提前替换。

##### 终止

优雅的终止Pod是非常重要的（给进程一个clean up的机会）。当客户发起删除请求时，系统会记录预期的宽限期在删除之前，过期后会强制删除。如果kubelet重启，会重新尝试等待宽限期然后再kill。

案例:

1. 用户发送删除命令，默认等待30秒宽限期。
2. API server更新Pod，超出宽限期认为Pod死亡。
3. Pod变成Terminating状态，当查询的时候。
4. 于此同时（步骤3）kubelet看到pod标记为终止状态，pod更新时间已在步骤2设置，开始终止进程。
   1. 如果pods内的容器定义了preStop钩子，则调用钩子。如果钩子执行超过了宽限期，则等待扩展宽限期。
   2. 容器接受到TERM信号，并不是所有容器都在同一时间接受到终止信号，如果停止顺序很重要，则每个容器都需要一个preStop钩子。
5.    于此同时（步骤3）Pod从endpoints移除，不在参与负载均衡，不再是RC控制的一部分。
6. 当宽限期过后，Pod中的所有进程都会被强制KILL。
7. kubelet完成Pod的删除，在API server上设置宽限期为0（立即删除）。客户端不能再看见此Pod。

默认的，所有的删除都有30S宽限期。kubectl delete 命令支持--grace-period选项来改变宽限期。当值设置为0时，强制删除Pod。必须要指定--force与--grace-period=0才能完成强制删除。

**强制删除Pod**

强制删除定义为，Pod立即从集群和etcd中删除。当pod强制删除时，API server不会等待宽限期。它会立即从API中移除。节点上，立即终止的pod仍然有一小段宽限期。

强制删除pod可能存在危险，应当心。特别是在状态集中的pods。

##### 权限模型

pod中的任意容器都能启用权限模型。容器内的进程能访问外部的网络栈和设备。容器内的进程能获得与外部进程一样的权限。所有网络以及卷插件能从kubelet中分离。容器运行时必须支持相关的设置。

##### API对象

#### 生命周期

- [Pod phase](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-phase)
- [Pod conditions](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-conditions)
- [Container probes](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes)
- [Pod and Container status](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-and-container-status)
- [Container States](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-states)
- [Pod readiness gate](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-readiness-gate)
- [Restart policy](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#restart-policy)
- [Pod lifetime](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#pod-lifetime)
- [Examples](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#examples)

##### 阶段

Pods的status字段是PodStatus对象，拥有phase字段。

phase字段描述了Pod在生命周期的哪个阶段。phase既不是容器或Pod状态的汇总，也不是状态机。

phase字段只允许出现下面的值：

| Value       | Description                                                  |
| :---------- | :----------------------------------------------------------- |
| `Pending`   | Pod已被接受，但容器还没创建。可能还在调度或者拉取镜像。      |
| `Running`   | 已绑定到节点，所有容器已创建。至少还有一个容器在运行或者进程正在启动或重启。 |
| `Succeeded` | 所有容器成功终止且不被重启。                                 |
| `Failed`    | 所有容器已被终止，至少有一个容器终止失败。即非0退出状态或被系统终止。 |
| `Unknown`   | 出于某些原因Pod状态不能被获取，典型的由于网络问题不能与Pod主机通讯。 |

##### Pod Conditions

Pod的PodStatus中有一个PodConditions数组，通过Pod传递（也许没有）。PodCondition中的每一个元素有六个可能字段:

- lastProbeTime最后一次探针时间。

- lastTransitionTime最后一次转换时间，由一个状态转换为另一个状态。

- message转换详情，人类可读。

- reason，唯一，一个单词，驼峰命名，条件的最后一次转换原因。

- status字段，可能是True，False，Unknown。

- type字段：

  - PodScheduled：Pod已被调度到一个节点。
  - Ready：有能力服务请求，应当被添加到负载均衡中。
  - Initialized：所有容器启动成功。
  - Unschedulable：调度器现在不能调度此Pod，例如资源泄漏或其他问题。
  - ContainerReady：所有容器就绪。

```
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 

```



##### 探针

探针是对容器周期性的诊断通过kubelet完成。有三种类型的探针实现:

- ExecAction：在容器内执行特定的命令。0退出即成功。
- TCPSocketAction：检查容器的的端口是否打开。
- HTTPGetAction：完成一个HTTP请求。返回码在200-400即成功。

探针结果:

- Success：容器通过诊断。
- Failure：容器未通过诊断。
- Unknown：诊断失败，无动作采取。

探针类型:

- livenessProbe：指示容器是否在运行。如果探测失败，kubelet会杀死容器，然后服从restartPolic策略。如果为提供活性探针，则默认为Sucess。
- readinessProbe：指示容器是否能服务请求。如未就绪，EndpointController会移除Pod的IP不参与LB。初始延迟之前默认状态是Failure。如未提供则默认Sucess。
- startupProbe：指示容器是否启动。所有其他探针都在此探针之后（这是个alpha特性），直到此探针成功。如果失败，kubelet杀死容器，然后服从restartPolicy。如未提供默认Sucess。

**何时使用活性探针。**

**FEATURE STATE:** `Kubernetes v1.0` [stable](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#)

在探针失败时，自定义容器kill或重启逻辑。

检测死锁。

**何时使用就绪探针：**

**FEATURE STATE:** `Kubernetes v1.0` [stable](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#)

启动时需加载大量数据，配置文件，迁移数据，下线维护。

**何时使用启动探针：**

启动时间大于活性探针的失败时间：初始延迟+失败阈值*周期

**容器状态：**

一旦Pod分配至节点，kubelet开始创建容器。容器状态：Waiting，Running，Terminated。

查看命令：kubectl describe pod podname。

`Waiting`: 默认状态。正在拉取镜像或使用密钥等其他操作。

- ```yaml
  ...
    State:          Waiting
     Reason:       ErrImagePull
    ...
  ```

- `Running`: 进入此状态时，postStart钩子已被执行。

  ```yaml
  ...
    State:          Running
     Started:      Wed, 30 Jan 2019 16:46:38 +0530
  ...
  ```

- `Terminated`: 进入此状态时,preStop钩子已执行完。正常或异常终止。

  ```yaml
  ...
    State:          Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Wed, 30 Jan 2019 11:45:26 +0530
      Finished:     Wed, 30 Jan 2019 11:45:26 +0530
  ...
  ```

**重启策略：**

restartPolicy：Always，OnFailure，Never。

**生存期：**

Pod会一直存活至控制器或管理员手动删除它。

三种可用的控制器:

- Job的重启策略只适合OnFailre以及Never
- RC，RS，Deployment的重启策略只适合Always。
- DS的Pod运行在每一台机器上。

这三种类型的控制使用PodTemplate来创建Pod。使用合适的控制器来创建合适的Pod。

如果节点失去连接，则此节点的Pod都会设置为Failed。

**案例：**

Advanced liveness probe example

Liveness probes are executed by the kubelet, so all requests are made in the kubelet network namespace.

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    test: liveness
  name: liveness-http
spec:
  containers:
  - args:
    - /server
    image: k8s.gcr.io/liveness
    livenessProbe:
      httpGet:
        # when "host" is not defined, "PodIP" will be used
        # host: my-host
        # when "scheme" is not defined, "HTTP" scheme will be used. Only "HTTP" and "HTTPS" are allowed
        # scheme: HTTPS
        path: /healthz
        port: 8080
        httpHeaders:
        - name: X-Custom-Header
          value: Awesome
      initialDelaySeconds: 15
      timeoutSeconds: 1
    name: liveness
```

##### Example states

- Pod is running and has one Container. Container exits with success.
  - Log completion event.
  - If restartPolicy is:
    - Always: Restart Container; Pod `phase` stays Running.
    - OnFailure: Pod `phase` becomes Succeeded.
    - Never: Pod `phase` becomes Succeeded.
- Pod is running and has one Container. Container exits with failure.
  - Log failure event.
  - If restartPolicy is:
    - Always: Restart Container; Pod `phase` stays Running.
    - OnFailure: Restart Container; Pod `phase` stays Running.
    - Never: Pod `phase` becomes Failed.
- Pod is running and has two Containers. Container 1 exits with failure.
  - Log failure event.
  - If restartPolicy is:
    - Always: Restart Container; Pod `phase` stays Running.
    - OnFailure: Restart Container; Pod `phase` stays Running.
    - Never: Do not restart Container; Pod `phase` stays Running.
  - If Container 1 is not running, and Container 2 exits:
    - Log failure event.
    - If restartPolicy  is:
      - Always: Restart Container; Pod `phase` stays Running.
      - OnFailure: Restart Container; Pod `phase` stays Running.
      - Never: Pod `phase` becomes Failed.
- Pod is running and has one Container. Container runs out of memory.
  - Container terminates in failure.
  - Log OOM event.
  - If restartPolicy is:
    - Always: Restart Container; Pod `phase` stays Running.
    - OnFailure: Restart Container; Pod `phase` stays Running.
    - Never: Log failure event; Pod `phase` becomes Failed.
- Pod is running, and a disk dies.
  - Kill all Containers.
  - Log appropriate event.
  - Pod `phase` becomes Failed.
  - If running under a controller, Pod is recreated elsewhere.
- Pod is running, and its node is segmented out.
  - Node controller waits for timeout.
  - Node controller sets Pod `phase` to Failed.
  - If running under a controller, Pod is recreated elsewhere.

#### 初始化容器

#### 预设置

#### 拓扑

#### Disruptions

#### 瞬时容器

### Controller

## 服务，负载均衡，网络

### Endpoint Slices

### 服务

### 服务拓扑

### 服务与Pods的DNS

### 使用服务连接应用程序

### Ingress

### Ingress Controllers

### 网络策略

### 使用主机别名添加入口至Pod的/ets/hosts

### IPv4/IPv6 dual-stack

## 存储

### 卷

### 持久卷

### 卷快照

### CSI卷克隆

### 存储类

### 卷快照类

### 动态卷供应

### 特殊节点卷限制

## 配置

### 配置最佳实践

### Resource Bin Packing for Extended Resources

### 管理容器计算资源

### Pod Overhead

### 赋予Pod至节点

### 污点与容错

### Secrets

### 使用kubeconfig文件组织集群访问

### Pod优先级与抢占

### 调度框架

## 安全

## 策略

### Limit Ranges

### Resource Quotas

### Pod Security Policies

## 调度

### K8S调度器

### 调度器性能调优

## 集群管理

### 集群管理大纲

### 证书

### 云提供商

### 管理资源

### 集群网络

### 日志架构

### 配置kubelet垃圾回收器

### Federation

### K8S中的代理

### Controller manager metrics

### 安装插件

## 扩展