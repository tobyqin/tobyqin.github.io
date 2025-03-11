---
title: MacOXS上快速启动一个ELK
categories: [Tech]
tags: [Tech]
date: 2020-12-16
---

万物都可以用Docker快速启动。

## 快速开始

1. 确保你的 Docker 已经安装完毕。
2. 配置一个 docker 的仓库镜像，任选其一或者自行搜索。

```
https://hub-mirror.c.163.com
https://ngim31fm.mirror.aliyuncs.com
```

3. 只需要一行命令即可。

```bash
$ sudo docker run -p 5601:5601 -p 9200:9200 -p 5044:5044 -it \
  -e MAX_MAP_COUNT=262144 --name elk sebp/elk
```

## 功能验证

如果启动过程没报错，那么可以通过以下地址访问服务。

1. ElasticSearc: http://localhost:9200
2. Kibana: http://localhost:5601

This command publishes the following ports, which are needed for proper operation of the ELK stack:

- 5601 (Kibana web interface).
- 9200 (Elasticsearch JSON interface).
- 5044 (Logstash Beats interface, receives logs from Beats such as Filebeat – see the _[Forwarding logs with Filebeat](https://elk-docker.readthedocs.io/#forwarding-logs-filebeat)_ section).

The image exposes (but does not publish):

- Elasticsearch's transport interface on port 9300. Use the `-p 9300:9300` option with the `docker` command above to publish it. This transport interface is notably used by [Elasticsearch's Java client API](https://www.elastic.co/guide/en/elasticsearch/client/java-api/current/index.html), and to run Elasticsearch in a cluster.
- [Logstash's monitoring API](https://www.elastic.co/guide/en/logstash/current/monitoring-logstash.html) on port 9600. Use the `-p 9600:9600` option with the `docker` command above to publish it.

## 相关文档

1. https://elk-docker.readthedocs.io/

其他操作系统的相关介绍也可以在这里找到。
