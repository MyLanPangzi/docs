# K8S-Lab

## 多Pods部署微服务

### Goal

- 部署一个Server。

- 部署一个Provider。

- Provider通过Service Name注册到Server。
- 暴露Server至外部，可通过IP:Port访问，验证Provider是否注册成功。

### Steps

1. 编写Server

   1. pom.xml，使用谷歌的jib插件进行容器化

      ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
          <modelVersion>4.0.0</modelVersion>
          <parent>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-parent</artifactId>
              <version>2.2.2.RELEASE</version>
              <relativePath/> <!-- lookup parent from repository -->
          </parent>
          <groupId>com.hiscat</groupId>
          <artifactId>spring-discovery</artifactId>
          <version>0.0.1-SNAPSHOT</version>
          <name>spring-discovery</name>
          <description>spring-discovery</description>
      
          <properties>
              <java.version>1.8</java.version>
              <spring-cloud.version>Hoxton.SR1</spring-cloud.version>
          </properties>
      
          <dependencies>
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-starter-web</artifactId>
              </dependency>
              <dependency>
                  <groupId>org.springframework.cloud</groupId>
                  <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
              </dependency>
      
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-devtools</artifactId>
                  <scope>runtime</scope>
                  <optional>true</optional>
              </dependency>
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-configuration-processor</artifactId>
                  <optional>true</optional>
              </dependency>
              <dependency>
                  <groupId>org.projectlombok</groupId>
                  <artifactId>lombok</artifactId>
                  <optional>true</optional>
              </dependency>
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-starter-test</artifactId>
                  <scope>test</scope>
                  <exclusions>
                      <exclusion>
                          <groupId>org.junit.vintage</groupId>
                          <artifactId>junit-vintage-engine</artifactId>
                      </exclusion>
                  </exclusions>
              </dependency>
          </dependencies>
      
          <dependencyManagement>
              <dependencies>
                  <dependency>
                      <groupId>org.springframework.cloud</groupId>
                      <artifactId>spring-cloud-dependencies</artifactId>
                      <version>${spring-cloud.version}</version>
                      <type>pom</type>
                      <scope>import</scope>
                  </dependency>
              </dependencies>
          </dependencyManagement>
      
          <build>
              <plugins>
                  <plugin>
                      <groupId>org.springframework.boot</groupId>
                      <artifactId>spring-boot-maven-plugin</artifactId>
                  </plugin>
                  <plugin>
                      <groupId>com.google.cloud.tools</groupId>
                      <artifactId>jib-maven-plugin</artifactId>
                      <version>1.8.0</version>
                      <executions>
                          <execution>
                              <phase>package</phase>
                              <goals>
                                  <goal>build</goal>
                              </goals>
                          </execution>
                      </executions>
                      <configuration>
                          <from>
                              <image>openjdk:8-jdk-alpine</image>
                          </from>
                          <to>
                              <image>registry.cn-hangzhou.aliyuncs.com/twocat/spring-discovery-server</image>
                          </to>
                      </configuration>
                  </plugin>
              </plugins>
          </build>
      <!-- 需要配置阿里云的账号密码，在settings里的server节点-->
      </project>
      
      ```

   2. application.yml

      ```yml
      server:
        port: 19000
      eureka:
        client:
          fetch-registry: false
          register-with-eureka: false
      ```

   3. 代码

      ```java
      package com.hiscat.springdiscovery;
      
      import org.springframework.boot.SpringApplication;
      import org.springframework.boot.autoconfigure.SpringBootApplication;
      import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;
      
      /**
       * @author Administrator
       */
      @SpringBootApplication
      @EnableEurekaServer
      public class SpringDiscoveryApplication {
      
          public static void main(String[] args) {
              SpringApplication.run(SpringDiscoveryApplication.class, args);
          }
      
      }
      
      ```

      

2. 容器化Server：通过Jib插件实现

3. 编写Deployment以及Service

   ```yml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: server-deployment
     labels:
       app.kubernetes.io/name: server
       app.kubernetes.io/instance: server-instance
       app.kubernetes.io/component: discovery
       app.kubernetes.io/version: 1.0.0
       app.kubernetes.io/part-of: spring-cloud-labs
   spec:
     selector:
       matchLabels:
         app.kubernetes.io/name: server
         app.kubernetes.io/instance: server-instance
         app.kubernetes.io/component: discovery
         app.kubernetes.io/version: 1.0.0
         app.kubernetes.io/part-of: spring-cloud-labs
     template:
       metadata:
         labels:
           app.kubernetes.io/name: server
           app.kubernetes.io/instance: server-instance
           app.kubernetes.io/component: discovery
           app.kubernetes.io/version: 1.0.0
           app.kubernetes.io/part-of: spring-cloud-labs
       spec:
         containers:
           - name: server
             image: registry.cn-hangzhou.aliyuncs.com/twocat/spring-discovery-server
             ports:
               - containerPort: 19000
   ---
   apiVersion: v1
   kind: Service
   metadata:
     name: server-service
     labels:
       app.kubernetes.io/name: server
       app.kubernetes.io/instance: server-instance
       app.kubernetes.io/component: discovery
       app.kubernetes.io/version: 1.0.0
       app.kubernetes.io/part-of: spring-cloud-labs
   spec:
     selector:
       app.kubernetes.io/name: server
       app.kubernetes.io/instance: server-instance
       app.kubernetes.io/component: discovery
       app.kubernetes.io/version: 1.0.0
       app.kubernetes.io/part-of: spring-cloud-labs
     type: NodePort
     ports:
       - port: 19000
         targetPort: 19000
   ```

   

4. 部署Server

   ```
   kubectl apply -f server.yml
   ```

   

5. 编写Provider

   1. pom.xml

      ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
               xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
          <modelVersion>4.0.0</modelVersion>
          <parent>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-parent</artifactId>
              <version>2.2.2.RELEASE</version>
              <relativePath/> <!-- lookup parent from repository -->
          </parent>
          <groupId>com.hiscat</groupId>
          <artifactId>spring-discovery-client</artifactId>
          <version>0.0.1-SNAPSHOT</version>
          <name>spring-discovery-client</name>
          <description>spring-discovery-client</description>
      
          <properties>
              <java.version>1.8</java.version>
              <spring-cloud.version>Hoxton.SR1</spring-cloud.version>
          </properties>
      
          <dependencies>
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-starter-web</artifactId>
              </dependency>
              <dependency>
                  <groupId>org.springframework.cloud</groupId>
                  <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
              </dependency>
      
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-devtools</artifactId>
                  <scope>runtime</scope>
                  <optional>true</optional>
              </dependency>
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-configuration-processor</artifactId>
                  <optional>true</optional>
              </dependency>
              <dependency>
                  <groupId>org.projectlombok</groupId>
                  <artifactId>lombok</artifactId>
                  <optional>true</optional>
              </dependency>
              <dependency>
                  <groupId>org.springframework.boot</groupId>
                  <artifactId>spring-boot-starter-test</artifactId>
                  <scope>test</scope>
                  <exclusions>
                      <exclusion>
                          <groupId>org.junit.vintage</groupId>
                          <artifactId>junit-vintage-engine</artifactId>
                      </exclusion>
                  </exclusions>
              </dependency>
          </dependencies>
      
          <dependencyManagement>
              <dependencies>
                  <dependency>
                      <groupId>org.springframework.cloud</groupId>
                      <artifactId>spring-cloud-dependencies</artifactId>
                      <version>${spring-cloud.version}</version>
                      <type>pom</type>
                      <scope>import</scope>
                  </dependency>
              </dependencies>
          </dependencyManagement>
      
          <build>
              <plugins>
                  <plugin>
                      <groupId>org.springframework.boot</groupId>
                      <artifactId>spring-boot-maven-plugin</artifactId>
                  </plugin>
                  <plugin>
                      <groupId>com.google.cloud.tools</groupId>
                      <artifactId>jib-maven-plugin</artifactId>
                      <version>1.8.0</version>
                      <executions>
                          <execution>
                              <phase>package</phase>
                              <goals>
                                  <goal>build</goal>
                              </goals>
                          </execution>
                      </executions>
                      <configuration>
                          <from>
                              <image>openjdk:8-jdk-alpine</image>
                          </from>
                          <to>
                              <image>registry.cn-hangzhou.aliyuncs.com/twocat/spring-discovery-client</image>
                          </to>
                      </configuration>
                  </plugin>
              </plugins>
          </build>
      <!-- 注意配置settings的server节点 -->
      </project>
      
      ```

   2. yml

      ```yml
      spring:
        application:
          name: hello
      #这里的注册中心地址可以改。通过commond命令修改。
      eureka:
        client:
          service-url:
            defaultZone: http://server-service:19000/eureka
        instance:
          instance-id: ${spring.cloud.client.ip-address}:${server.port}
          prefer-ip-address: true
      server:
        port: 8080
      ```

   3. 代码

      ```java
      package com.hiscat.springdiscoveryclient;
      
      import org.springframework.boot.SpringApplication;
      import org.springframework.boot.autoconfigure.SpringBootApplication;
      import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
      
      /**
       * @author Administrator
       */
      @SpringBootApplication
      @EnableDiscoveryClient
      public class SpringDiscoveryClientApplication {
      
          public static void main(String[] args) {
              SpringApplication.run(SpringDiscoveryClientApplication.class, args);
          }
      
      }
      
      ```

      

