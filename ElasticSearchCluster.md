# ElasticSearch Cluster In Docker

本文记录在Docker容器中采用Docker Compose搭建3个ES节点集群的部署过程。

## 1. 新建Docker-Compose根目录

```shell
mkdir -p /opt/es-cluster/{es01,es02,es03}/{data,logs} 
```

- es01-es03：es节点本地映射路径，用作数据卷，映射ES容器中的关键路径
  - data：映射ES数据目录
  - logs：映射ES日志目录

## 2. 编写Docker-Compose配置文件

在根目录下新建docker-compose.yml文件，并写入以下内容：

```yaml
version: '2.2'
services:
  es01:
    image: elastic/elasticsearch:7.9.0
    container_name: es01
    privileged: true
    environment:
      - node.name=es01
      - cluster.name=es-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m" 
      - http.cors.enabled=true
      - http.cors.allow-origin=*
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      # 挂载data
      - ./es01/data:/usr/share/elasticsearch/data
      # 挂载日志文件
      - ./es01/logs:/usr/share/elasticsearch/logs
      # 挂在配置文件
      # - ./es01/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      # - ./es01/jvm.options:/usr/share/elasticsearch/config/jvm.options
    ports:
      - 9200:9200
      - 9300:9300
    networks:
      - elastic    

  es02:
    image: elastic/elasticsearch:7.9.0
    container_name: es02
    privileged: true
    environment:
      - node.name=es02
      - cluster.name=es-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
      - http.cors.enabled=true
      - http.cors.allow-origin=*
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - ./es02/data:/usr/share/elasticsearch/data
      # 挂载日志文件
      - ./es02/logs:/usr/share/elasticsearch/logs
      # 挂在配置文件
      # - ./es02/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      # - ./es02/jvm.options:/usr/share/elasticsearch/config/jvm.options
    ports:
      - 9201:9200
      - 9301:9300
    networks:
      - elastic
  
  es03:
    image: elastic/elasticsearch:7.9.0
    container_name: es03
    privileged: true
    environment:
      - node.name=es03
      - cluster.name=es-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms256m -Xmx256m"
      - http.cors.enabled=true
      - http.cors.allow-origin=*
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - ./es03/data:/usr/share/elasticsearch/data
      # 挂载日志文件
      - ./es03/logs:/usr/share/elasticsearch/logs
      # 挂在配置文件
      # - ./es03/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml
      # - ./es03/jvm.options:/usr/share/elasticsearch/config/jvm.options
    ports:
      - 9202:9200
      - 9302:9300 
    networks:
      - elastic

  kibana:
    image: docker.elastic.co/kibana/kibana:7.9.0
    container_name: kibana
    privileged: true
    ports:
      - 5601:5601
    environment:
      ELASTICSEARCH_URL: http://es01:9200
      ELASTICSEARCH_HOSTS: http://es01:9200
    networks:
      - elastic      

  es-head:
    image: mobz/elasticsearch-head:5
    container_name: es-head
    ports:
      - 9100:9100
    environment:
      ELASTICSEARCH_HOSTS: http://es01:9200
    networks:
      - elastic

  es-cerebro:
    image: lmenezes/cerebro
    container_name: es-cerebro
    ports:
      - 9000:9000
    command:
      - -Dhosts.0.host=http://es01:9200
    networks:
      - elastic
      
volumes:
  es01:
    driver: local
  es02:
    driver: local
  es03:
    driver: local

networks:
  elastic:
    driver: bridge
```

本地目录树形结构如下：

```shell
.
├── docker-compose.yml
├── es01
│   ├── data
│   └── logs
├── es02
│   ├── data
│   └── logs
└── es03
    ├── data
    └── logs
```

## 3. 构建并启动ES集群

在根目录下执行命令：

```shell
docker-compose up -d
```

## 4. ES及其监控组件访问地址

- es-head: http://192.168.32.10:9100/
- kibana: http://192.168.32.10:5601/
- es-cerebro: http://192.168.32.10:9000/