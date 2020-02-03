# Kubernetes

## 谷歌镜像加速

1. docker.io加速，替换为dockerhub.azk8s.cn。
2. gcr.io加速，替换为 gcr.azk8s.cn/google_containers。
3. k8s.grc.io加速，替换为gcr.azk8s.cn/google-containers。
4. quay.io加速，替换为quay.azk8s.cn。

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

### 官网教程

1. 创建一个集群![img](https://d33wubrfki0l68.cloudfront.net/99d9808dcbf2880a996ed50d308a186b5900cec9/40b94/docs/tutorials/kubernetes-basics/public/images/module_01_cluster.svg)

   ```
     minikube version
         minikube start
         kubectl version
         kubectl cluster-info
         kubectl get nodes
   ```

2. 部署一个应用

![img](https://d33wubrfki0l68.cloudfront.net/152c845f25df8e69dd24dd7b0836a289747e258a/4a1d2/docs/tutorials/kubernetes-basics/public/images/module_02_first_app.svg)

```
kubectl version
kubectl get nodes
kubectl create deployment nginx --image=nginx
kubectl get deployments
kubectl proxy
curl localhost:8001
```

**Deployments负责创建，更新你的应用程序实例。**

**部署在kubernets中的应用需要容器化。**



3.探索你的应用

![img](https://d33wubrfki0l68.cloudfront.net/fe03f68d8ede9815184852ca2a4fd30325e5d15a/98064/docs/tutorials/kubernetes-basics/public/images/module_03_pods.svg)

***A Pod is a group of one or more application containers (such as Docker or rkt) and includes shared storage (volumes), IP address and information about how to run them.***

![img](https://d33wubrfki0l68.cloudfront.net/5cb72d407cbe2755e581b6de757e0d81760d5b86/a9df9/docs/tutorials/kubernetes-basics/public/images/module_03_nodes.svg)

***Containers should only be scheduled together in a single Pod if they are tightly coupled and need to share resources such as disk.***

***A node is a worker machine in Kubernetes and may be a VM or physical machine, depending on the cluster. Multiple Pods can run on one Node.***

```shell
kubectl get pods
kubectl describe pods
echo -e "\n\n\n\e[92mStarting Proxy. After starting it will not output a response. Please click the first Terminal Tab\n"; kubectl proxy
export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
echo Name of the Pod: $POD_NAME
curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/proxy/
kubectl logs $POD_NAME
kubectl exec $POD_NAME env
kubectl exec -ti $POD_NAME bash
cat server.js
curl localhost:8080
exit
```

**4.暴露你的服务**

![img](https://d33wubrfki0l68.cloudfront.net/cc38b0f3c0fd94e66495e3a4198f2096cdecd3d5/ace10/docs/tutorials/kubernetes-basics/public/images/module_04_services.svg)

***A Kubernetes Service is an abstraction layer which defines a logical set of Pods and enables external traffic exposure, load balancing and service discovery for those Pods.***

***You can create a Service at the same time you create a Deployment by using
`--expose` in kubectl.***

![img](https://d33wubrfki0l68.cloudfront.net/b964c59cdc1979dd4e1904c25f43745564ef6bee/f3351/docs/tutorials/kubernetes-basics/public/images/module_04_labels.svg)

```shell 
kubectl get pods #查询Pods
kubectl get services #查询服务
kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080 #暴露服务
kubectl get services #查询服务
kubectl describe services/kubernetes-bootcamp #查询服务详情
export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')#导出服务端口
echo NODE_PORT=$NODE_PORT
curl $(minikube ip):$NODE_PORT#验证服务
kubectl describe deployment#查询deploymeng详情
kubectl get pods -l run=kubernetes-bootcamp#根据标签查询Pods
kubectl get services -l run=kubernetes-bootcamp#根据标签查询服务
export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')#导出POD_NAME
echo Name of the Pod: $POD_NAME
kubectl label pod $POD_NAME app=v1#给Pod加上新标签
kubectl describe pods $POD_NAME#查询Pods详情，验证新标签
kubectl get pods -l app=v1#根据新标签查询Pods
kubectl delete service -l run=kubernetes-bootcamp#根据标签删除服务
kubectl get services#验证服务是否删除
curl $(minikube ip):$NODE_PORT#验证外部是否可访问
kubectl exec -ti $POD_NAME curl localhost:8080#验证Pods内部是否可访问

```

5.**扩容**

**You can create from the start a Deployment with multiple instances using the --replicas parameter for the kubectl run command**

**Scaling is accomplished by changing the number of replicas in a Deployment.**

```shell
kubectl get deployments
kubectl scale deployments/kubernetes-bootcamp --replicas=4
kubectl get deployments
kubectl get pods -o wide
kubectl describe deployments/kubernetes-bootcamp
kubectl describe services/kubernetes-bootcamp
export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
echo NODE_PORT=$NODE_PORT
curl $(minikube ip):$NODE_PORT
curl $(minikube ip):$NODE_PORT
curl $(minikube ip):$NODE_PORT
curl $(minikube ip):$NODE_PORT

kubectl scale deployments/kubernetes-bootcamp --replicas=2
kubectl get deployments
kubectl get pods -o wide

```

6.**滚动更新**

**Rolling updates allow Deployments' update to take place with zero downtime by incrementally updating Pods instances with new ones.**

![img](https://d33wubrfki0l68.cloudfront.net/30f75140a581110443397192d70a4cdb37df7bfc/fa906/docs/tutorials/kubernetes-basics/public/images/module_06_rollingupdates1.svg)

![img](https://d33wubrfki0l68.cloudfront.net/678bcc3281bfcc588e87c73ffdc73c7a8380aca9/703a2/docs/tutorials/kubernetes-basics/public/images/module_06_rollingupdates2.svg)

![img](https://d33wubrfki0l68.cloudfront.net/9b57c000ea41aca21842da9e1d596cf22f1b9561/91786/docs/tutorials/kubernetes-basics/public/images/module_06_rollingupdates3.svg)

![img](https://d33wubrfki0l68.cloudfront.net/6d8bc1ebb4dc67051242bc828d3ae849dbeedb93/fbfa8/docs/tutorials/kubernetes-basics/public/images/module_06_rollingupdates4.svg)



**If a Deployment is exposed publicly, the Service will load-balance the traffic only to available Pods during the update.**

```shell
kubectl get deployments
kubectl get pods
kubectl describe pods
kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
kubectl get pods
kubectl describe services/kubernetes-bootcamp
export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
echo NODE_PORT=$NODE_PORT
curl $(minikube ip):$NODE_PORT
kubectl rollout status deployments/kubernetes-bootcamp
kubectl describe pods
kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=gcr.io/google-samples/kubernetes-bootcamp:v10
kubectl get deployments
kubectl get pods
kubectl describe pods
kubectl rollout undo deployments/kubernetes-bootcamp
kubectl get pods
kubectl describe pods

```



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
kubeadm init --config kubeadm.yml --upload-certs | tee init.log

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

## 配置iptables转发链启用

```
vi /etc/sysctl.conf
net.ipv4.ip_forward=1
reboot
#sysctl -w net.ipv4.ip_forward=1
```



## 配置Docker远程链接

```json
{
    "registry-mirrors": ["https://a17tqp4p.mirror.aliyuncs.com"]，
    "hosts": ["unix:///var/run/docker.sock","tcp://127.0.0.1:2375","tcp://192.168.2.10:2375"]

}
```

## 配置动态PV

```sh
#安装helm 安装nfs nfs-utils nfs-common
helm install  --set nfs.server=192.168.2.40 --set nfs.path=/mnt/sharedfolder nfs-provisioner stable/nfs-client-provisioner
#注意镜像拉取，设置加速，或者tag方式
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

