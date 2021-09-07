---
layout:     post
title:      ELK系统搭建并与spring boot集成
subtitle:   ELK系统搭建并与spring boot集成
date:       2021-09-06
author:     LIWC
header-img: img/post-bg-desk.jpg
catalog: true
tags:
    - ELK、springboot
---

### 一、ELK含义及其作用:

##### 我们知道如果一旦使用spring boo启动jar包的话，启动时一般会使用nohup方式启动，而这种方式通常会将日志放到.out文件中，而当.out文件过大时，查看日志就非常麻烦了，那么一般我们会使用ELK系统将日志存储在Elasticsearch中，ELK为以下三者单词首字母，除了有ELK还有EFK。这里主要介绍ELK日志系统的搭建

- Elasticsearch:用于存储收集到的日志信息；
- Logstash:用于收集日志，SpringBoot应用整合了Logstash以后会把日志发送给ogstash,Logstash再把日志转发给Elasticsearch；
- Kibana:通过Web端的可视化界面来查看日志。

### 二、开始搭建：

- 使用liunx，创建目录
  `mkdir /usr/local/docker/elk`

- 创建`docker-compose.yml`文件

- 编辑docker-compose.yml文件

  ```yml
  version: '3'
  services:
  elasticsearch:
    image: elasticsearch:7.7.0
    container_name: elasticsearch
    environment:
      - "cluster.name=elasticsearch" #设置集群名称为elasticsearch
      - "discovery.type=single-node" #以单一节点模式启动
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m" #设置使用jvm内存大小
    volumes:
      - /usr/local/docker/elk/elasticsearch/plugins:/usr/share/elasticsearch/plugins #插件文件挂载
      - /usr/local/docker/elk/elasticsearch/data:/usr/share/elasticsearch/data #数据文件挂载
    ports:
      - 9200:9200
  kibana:
    image: kibana:7.7.0
    container_name: kibana
    depends_on:
      - elasticsearch #kibana在elasticsearch启动之后再启动
    environment:
      - ELASTICSEARCH_URL=http://elasticsearch:9200 #设置访问elasticsearch的地址
    ports:
      - 5601:5601
  logstash:
    image: logstash:7.7.0
    container_name: logstash
    volumes:
      - /usr/local/docker/elk/logstash/logstash-springboot.conf:/usr/share/logstash/pipeline/logstash.conf #挂载logstash的配置文件
    depends_on:
      - elasticsearch #kibana在elasticsearch启动之后再启动
    links:
      - elasticsearch:es #可以用es这个域名访问elasticsearch服务
    ports:
      - 4560:4560
  ```

- 创建 `logstash`目录
   `mkdir /usr/local/docker/elk/logstash`

- 进入`logstash` 目录
   `cd logstash`

- 创建`logstash-springboot.conf`配置文件

- 编辑`logstash-springboot.conf`文件

  ```shell
  input {
    tcp {
      mode => "server"
      host => "0.0.0.0"//ELK地址
      port => 4560
      codec => json_lines
    }
  }
  output {
    elasticsearch {
      hosts => "es:9200"
      index => "springboot-logstash-%{+YYYY.MM.dd}"
    }
  }
  ```

- 启动项目:
  `docker-compose up`

- 访问`Kibana`(访问地址：`http://ip:5601`)

- 启动成功后`logstash`中安装`json_lines`插件

```shell
  # 进入logstash容器
  docker exec -it logstash /bin/bash
  # 进入bin目录
  cd /bin/
  # 安装插件
  logstash-plugin install logstash-codec-json_lines
  # 退出容器
  exit
  # 重启logstash服务
  docker restart logstash
```

###   三、`kibana` 汉化

交互式进入`kibana`容器

- `docker exec -it kibana /bin/bash`

在该镜像中编辑该配置文件

- `vi /opt/kibana/config/kibana.yml`

修改该文件 在文件最后加上一行配置

- `i18n.locale: zh-CN`
- 注意：`zhe-CN`和`:`号之间必须有个空格，否则kibana无法启动

重启，即可看到访问的 `kibana`已汉化。

![](/img/elk1.jpg)



### 四、`SpringBoot`集成`Logstash`

- 在项目的`pom.xml`中添加`logstash-logback-encoder`依赖

```xml
<!--集成logstash-->
<dependency>
    <groupId>net.logstash.logback</groupId>
    <artifactId>logstash-logback-encoder</artifactId>
    <version>4.11</version>
</dependency>
```

- 新建日志配置文件 `logback-spring.xml` 并添加配置文件`logback-spring.xml让logback`的日志输出到`logstash`

> 注意appender节点下的destination需要改成自己的logstash服务地址，比如我的是：39.96.74.91:4560。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration>
<configuration>
    <include resource="org/springframework/boot/logging/logback/defaults.xml"/>
    <include resource="org/springframework/boot/logging/logback/console-appender.xml"/>
    <!--应用名称-->
    <property name="APP_NAME" value="demo-elk"/>
    <!--日志文件保存路径-->
    <property name="LOG_FILE_PATH" value="${LOG_FILE:-${LOG_PATH:-${LOG_TEMP:-${java.io.tmpdir:-/tmp}}}/logs}"/>
    <contextName>${APP_NAME}</contextName>
    <!--每天记录日志到文件appender-->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_FILE_PATH}/${APP_NAME}-%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder>
            <pattern>${FILE_LOG_PATTERN}</pattern>
        </encoder>
    </appender>
    <!--输出到logstash的appender-->
    <appender name="LOGSTASH" class="net.logstash.logback.appender.LogstashTcpSocketAppender">
        <!--可以访问的logstash日志收集端口-->
        <destination>192.168.118.131:4560</destination>
        <encoder charset="UTF-8" class="net.logstash.logback.encoder.LogstashEncoder"/>
    </appender>
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
        <appender-ref ref="LOGSTASH"/>
    </root>
</configuration>
```

- 启动SpringBoot应用

### 五、使用 kibana 查看日志信息

- 创建索引

![img](E:\liwc\feiwanglantian.github.io\img\elk2.webp)



![img](E:\liwc\feiwanglantian.github.io\img\elk3.webp)



![img](E:\liwc\feiwanglantian.github.io\img\elk4.webp)



![img](E:\liwc\feiwanglantian.github.io\img\elk5.webp)



- 查看日志收集

![img](E:\liwc\feiwanglantian.github.io\img\elk6.webp)

### 六、总结

当spring boot启动日志特别多的时候，我们可以使用elk的方式来查看日志，避免查看.out文件时间过长，导致无法快速定位问题。

### 本文转载于简书

 作者：onnoA
 链接：https://www.jianshu.com/p/203447e25ad5
 来源：简书
 著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。

  