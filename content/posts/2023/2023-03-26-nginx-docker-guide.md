---
title: 使用Docker配置启动Nginx
slug: nginx-docker-guide
date: 2023-03-26
coverImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.2/img/gallery/p/pixabay-italy.jpg
thumbnailImagePosition: right
thumbnailImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.2/img/gallery/p/pixabay-italy.jpg
categories:
- dev
tags:
- docker
- nginx
---


Nginx（发音为 "engine x"）是一个高性能的Web服务器和反向代理服务器，也可以作为一个负载平衡器、HTTP缓存和邮件代理服务器。它被设计用来处理大量的并发连接，而内存使用量却很低，这使得它成为高流量网站的热门选择。

<!--more-->

{{< toc >}}

## Nginx是什么

Nginx可以执行各种任务，包括：

- 提供静态和动态内容： Nginx可以提供静态文件，如HTML、CSS和图像，以及由Web应用程序生成的动态内容。
- 反向代理： Nginx可以作为一个反向代理，根据URL模式或其他标准将HTTP请求转发给后端服务器。
- 负载平衡： Nginx可以将传入的流量分配到多个后端服务器，以提高性能和可靠性。
- 缓存：Nginx可以将经常访问的内容缓存在内存或磁盘上，以减少后端服务器的负载，提高响应时间。
- SSL终止： Nginx可以处理SSL加密和解密，使后端服务器专注于服务内容而不是加密。
- URL重写： Nginx可以重写URL，改变传入请求的路径或查询字符串，这对于实现重定向或修改请求以符合后端服务器的要求非常有用。

总的来说，Nginx是一个多功能的、强大的网络服务器，可以处理与服务、缓存和代理网络内容有关的各种任务。对于需要快速、可靠和可扩展的基础设施的高流量网站，它是一个受欢迎的选择。


## 使用Docker配置Nginx

### 静态文件服务器的例子

用Docker配置Nginx，为静态HTML文件提供服务，可以按照以下步骤进行：

创建一个`Dockerfile`： 

```dockerfile
FROM nginx:latest
COPY nginx.conf /etc/nginx/conf.d/default.conf
COPY website_files /usr/share/nginx/html
```

在这个例子中，Dockerfile从最新版本的官方Nginx镜像开始（FROM nginx:fresh），并复制Nginx配置文件（nginx.conf）和网站文件（website_files）到容器中的各自位置。

创建一个Nginx配置文件： 

```conf
server {
    listen 80；
    server_name example.com；
    root /usr/share/nginx/html；
    index index.html；

    location / {
        try_files $uri $uri/ =404；
    }
}
```

在这个例子中，server块指定Nginx应该在80端口监听对example.com域名的请求，从/usr/share/nginx/html目录提供内容，并使用index.html作为默认索引文件。位置块指定了应该如何处理请求。

构建Docker镜像： 使用docker build命令建立Docker镜像。

```bash
docker build -t my-nginx .
```

在这个例子中，`-t`选项指定了Docker镜像的名称和标签（my-nginx），而`.`则指定了构建环境（即包含`Dockerfile`的目录）。

启动Docker容器： 使用docker run命令启动Docker容器，并将容器的80端口映射到主机的80端口。

```bash
docker run -d -p 80:80 --name my-nginx-container my-nginx
```

在这个例子中，`-d`选项指定容器应该以分离模式运行，`-p`选项将容器中的80端口映射到主机上的80端口，而`--name`选项则指定了容器的名称（my-nginx-container）。

### Nginx的配置文件

Nginx配置文件的语法是基于一系列的指令(directives)和块(blocks)。指令用于定义Nginx的行为，而块则用于将相关的指令组合在一起。该语法使用大括号来定义块，使用分号来分隔指令。

在用Docker配置Nginx时，有两个主要的配置文件可以使用： `/etc/nginx/nginx.conf`和`/etc/nginx/conf.d/default.conf`。

- `/etc/nginx/nginx.conf`文件是Nginx的主配置文件，它包括核心配置设置，如工作进程、事件模型和日志。该文件用于Nginx的整体配置。
- `/etc/nginx/conf.d/default.conf`文件是一个额外的配置文件，可以用来为特定的应用程序或网站定义`server blocks`和`location blocks`。该文件使用`include`指令被包含在主配置文件中。

当用Docker配置Nginx时，通常使用`/etc/nginx/conf.d/default.conf`文件，为Docker容器中运行的应用程序或网站定义`server blocks`和`location blocks`。这个文件可以作为一个`volume`装载在容器中，以方便更新和更改，而不需要新建Docker镜像。

通常有几个配置文件可以用来配置Nginx，包括：
- `/etc/nginx/nginx.conf`： 这是Nginx的主要配置文件，包括核心配置设置，如工作进程、事件模型和日志。
- `/etc/nginx/conf.d/*`： 该目录包含Nginx的其他配置文件。这些文件可以用来为特定的应用程序或网站定义`server blocks`和`location blocks`。
- `/etc/nginx/sites-available/*` 和 `/etc/nginx/sites-enabled/*`： 这些目录与主配置文件中的include指令一起使用，以启用或禁用特定的网站配置。
- `/etc/nginx/mime.type`： 该文件定义了Nginx将提供的MIME类型。