6. 容器化Provider：通过Jib插件实现

7. 编写Deployment

   ```yml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: client-deployment
     labels:
       app.kubernetes.io/name: client
       app.kubernetes.io/instance: client-instance
       app.kubernetes.io/component: client
       app.kubernetes.io/version: 1.0.0
       app.kubernetes.io/part-of: spring-cloud-labs
   spec:
     replicas: 2
     selector:
       matchLabels:
         app.kubernetes.io/name: client
         app.kubernetes.io/instance: client-instance
         app.kubernetes.io/component: client
         app.kubernetes.io/version: 1.0.0
         app.kubernetes.io/part-of: spring-cloud-labs
     template:
       metadata:
         labels:
           app.kubernetes.io/name: client
           app.kubernetes.io/instance: client-instance
           app.kubernetes.io/component: client
           app.kubernetes.io/version: 1.0.0
           app.kubernetes.io/part-of: spring-cloud-labs
       spec:
         containers:
           - name: client
             image: registry.cn-hangzhou.aliyuncs.com/twocat/spring-discovery-client
             ports:
               - containerPort: 8080
             env:
               - name: EUREKA.SERVER
                 value: --eureka.client.serviceUrl.defaultZone=http://server-service:19000/eureka
             command:
               - sh
               - -c
               - java ${JAVA_OPTS} -cp /app/resources:app/classes:app/libs com.hiscat.springdiscoveryclient.SpringDiscoveryClientApplication ${EUREKA.SERVER}
   #这里的cmmond可以替换端口啥的。有点长，可以替换用环境变量替换。
   #注册中心的域名一定要写Server暴露的服务名
   ```

   

