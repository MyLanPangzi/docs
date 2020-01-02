# Docker

## Docker Compose

### 特性

#### 单主机多个隔离环境

使用项目名进行隔离。

1. 开发环境：单环境，多拷贝，每个特性分支一个拷贝

2. CI环境：每个构建携带一个版本号

3. 共享环境或开发环境：阻止不同的使用同名服务，互不干扰
4. -p 选项改变项目名，COMPOSE_PROJECT_NAME环境变量改变项目名

#### 容器创建时保留卷数据

保留服务中所有的卷，每次运行新容器会拷贝旧容器的数据卷

#### 当容器改变时才会重新创建

#### 可使用变量定制化环境

## Docker Compose Get Started

### 1.设置，初始化项目

$ mkdir composetest
$ cd composetest
app.py

``` python
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)


def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)


@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```

requirements.txt

```txt
flask
redis
```

### 2.创建Dockerfile

Dockerfile

```Dockerfile
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP app.py
ENV FLASK_RUN_HOST 0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
COPY . .
CMD ["flask", "run"]
```

### 3.编写docker-compose

docker-compose.yml

```yml
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
  redis:
    image: "redis:alpine"
```

### 4.构建应用程序

```bash
docker-compose up
curl -XGET 'http://localhost:5000'
#'Hello World! I have been seen 1 times.'
curl -XGET 'http://localhost:5000'
#'Hello World! I have been seen 2 times.'
docker-compose down
```

### 5.添加卷

```yml
version: '3'
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
    environment:
      FLASK_ENV: development
  redis:
    image: "redis:alpine"
```

### 6.重新构建

```bash
docker-compose up
curl -XGET 'http://localhost:5000'
#'Hello World! I have been seen 3 times.'
```

### 7.实时更新代码

```python
return 'Hello from Docker! I have been seen {} times.\n'.format(count)
```

```bash
curl -XGET 'http://localhost:5000'
#'Hello World! I have been seen 4 times.'
```

### 8.查看服务

```bash
docker-compose up -d
docker-compose run web env
docker-compose stop
docker-compose down --volumes
```

## Configure networking

### Networking overview

#### Bridge

最常使用，单主机多容器通讯。

#### Host

多容器共享主机网络时使用，网络不隔离，但隔离其他方面。

#### Overlay

跨主机通讯，swarm，K8S部署多服务时。

#### Macvlan

使用物理网络通讯，每个容器会绑定一个Mac地址。

#### None

什么网络都没有。

### Use bridge networks

1. 用户自定义网络在容器化应用间提供了更好的隔离以及互操作性。
2. 自定义桥接网络提供了自动DNS解决方案，容器间可以通过容器名进行通讯。
3. 容器可以随时从桥接网络中脱离或加入。
4. 每个自定义桥接网络创建了一个可配置的桥接网络。
5. Docker自带的默认桥接网络可以在容器间共享环境变量。
   1. 多容器间可通过Volume进行数据共享。
   2. 多容器间可以通过docker-compose共享变量。
   3. 可使用K8S，swarm共享密钥以及配置。
6. 自定义桥接网络命令![image-20191225211808941](images\docker-network.png)

## Swarm

### Create a swarm

```shell
docker swarm init --advertise-addr <MANAGER-IP>
docker node ls
```

### Add nodes to the swarm

```shell
docker swarm join-token worker
docker swarm join \
  --token  SWMTKN-1-49nj1cmql0jkz5s954yi3oex3nedyz0fb0xx14ie39trti4wxv-8vxv8rssmk743ojnwacrr2e7c \
  192.168.99.100:2377
  
```

### Deploy a service to the swarm

```shell
 docker service create --replicas 1 --name helloworld alpine ping docker.com
 docker service ls
```

### Inspect a service on the swarm

```shell
docker service inspect --pretty helloworld
docker service ps helloworld
docker ps
```

### Scale the service in the swarm

```shell
docker service scale <SERVICE-ID>=<NUMBER-OF-TASKS>
docker service ps helloworld
docker ps
```

### Delete the service running on the swarm

```shell
docker service rm helloworld
docker service inspect helloworld
docker ps
```

### Apply rolling updates to a service

```shell
docker service create \
  --replicas 3 \
  --name redis \
  --update-delay 10s \
  redis:3.0.6

docker service inspect --pretty redis
docker service update --image redis:3.0.7 redis
#docker service inspect --pretty redis
docker service ps redis
```

### Drain a node on the swarm（下线一个节点）

```shell
docker node ls
docker node update --avaliability drain node-id
docker node inspect node-id
docker service ps redis
docker node update --avaliability active node-id
docker node inspect node-id
docker service scale redis=4
```

### Use swarm mode routing mesh

```shell
docker service create --name nginx --publish published=80,target=80 --replicas 2 nginx
docker service ls
docker service ps nginx
```

#### Bypass the routing mesh

```shell
docker service create --name dns-cache \
  --publish published=53,target=53,protocol=udp,mode=host \
  --mode global \
  dns-cache
```

