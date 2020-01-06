# Kubernetes

## 谷歌镜像构建

1. 构建github仓库，编写dockerfile
2. 构建阿里云镜像仓库，选择海外机器构建
3. 拉取镜像到本地 ，tag方式 重命名为谷歌镜像

参考链接:https://www.jianshu.com/p/21cd9aeee12b

```dockerfile
FROM k8s.gcr.io/kube-apiserver:v1.17.0
LABEL maintainer="hiscat <1251723871@qq.com>" 
```

```shell
docker pull registry.cn-hangzhou.aliyuncs.com/twocat/kube-apiserver:1.17.0
docker tag registry.cn-hangzhou.aliyuncs.com/twocat/kube-apiserver:1.17.0 k8s.gcr.io/kube-apiserver:v1.17.0
```



## Minikube

### Features

1. DNS
2. NodePorts
3. ConfigMaps and Secrets
4. Dashboards
5. Container Runtime: Docker, CRI-O,and Cotainerd
6. Enabling CNI(Container Network Interface)
7. Ingress

### Install

参考Ubuntu.md

## 搭建Cluster

1. **搭建之前，先下载谷歌镜像，参考谷歌镜像构建一节。确保每个节点都有镜像**

   1. ```shell
      docker pull registry.cn-hangzhou.aliyuncs.com/twocat/kube-apiserver:1.17.0
      docker pull registry.cn-hangzhou.aliyuncs.com/twocat/kube-controller-manager:1.17.0
      docker pull registry.cn-hangzhou.aliyuncs.com/twocat/kube-proxy:1.17.0
      docker pull registry.cn-hangzhou.aliyuncs.com/twocat/kube-scheduler:1.17.0
      docker pull registry.cn-hangzhou.aliyuncs.com/twocat/etcd:3.4.3-0
      docker pull registry.cn-hangzhou.aliyuncs.com/twocat/coredns:1.6.5
      docker pull registry.cn-hangzhou.aliyuncs.com/twocat/pause:3.1
      docker pull registry.cn-hangzhou.aliyuncs.com/twocat/nginx-ingress-controller:0.26.2
      
      docker tag registry.cn-hangzhou.aliyuncs.com/twocat/kube-apiserver:1.17.0 k8s.gcr.io/kube-apiserver:1.17.0
      docker tag registry.cn-hangzhou.aliyuncs.com/twocat/kube-controller-manager:1.17.0 k8s.gcr.io/kube-controller-manager:1.17.0
      docker tag registry.cn-hangzhou.aliyuncs.com/twocat/kube-proxy:1.17.0 k8s.gcr.io/kube-proxy:1.17.0
      docker tag registry.cn-hangzhou.aliyuncs.com/twocat/kube-scheduler:1.17.0 k8s.gcr.io/kube-scheduler:1.17.0
      docker tag registry.cn-hangzhou.aliyuncs.com/twocat/etcd:3.4.3-0 k8s.gcr.io/etcd:3.4.3-0
      docker tag registry.cn-hangzhou.aliyuncs.com/twocat/coredns:1.6.5 k8s.gcr.io/coredns:1.6.5
      docker tag registry.cn-hangzhou.aliyuncs.com/twocat/pause:3.1 k8s.gcr.io/pause:3.1
      docker tag registry.cn-hangzhou.aliyuncs.com/twocat/nginx-ingress-controller:0.26.2 quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.26.2
      
      docker rmi registry.cn-hangzhou.aliyuncs.com/twocat/kube-apiserver:1.17.0
      docker rmi registry.cn-hangzhou.aliyuncs.com/twocat/kube-controller-manager:1.17.0
      docker rmi registry.cn-hangzhou.aliyuncs.com/twocat/kube-proxy:1.17.0
      docker rmi registry.cn-hangzhou.aliyuncs.com/twocat/kube-scheduler:1.17.0
      docker rmi registry.cn-hangzhou.aliyuncs.com/twocat/etcd:3.4.3-0
      docker rmi registry.cn-hangzhou.aliyuncs.com/twocat/coredns:1.6.5
      docker rmi registry.cn-hangzhou.aliyuncs.com/twocat/pause:3.1
      docker rmi registry.cn-hangzhou.aliyuncs.com/twocat/nginx-ingress-controller:0.26.2
      ```

2. 初始化master节点

3. 配置网络插件calico

   1. ```shell
      wget https://docs.projectcalico.org/v3.7/manifests/calico.yaml
      需要修改Deployment DaemonSet配置的apiVersion: apps/v1，注意加上spec.selector
      ```

4. 运行实例

   ```
   kubectl get cs
   kubectl cluster-info
   kubectl run nginx --image=nginx --replicas=2 --port=80
   kubectl get pods
   kubectl get deployment
   kubectl expose deployment nginx --port=80 --type=LoadBalancer
   kubectl get services
   kubectl describe service nginx
   kubectl delete service nginx
   kubectl delete deployment nginx
   ```

   

calico：https://docs.projectcalico.org/v3.11/getting-started/kubernetes/

kubeadm：https://kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

