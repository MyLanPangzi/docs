# 1 ES安装

## **（ES7.5需要JDK版本8以上，且配置JAVA_HOME环境变量）**

## Windows安装

### [下载安装包](https://www.elastic.co/cn/downloads/elasticsearch)

### 修改配置文件

​	**config/elasticsearch.yml**

```yaml
action.auto_create_index: .monitoring*,.watches,.triggered_watches,.watcher-history*,.ml*
```



### 启动

 ![image-20191221200535649](images\es-start-win.png)

### 验证

![image-20191221200744557](images\es-validate-win.png)

### 	

## Linux安装

## Docker安装（需要指定版本号）

### 拉取镜像

```sh
docker pull elasticsearch:7.5.1
```

![image-20191221213708633](images\es-docker-image.png)

### 运行

```sh
docker network create es-network

docker run --network=es-network -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" --name es elasticsearch:7.5.1
```

### 验证

![](images\es-docker-validate.png)

# 2 Elastic Stack On Docker

## [安装链接](https://www.elastic.co/guide/en/elastic-stack-get-started/current/get-started-docker.html)（注意修改镜像）