8. 部署Provider

   ```
   kubectl apply -f client.yml
   ```

   

### Summary

**Pod可以与Service进行通讯，可以使用服务名作为域名，进行DNS解析。**

**其实服务的默认DNS为：服务名.default.svc.cluster.local。**

## 探索容器生命周期

### Goal

掌握容器生命周期的执行顺序，以及定义。

### Steps

1. 使用初始化容器，初始化一个卷，在卷中放置一个hello文件，并追加init。

2. 容器启动后执行命令，echo app >> /data/hello && sleep 100;

3. 使用postStart钩子，追加postStart。

4. ~~使用startupProbe，追加startupProbe，周期为2S，失败阈值为10。目前还不知道如何启用alpha特性。。此步骤先省略~~

5. 使用readinessProbe，1秒后追加readinessProbe，周期为100S。

6. 使用livenessProbe，3秒后追加livenessProbe，周期为100S。

7. 将卷挂载至容器的/data目录。

8. 使用preStop钩子，追加preStop，并sleep 10S。

9. 使用kubectl delete删除Pod。

10. 在容器删除之前，开启一个新的终端，使用kubectl cp命令将文件拷贝至~目录，验证文件内容是否与下面输出一致。

    ```shell
    init
    app
    postStart
    readinessProbe
    livenessProbe 
    preStop
    ```

### Suammary

init -> app -> post -> **startup** -> readiness | liveness -> pre

探针的首次启动时间为**初始延迟时间+周期时间**

## 部署StatefulSet应用

### Goal   

1. 掌握PV，PVC的使用。
2. 掌握无头服务的使用。
3. 掌握状态集应用的部署。
4. 观察状态集应用的启停顺序。
5. 使用curl观察，Pod的DNS。
6. 回收PV。

### Steps

