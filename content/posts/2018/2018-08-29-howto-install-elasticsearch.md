---
title: "How to Install and Configure Elasticsearch Clusters"
date: 2018-08-29T21:45:00+08:00
coverImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.2/img/gallery/p/pixabay-scotland.jpg
thumbnailImagePosition: left
thumbnailImage: //cdn.jsdelivr.net/gh/lijqhs/cdn@1.2/img/gallery/p/pixabay-scotland.jpg
categories:
- dev
tags:
- elasticsearch
- kibana
---



Elasticsearch is a highly scalable open-source full-text search and analytics engine based on Lucene. This post focus on how to setup an Elasticsearch cluster. <!--more-->I have three Linux servers to build an Elasticsearch cluster. I referred to many materials on the web to get a clear configuration on how to setup an Elasticsearch cluster but many articles were too old with a lot of outdated parameters. The best reference is [documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/setup.html) on official website of Elasticsearch. Here I will get you a concise and informative reference. Following these steps the cluster will be easily setup. 

{{< toc >}}

Elasticsearch here is based on version 6.2.4.

## Install Java and Elasticsearch

### Create a user for Elasticsearch

Elasticsearch is not allowed to run by root user. Elasticsearch will execute scripts input by users so for the sake of system security starting Elasticsearch by root is banned. Create a user `elastic` for Elasticsearch within group `elastic`. A directory is also created in `/opt` where Elasticsearch will be installed.

```bash
groupadd elastic
useradd elastic -g elastic
mkdir /opt/elastic
chown -R elastic:elastic /opt/elastic
```

### Update the Installation packages

