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
3. matadata：对象元素，用于标志对象的唯一性。
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

一个选择器可以由多个条件组成，用逗号分隔。每个逗号相当于逻辑或运算符。

空的选择器或未指定的选择器取决于上下文，使用选择器的API类型，应当记录他们的有效性以及含义。

某些选择器，选择的资源不能重叠，例如ReplicaSet。

#### 等价行选择器

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