{{< alert info >}}
[媒体类型](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Basics_of_HTTP/MIME_types)（通常称为 Multipurpose Internet Mail Extensions 或 MIME 类型）是一种标准，用来表示文档、文件或字节流的性质和格式。它在[IETF RFC 6838](https://www.rfc-editor.org/rfc/rfc6838)中进行了定义和标准化。

MIME消息可以包含文本、图像、音频、视频和其他特定应用的数据。浏览器通常使用 MIME 类型（而不是文件扩展名）来确定如何处理 URL，因此 Web 服务器在响应头中添加正确的 MIME 类型非常重要。如果配置不正确，浏览器可能会曲解文件内容，网站将无法正常工作，并且下载的文件也会被错误处理。MIME 类型对大小写不敏感，但是传统写法都是小写。通用结构为`type/subtype`，如`text/css`。
{{< /alert >}}


## 使用Nginx配置负载平衡

负载平衡是将进入的网络流量分配到多个服务器上的过程，以避免任何单一服务器过载，并确保每个服务器被充分利用。负载平衡可用于提高网络应用程序、数据库和其他网络服务的性能、可用性和可扩展性。负载平衡通常涉及一个负载平衡器，它位于客户端和后端服务器之间，并根据一组规则或算法将传入的请求引导到可用的服务器之一。负载平衡器还可以执行其他功能，如SSL终止、会话持久性和健康检查，以确保后端服务器的可用性和可靠性。

负载平衡可以使用硬件负载平衡器、软件负载平衡器或两者的组合来实现。硬件负载平衡器是专门为高性能和可扩展性而设计的设备，而软件负载平衡器可以部署在商品硬件或虚拟化环境中，提供更大的灵活性和易于管理。负载平衡是许多现代网络架构的一个重要组成部分，被大型网络应用程序、电子商务网站和基于云的服务用来处理每秒数百万的请求，并确保高可用性和性能。

### Nginx负载平衡原理

Nginx可以通过充当反向代理服务器，将传入的请求分配到多个后端服务器上，从而进行负载平衡。Nginx的负载平衡功能是通过上游模块实现的，它允许你定义一组后端服务器，并指定传入的请求如何在它们之间分配。

下面是Nginx执行负载平衡的基本步骤：

- Nginx收到一个来自客户端的传入请求。
- Nginx将该请求转发到上游模块中定义的一个后台服务器。
- Nginx接收来自后端服务器的响应，并将其返回给客户端。

![load balancing diagram](https://www.nginx.com/wp-content/uploads/2014/07/what-is-load-balancing-diagram-NGINX-640x324.png)

Nginx支持多种负载平衡算法，包括轮流、最少连接、IP散列和随机。这些算法可以在上游模块中配置，根据服务器可用性、服务器负载和客户端IP地址等因素，以不同的方式分配进入的请求。

{{< alert success no-icon >}}
Nginx最常用的一些负载平衡算法：

- 轮流(Round-robin)：在轮流负载平衡中，Nginx将传入的请求以轮流的方式平均分配到所有后端服务器。
- 最少连接(Least connections)： 在最少连接的负载平衡中，Nginx将传入的请求路由到活动连接最少的后端服务器。
- IP散列(IP hash)： 在IP哈希负载平衡中，Nginx根据客户的IP地址，将传入的请求路由到后端服务器。这确保来自同一个客户的后续请求总是被发送到同一个后端服务器。
- 随机(Random)： 在随机负载平衡中，Nginx将传入的请求路由到一个随机选择的后端服务器。
{{< /alert >}}

不同的负载平衡算法可能更适合于不同的用例，这取决于后端服务器的数量、工作负载的类型以及所需的容错和性能水平等因素。

要配置Nginx的负载平衡，可以在nginx.conf文件中使用upstream指令。例如

```conf
upstream backend {
  server backend1:8080；
  server backend2:8080；
  server backend3:8080；
}
```

### 使用Docker Compose配置Nginx负载平衡实例

以下是Nginx实现负载平衡的简单实例，通过Flask启动多个应用在不同端口上，Nginx监听7000端口，实现对该应用不同端口的调用。


Flask App代码（为了展示每次刷新页面Nginx调用了不同的Flask应用，特别设置了不同`app{1-4}.py`以示区别用于测试）：


{{< tabbed-codeblock hello_nginx >}}
    <!-- tab py -->
from flask import Flask

app = Flask(__name__)

@app.route('/')
def hello_world():
    return 'Hello, Nginx! 1'
    <!-- endtab -->
    <!-- tab conf -->
upstream backendserver {
    server 127.0.0.1:7001;
    server 127.0.0.1:7002;
    server 127.0.0.1:7003;
    server 127.0.0.1:7004;
}

server {
    listen 7000;

    location / {
        proxy_pass http://backendserver;
    }
}
    <!-- endtab -->
    <!-- tab yml -->
version: "3"
services:

  nginx:
    image: nginx:latest
    ports:
      - "7000:7000"
      - "9000:9000"
    volumes:
      - ./conf/app.nginx.conf:/etc/nginx/conf.d/default.conf
      - ./html:/usr/share/nginx/html
    depends_on:
      - web
      
  web:
    build:
      context: .
      dockerfile: ./conf/app.dockerfile
    image: hello_nginx_web
    command: gunicorn -w 4 -b 0.0.0.0:7001 --chdir ./web app1:app
    ports:
      - "7001:7001"
      
  web2:
    image: hello_nginx_web
    command: gunicorn -w 4 -b 0.0.0.0:7002 --chdir ./web app2:app
    ports:
      - "7002:7002"
    depends_on:
      - web

  web3:
    image: hello_nginx_web
    command: gunicorn -w 4 -b 0.0.0.0:7003 --chdir ./web app3:app
    ports:
      - "7003:7003"
    depends_on:
      - web

  web4:
    image: hello_nginx_web
    command: gunicorn -w 4 -b 0.0.0.0:7004 --chdir ./web app4:app
    ports:
      - "7004:7004"
    depends_on:
      - web
    <!-- endtab -->
{{< /tabbed-codeblock >}}

完整的代码请访问[GitHub repo](https://github.com/lijqhs/hello-anything/tree/main/hello-nginx).