It is required to install Java before Elasticsearch. A supported LTS version of Java version 1.8.0_131 or a later version in the Java 8 release series is preferred. Before continuing, download [Java](http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html) and [Elasticsearch](https://www.elastic.co/downloads/elasticsearch) and [Kibana](https://www.elastic.co/downloads/kibana). Upload three file onto every server you want them installed. Suppose you have the following three file:

- elasticsearch-6.2.4.tar.gz
- jdk-8u171-linux-x64.tar.gz
- kibana-6.2.4-linux-x86_64.tar.gz

I used to put files on the user home directory here for user elastic is `/home/elastic`

### Install Java

Check if Java is already installed in bash mode:

```bash
which java
```

If nothing replied go on the installation or you must examine the Java version can satisfy the requirement. If not, I suggest to reinstall Java. It is better to install Java in `/usr` and Elasticsearch in `/opt`.

```bash
mkdir /usr/java/
tar -zxvf /home/elastic/jdk-8u171-linux-x64.tar.gz -C /usr/java/
```

### Modify PATH

```bash
vi /etc/profile
```

Append the following four lines in the end of profile.

```bash
export JAVA_HOME=/usr/java/jdk1.8.0_171
export JRE_HOME=${JAVA_HOME}/jre
export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
export  PATH=${JAVA_HOME}/bin:$PATH
```

Save the file and exit edit. Make the setting effective by command:

```bash
source /etc/profile
```

### Install Elasticsearch

First give a password for elastic for the next login (optional since you can switch to elastic from root) and unpack the file in `/opt/elastic`.

```bash
passwd elastic
su elastic
cd /home/elastic
tar -zxvf elasticsearch-6.2.4.tar.gz -C /opt/elastic/
```

## Configure Elasticsearch

### elasticsearch.yml

The first part is trivial. To make a reliable cluster for Elasticsearch some parameters should be taken carefully. Come back to root user and create directories `data` and `logs` for Elasticsearch which are usually put on the directories away from the installation directory of Elasticsearch.

```bash
exit
mkdir -p /var/elasticsearch/data
mkdir -p /var/elasticsearch/logs
chown -R elastic:elastic /var/elasticsearch
su elastic
cd /opt/elastic/elasticsearch-6.2.4/
vi config/elasticsearch.yml
```

The most important part is Elasticsearch configuration file `config/elasticsearch.yml`.  Replace `192.168.0.1`, `192.168.0.2` and `192.168.0.3` by your three server IPs and each `node.name` set to be `ES1`, `ES2` and `ES3` or any strings you like.

```yaml
#======================== Elasticsearch Configuration =========================
#
# NOTE: Elasticsearch comes with reasonable defaults for most settings.
#       Before you set out to tweak and tune the configuration, make sure you
#       understand what are you trying to accomplish and the consequences.
#
# The primary way of configuring a node is via this file. This template lists
# the most important settings you may want to configure for a production cluster.
#
# Please consult the documentation for further information on configuration options:
# https://www.elastic.co/guide/en/elasticsearch/reference/index.html
#
# ————————————————— Cluster —————————————————
#
# Use a descriptive name for your cluster:
#
cluster.name: ES_Cluster
node.master: true
node.data: true

#
# —————————————————— Node ——————————————————
#
# Use a descriptive name for the node:
#
node.name: “ES1”
#
# Add custom attributes to the node:
#
#node.attr.rack: r1
#
# ————————————————— Paths ——————————————————
#
# Path to directory where to store the data (separate multiple locations by comma):
#
path.data: /var/elasticsearch/data
#
# Path to log files:
#
path.logs: /var/elasticsearch/logs
#
# ————————————————— Memory —————————————————
#
# Lock the memory on startup:
#
bootstrap.memory_lock: false
bootstrap.system_call_filter: false

#
# Make sure that the heap size is set to about half the memory available
# on the system and that the owner of the process is allowed to use this
# limit.
#
# Elasticsearch performs poorly when the system is swapping the memory.
#
# ————————————————— Network —————————————————
#
# Set the bind address to a specific IP (IPv4 or IPv6):
#
network.host: 192.168.0.1
#
# Set a custom port for HTTP:
#
http.port: 9200
#
# For more information, consult the network module documentation.
#
# ———————————————— Discovery —————————————————
#
# Pass an initial list of hosts to perform discovery when new node is started:
# The default list of hosts is [“127.0.0.1”, “[::1]”]
#
discovery.zen.ping.unicast.hosts: [“192.168.0.1”, “192.168.0.2”, “192.168.0.3”]
#
# Prevent the “split brain” by configuring the majority of nodes (total number of master-eligible nodes / 2 + 1):
#
discovery.zen.minimum_master_nodes: 2
#
# For more information, consult the zen discovery module documentation.
#
# ————————————————— Gateway —————————————————
#
# Block initial recovery after a full cluster restart until N nodes are started:
#
#gateway.recover_after_nodes: 3
#
# For more information, consult the gateway module documentation.
#
# ————————————————— Various —————————————————
#
# Require explicit names when deleting indices:
#
#action.destructive_requires_name: true
```

**Note:** Prevent the “split brain” by configuring the majority of nodes (total number of master-eligible nodes / 2 + 1). In this case it is 2.

### jvm.options

By default, Elasticsearch tells the JVM to use a heap with a minimum and maximum size of 1 GB. When moving to production, it is important to configure heap size to ensure that Elasticsearch has enough heap available. [See: setting heap size](https://www.elastic.co/guide/en/elasticsearch/reference/6.2/heap-size.html). To make things easier, let’s skip this step.

### Important system configuration

Switch to root.

```bash
vi /etc/security/limits.conf
```

Add these lines:

```bash
* soft nofile 65536
* hard nofile 131072
* soft nproc 4096
* hard nproc 4096
* soft memlock unlimited
* hard memlock unlimited
```

Save and edit `/etc/security/limits.d/90-nproc.conf`

```bash
cp /etc/security/limits.d/90-nproc.conf /etc/security/limits.d/90-nproc.conf_bak
vi /etc/security/limits.d/90-nproc.conf
```

Add the same content:

```bash
* soft nofile 65536
* hard nofile 131072
* soft nproc 4096
* hard nproc 4096
* soft memlock unlimited
* hard memlock unlimited
```

Logout and relogon as root. Edit `/etc/sysctl.conf` and append `vm.max_map_count=262144`.

```bash
vi /etc/sysctl.conf
vm.max_map_count=262144
```

Make setting effective:

```bash
sudo sysctl -p /etc/sysctl.conf
```

Do the above settings the same to the other servers except `network.host`. After all those done all you need is to start Elasticsearch one by one.

## Start Elasticsearch

Change user to elastic:

```bash
cd /opt/elastic/elasticsearch-6.2.4/bin
./elasticsearch -d
```

Open browser on your desktop and refer to the address `http://192.168.0.1:9200` to check whether it works.

## One more thing: Kibana

> Kibana is an open source analytics and visualization platform designed to work with Elasticsearch. You use Kibana to search, view, and interact with data stored in Elasticsearch indices.

Logon as user elastic and change directory to `/home/elastic`.

```bash
tar –zxvf kibana-6.2.4-linux-x86_64.tar.gz -C /opt/elastic/
```

In the kibana configuration file `/opt/elastic/kibana-6.2.4-linux-x86_64/config/kibana.yml` three parameters should be replaced:

```yaml
server.port:5601
server.host:192.168.0.1
elasticsearch.url:192.168.0.1:9200
```

Now you can start kibana on the background.

```bash
nohup ./kibana &
```

Open kibana by input `http://192.168.0.1:5601` in the browser.
![kibana](https://www.elastic.co/cn/assets/bltd5ae161f013bea14/5.5-console-80pct-generic-rgb.jpg)

Read [Elastic documentation](https://www.elastic.co/guide/index.html) for further reference or watch [Getting Started video](https://www.elastic.co/webinars/getting-started-elasticsearch?baymax=default&elektra=docs&storm=top-video).