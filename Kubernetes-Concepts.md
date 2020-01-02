# K8S概念

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

