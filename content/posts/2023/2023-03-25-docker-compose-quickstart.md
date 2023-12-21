---
title: 使用Docker Compose快速部署ElasticSearch
slug: docker-compose-quickstart
date: 2023-03-25
coverImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.2/img/gallery/p/pixabay-polynesia.jpg
thumbnailImagePosition: right
thumbnailImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.2/img/gallery/p/pixabay-polynesia.jpg
categories:
- dev
tags:
- docker
- docker-compose
- elasticsearch
- kibana
---

有时需要在Windows上部署ElasticSearch，那么最方便的方法还是用docker-compose来安装部署。也可以将本文作为快速上手docker-compose的简单案例。

<!--more-->

{{< toc >}}

## 安装 Docker Desktop

参考官方文档，下载安装即可 👉[安装Docker Desktop](https://docs.docker.com/desktop/install/windows-install/)，可能还需要注册一个Docker Hub帐号。

## 创建docker-compose.yml配置文件

可以同时部署ElasticSearch和Kibana，在本地创建一个空目录`D:/elastic`，在该目录下创建一个docker-compose.yml配置文件，内容如下:

```yaml
version: "3.0"
services:
  elastic:
    container_name: es-app
    image: docker.elastic.co/elasticsearch/elasticsearch:8.6.1
    # build: .
    environment:
      - xpack.security.enabled=false
      - "discovery.type=single-node"
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - ELASTIC_USERNAME=elastic
      - ELASTIC_PASSWORD=test
    volumes:
      - "D:/elastic/data:/usr/share/elasticsearch/data"
      - "D:/elastic/logs:/usr/share/elasticsearch/logs"
    restart: always
    networks:
      - esnet
    ports:
      - 9200:9200
  kibana:
    container_name: kibana-app
    image: docker.elastic.co/kibana/kibana:8.6.1
    restart: always
    environment:
      - ELASTICSEARCH_HOSTS=http://es-app:9200
    networks:
      - esnet
    depends_on:
      - elastic
    ports:
      - 5601:5601
networks:
  esnet:
    driver: bridge
```

为了避免报错，在Windows中需要修改`vm.max_map_count`，在Windows的WSL中，执行命令`sudo sysctl -w vm.max_map_count=262144`，参考[这里](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html#_windows_with_docker_desktop_wsl_2_backend)。

在目录`D:/elastic`下，通过命令行执行`docker-compose up`，如果需要多次重新创建docker容器，可以使用命令`docker-compose up --build --force-recreate`来强制重建。

执行完该命令将自动启动容器`es-app`和`kibana-app`。

## 安装ElasticSearch中文分词器插件

在上述命令行窗口依次执行以下命令，
1. 首先通过`docker exec -it es-app /bin/bash`进入容器`es-app`，并且使用`/bin/bash`shell脚本工具；
2. 在容器中的bash命令行中执行第二行命令，安装与上述ElasticSearch相对于的ik分词器版本，可能需要确认，输入`y`回车即可
3. 安装完成需要重新启动容器，先在第二步bash中输入exit退出容器，之后在Windows的命令行中执行`docker restart es-app`命令来重启容器，也可以手动在Docker Desktop界面中重启。

```bash
docker exec -it es-app /bin/bash
./bin/elasticsearch-plugin install https://github.com/medcl/elasticsearch-analysis-ik/releases/download/v8.6.1/elasticsearch-analysis-ik-8.6.1.zip
docker restart es-app
```

## 在ElasticSearch中创建索引、添加数据

在Windows中的浏览器中打开`http://localhost:5601`打开Kibana，进而打开Dev Tools，在界面中执行以下脚本，创建名为`hello`的索引（包含两个field，`greetings`和`name`），及添加一条数据。

```
PUT /hello
{
    "mappings": {
        "properties": {
            "greetings": {
                "type": "keyword"
            },
            "name": {
                "type": "keyword"
            }
        }
    }
}
```

```
PUT hello/_doc/1
{
  "greetings": "hello",
  "name": "World"
}
```

## Docker容器配置网络，外部容器访问

部署完ElasticSearch，如果需要在其他部署了应用的容器中访问这个ElasticSearch服务，那就需要用到`docker network`来配置容器的网络，外部容器如果需要访问上面创建的`es-app`容器，需要跟`es-app`在同一个容器网络中。

```bash
> docker network ls
NETWORK ID     NAME                          DRIVER    SCOPE
2a222daac67e   bridge                        bridge    local
59aaa80c43fe   elastic_esnet                 bridge    local
bc18b55cdc3c   none                          null      local
```

通过命令`docker network ls`可以看到上面我们在`docker-compose.yml`中为容器`es-app`设置的网络`esnet`名称变为`elastic_esnet`，其中`elastic`是project名称，对应上面的目录名称。当我们创建其他容器时，可以将容器网络设为该网络，设置方法下面详细解释。

### 创建Python Flask应用程序

在其他地方创建另一个空目录`D:/hello-es`，我们要创建一个简单的Flask应用程序来访问ElasticSearch，首先在该目录下放一个文件`requirements.txt`，设置容器需要安装的Python软件包，内容如下：

```
flask
gunicorn
elasticsearch
```

创建一个Python文件`app.py`，这个应用很简单，就是通过Python的ElasticSearch模块来访问ElasticSearch服务，可以看到设置的host为`"http://es-app:9200"`，其中`es-app`就是上面部署的容器名，因为我们将配置启动该app的容器与`es-app`在同一虚拟网络中，所以应用程序中可以用容器名作为host名。

```python
from flask import Flask
from elasticsearch import Elasticsearch

app = Flask(__name__)
es = Elasticsearch("http://es-app:9200", basic_auth=('elastic','test'), request_timeout=3600)

@app.route('/')
def hello():
    res = es.search(index="hello", query={"match":{"greetings": "hello"}})
    name = res['hits']['hits'][0]['_source']['name']
    return f'Hello, {name}!'
```

### 使用`docker-compose`创建应用容器，并设置同一虚拟网络

为了创建一个启动上述app的容器，我们还需要在目录下创建一个`Dockerfile`，内容如下：

```Dockerfile
FROM python:3.9-slim-buster
WORKDIR /app
COPY . .
RUN pip install -r requirements.txt
```

再创建一个`docker-compose.yml`，内容如下：

```yml
version: "3"
services:

  app:
    build: .
    command: gunicorn -w 4 -b 0.0.0.0:6000 app:app
    ports:
      - "6000:6000"
    volumes:
      - ./log:/code/log
    networks:
      - appnet

networks:
  appnet:
    name: elastic_esnet
    external: true
```

通过`networks`设置app的网络为`appnet`，其`name`为上面容器中创建的虚拟网络`elastic_esnet`，并且设置`external: true`。这样该容器启动之后，就与`es-app`在同一网络`elastic_esnet`中。该容器的创建和启动方法还是在其目录下执行`docker-compose up`。

通过执行命令`docker network inspect elastic_esnet`可以查看在网络`elastic_esnet`中的所有容器信息。

启动成功后，如果一切正常，在浏览器中输入`http://localhost:6000`，应该得到`Hello, World!`。🥳