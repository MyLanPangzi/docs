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