1. 安装NFS

   ```shell
   apt-get update && apt install nfs-kernel-server -y
   mkdir -p /mnt/nfs{1..3}
   chown nobody:nogroup /mnt/nfs{1..3}
   chmod 777 /mnt/nfs{1..3}
   vim /etc/exports
   #/mnt/nfs1 192.168.2.0/24(rw,sync,no_subtree_check)
   #/mnt/nfs2 192.168.2.0/24(rw,sync,no_subtree_check)
   #/mnt/nfs3 192.168.2.0/24(rw,sync,no_subtree_check)
   exportfs -a
   systemctl restart nfs-kernel-server
   #ufw allow from [clientIP or clientSubnetIP] to any port nfs
   #ufw allow from 192.168.100/24 to any port nfs
   ufw status
   tee /mnt/nfs1/index.html <<-EOF 
   nfs1
   EOF
   tee /mnt/nfs2/index.html <<-EOF 
   nfs2
   EOF
   tee /mnt/nfs3/index.html <<-EOF 
   nfs3
   EOF
   
   ```

2. 编写PV

   ```yml
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: nfs1
   spec:
     capacity:
       storage: 5Gi
     volumeMode: Filesystem
     accessModes:
       - ReadWriteOnce
     persistentVolumeReclaimPolicy: Retain
     storageClassName: nfs
     nfs:
       path: /mnt/nfs1
       server: 192.168.2.40
   ---
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: nfs2
   spec:
     capacity:
       storage: 5Gi
     volumeMode: Filesystem
     accessModes:
       - ReadWriteOnce
     persistentVolumeReclaimPolicy: Retain
     storageClassName: nfs
     nfs:
       path: /mnt/nfs2
       server: 192.168.2.40
   ---
   apiVersion: v1
   kind: PersistentVolume
   metadata:
     name: nfs3
   spec:
     capacity:
       storage: 5Gi
     volumeMode: Filesystem
     accessModes:
       - ReadWriteOnce
     persistentVolumeReclaimPolicy: Retain
     storageClassName: nfs
     nfs:
       path: /mnt/nfs3
       server: 192.168.2.40
   
   ```

3. 编写无头服务

   ```yml 
   apiVersion: v1
   kind: Service
   metadata:
     name: web
   spec:
     clusterIP: 'None'
     selector:
       app: nginx
   ```

4. 编写状态集

   ```yml
   apiVersion: apps/v1
   kind: StatefulSet
   metadata:
     name: web
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: nginx
     serviceName: 'web'
     volumeClaimTemplates:
       - metadata:
           name: nfs
         spec:
           resources:
             requests:
               storage: '3Gi'
           storageClassName: 'nfs'
           accessModes: ['ReadWriteOnce']
     template:
       metadata:
         labels:
           app: nginx
       spec:
         containers:
           - name: nginx
             image: nginx
             ports:
               - containerPort: 80
             volumeMounts:
               - mountPath: /usr/share/nginx/html
                 name: nfs
   ```

5. 观察启停顺序

   ```shell
   kubectl delete sts web
   kubectl apply -f -<<EOF   #拷贝上面的状态集
   ```

6. 新起一个Pod观察DNS

   ```shell
   kubectl get ep #观察无头服务的Endpoint
   kubectl apply -f -<<EOF
   apiVersion: v1
   kind: Pod
   metadata:
     name: nginx
   spec:
     containers:
       - name: nginx
         image: nginx
   EOF
   kubectl exec -it nginx -- bash
   apt-get update && apt-get install -y curl
   curl web && curl web-0.web && curl web-1.web && curl web-2.web
   curl web && curl web-0.web && curl web-1.web && curl web-2.web
   curl web && curl web-0.web && curl web-1.web && curl web-2.web
   ```

7. 回收PV

   ```shell
   kubectl delete sts web --forece --grace-period 0
   kubectl get pvc
   kubectl delete pvc --all 
   kubectl get pv
   kubectl edit pv nfs1 #删除引用块，下面状态会变成可用
   kubectl get pv
   NAME   CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS     CLAIM               STORAGECLASS   REASON   AGE
   nfs1   5Gi        RWO            Retain           Released   default/nfs-web-0   nfs                     110m
   nfs2   5Gi        RWO            Retain           Released   default/nfs-web-1   nfs                     110m
   nfs3   5Gi        RWO            Retain           Released   default/nfs-web-2   nfs                     110m
   
   ```

   

### Summary

PV独立于Pod的生命周期，Pod死亡后，PV的内容不会丢失。

Volume独立于容器的生命周期，容器死亡后，只要Pod还在，Volume的内容不会丢失。

状态集应用的DNS，分配为podName.serviceName.svc.default.cluster.local。

Pod的启动是顺序启动，从0开始，上一个Pod必须是Ready或Running时，下一个Pod才会启动。

Pod的停止是倒序停止，从最后一个开始。