```yml
apiVersion: kubeadm.k9s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.2.10 #替换成本机IP
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: ubuntu18-server-master
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.aliyuncs.com/google_containers
kind: ClusterConfiguration
kubernetesVersion: v1.17.0
networking:
  dnsDomain: cluster.local
  podSubnet: 10.244.0.0/16 #添加POD子网
  serviceSubnet: 10.96.0.0/12
scheduler: {}

```

```shell
kubeadm cofig print ini-defaults >> kubeadm.yml
kubeadm init --config kubeadm.yml --upload-certs | tee kubeadm-init.log

#一定要在每个节点都有admin.conf 
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
kubectl apply -f https://docs.projectcalico.org/v3.11/manifests/calico.yaml
```

## 配置kubectl命令行提示

```shell
 # Installing bash completion on Linux
  ## If bash-completion is not installed on Linux, please install the 'bash-completion' package
  ## via your distribution's package manager.
  ## Load the kubectl completion code for bash into the current shell
  source <(kubectl completion bash)
  ## Write bash completion code to a file and source if from .bash_profile
  kubectl completion bash > ~/.kube/completion.bash.inc
  printf "
  # Kubectl shell completion
  source '$HOME/.kube/completion.bash.inc'
  " >> $HOME/.bash_profile
  source $HOME/.bash_profile

```



## 配置Docker远程链接

```json
{
    "registry-mirrors": ["https://a17tqp4p.mirror.aliyuncs.com"]，
    "hosts": ["unix:///var/run/docker.sock","tcp://127.0.0.1:2375","tcp://192.168.2.10:2375"]

}
```





## 通过资源配置运行容器

### 创建Deployment

```shell
kubectl delete deployments nginx-deployment
tee nginx-deployment.yaml <<- 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
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
        
EOF
kubectl create -f nginx-deployment.yaml 
kubectl get deployments.apps 
kubectl get pods
```



### 创建Service

```shell
kubectl delete services nginx-app

tee nginx-service.yml <<- 'EOF'
apiVersion: v1
kind: Service
metadata:
  name: nginx-app
spec:
  selector:
    app: nginx
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
EOF

kubectl create -f nginx-service.yml
kubectl get services
```

### 集成创建

```shell
kubectl delete deployments nginx-deployment
kubectl delete services nginx-app

tee nginx.yml <<- 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
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
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-app
spec:
  selector:
    app: nginx
  type: LoadBalancer
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
EOF

kubectl create -f nginx.yml
```



### 查看Service/Deployment/Pods

```shell
kubectl get services
kubectl get deployments
kubectl get pods
```

### 删除Service/Deployment

```shell
kubectl delete service nginx-app
kubectl get deployment nginx-deployment
```

### 修改端口范围

```shell
vi /etc/kubernetes/manifests/kube-apiserver.yaml
# --service-node-port-range=2-65535
```

## Ingress

### 裸机安装Ingress(Bare-metal)

```bash
#wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml
#https://github.com/kubernetes/ingress-nginx/blob/master/deploy/static/mandatory.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml
#基础设置

#服务设置
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/baremetal/service-nodeport.yaml
```

### 注意手动拉取镜像

```yml
      containers:
        - name: nginx-ingress-controller
          image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.26.2

```

**提前在从机上拉取ingress镜像，不然构建不成功。参考，谷歌镜像构建**

### 部署应用

```shell
tee spring-hello.yml <<- 'EOF'
apiVersion: apps/v1
kind: Deployment
metadata:
  name: spring-hello
spec:
  replicas: 2
  selector:
    matchLabels:
      app: spring-hello
  template:
    metadata:
      labels:
        app: spring-hello
    spec:
      containers:
        - name: spring-hello
          image: hiscat/spring-hello
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 8080
---

apiVersion: v1
kind: Service
metadata:
  name: spring-hello-service
spec:
  ports:
    - port: 8080
      targetPort: 8080

  type: NodePort
  selector:
    app: spring-hello
EOF
kubectl apply -f spring-hello.yml
```

### 部署Ingress

```shell
tee ingress.yml <<- 'EOF'
apiVersion: networking.k8s.io/v1beta1 # for versions before 1.14 use extensions/v1beta1
kind: Ingress
metadata:
  name: spring-hello-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
  rules:
    - host: springhello.info
      http:
        paths:
          - path: /
            backend:
              serviceName: spring-hello-service
              servicePort: 8080
EOF
kubectl apply -f ingress.yml
```

### 验证

```shell
kubectl get ingress
#NAME              HOSTS              ADDRESS       PORTS     AGE
#example-ingress   springhello.info   172.17.0.15   80        38s
vim /etc/hosts #注意添加host文件 
curl springhello.info/hello
```

### 参考文献

minikube-ingress：https://kubernetes.io/docs/tasks/access-application-cluster/ingress-minikube/

ingress-install：https://kubernetes.github.io/ingress-nginx/deploy/

李卫民-博客：[https://www.funtl.com/zh/service-mesh-kubernetes/Ingress-%E7%BB%9F%E4%B8%80%E8%AE%BF%E9%97%AE%E5%85%A5%E5%8F%A3.html#%E6%9C%AC%E8%8A%82%E8%A7%86%E9%A2%91](https://www.funtl.com/zh/service-mesh-kubernetes/Ingress-统一访问入口.html#本节视频)

