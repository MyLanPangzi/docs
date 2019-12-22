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

### 当容器改变时才会重新创建

### 可使用变量定制化环境

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

