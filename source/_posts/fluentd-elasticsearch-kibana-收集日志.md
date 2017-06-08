---
title: fluentd+elasticsearch+kibana 收集 flask 日志
date: 2017-06-07 16:41:54
tags:
---

部署的机器是ubuntu 14.04 内存 2G

1. 首先是 elasticsearch 安装
  * elasticsearch版本是5.4.1，[官网地址](https://www.elastic.co/downloads/elasticsearch)
  * 需要安装java环境，新版 5.4.1 需要的是jdk1.8，安装步骤如下(ubuntu 14.04)：
    - sudo add-apt-repository ppa:openjdk-r/ppa
    - sudo apt-get update
    - sudo apt-get install openjdk-8-jdk
    - 如果有多个java版本，可能需要切换
      * sudo update-alternatives --config java #查看并选择java的版本
      * sudo update-alternatives --config javac #配置javac的版本
    - java -version #查看java版本
  * 解压文件，并放置在特定的目录，**建议创建一个system用户，用来管理和启动相关的程序，后面kibana也要用到**
  * 使用supervisor启动elasticsearch
    - 配置文件
    ```
    [program:elasticsearch]
    command=/var/log_collecter/elasticsearch-5.4.1/bin/elasticsearch -s -E path.data=/data/es -E indices.fielddata.cache.size=1gb
    directory=/var/log_collecter/elasticsearch-5.4.1
    environment=ES_JAVA_OPTS="-Xms1g -Xmx1536m"
    numprocs=1
    user=log_collecter
    autostart=true
    autorestart=true
    redirect_stderr=true
    stdout_logfile=/var/log/supervisor/%(program_name)s.log
    stdout_logfile_maxbytes=100MB
    stdout_logfile_backups=10
    ```
    - 其中 ES_JAVA_OPTS 的配置主要是机器的内存小才用到的，两个参数均是jvm使用的参数， -Xms配置jvm最小堆内存，-Xmx配置jvm最大堆内存，[详细解释参考elasticsearch文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/heap-size.html)
    - **sudo ln -s /var/log_collecter/elasticsearch-5.4.1/supervisor_es.conf /etc/supervisor/conf.d/, 创建配置文件软连接到 /etc/supervisor/conf.d**
    - sudo supervisorctl update, 更新启动程序
  * 以上elasticsearch和java的安装只是参考，安装的方法有很多种，只要能保证elasticsearch正常运行即可，curl http://127.0.0.1:9200 能正常访问即可

2. fluentd 安装
  * 安装前需要可能需要配置一下系统，[官方安装前说明](http://docs.fluentd.org/v0.12/articles/before-install)
    - 文件描述符限制，ulimit -n 可以查看，最好大于 65536，如果是 1024等较小的数字，修改方法是编辑 /etc/security/limits.conf ，添加下面的配置，重启机器
    ```
    root soft nofile 65536
    root hard nofile 65536
    * soft nofile 65536
    * hard nofile 65536
    ```
    - 网络相关的内核参数的修改，本次没有修改，有需要，请参考[官方安装前说明](http://docs.fluentd.org/v0.12/articles/before-install)
  * 使用官网提供的命令安装(Trusty 版本)，curl -L https://toolbelt.treasuredata.com/sh/install-ubuntu-trusty-td-agent2.sh | sh
  * **查看ubuntu版本命令 lsb_release -a, 详细的安装过程请参考[官网ubuntu安装指导](http://docs.fluentd.org/v0.12/articles/install-by-deb)，本次安装没有踩坑，所以此处较为省略**

3. kibana 安装
  * 下载kibana-5.4.1，注意版本，一定要和之前elasticsearch兼容，[下载网页](https://www.elastic.co/downloads/kibana)
  * 解压文件，并放置在特定的目录，**建议创建一个system用户，用来管理和启动相关的程序**
  * 使用supervisor启动 kibana
  ```
    [program:kibana]
    command=/var/log_collecter/kibana-5.4.1/bin/kibana -e http://127.0.0.1:9200 -q
    directory=/var/log_collecter/kibana-5.4.1
    numprocs=1
    user=log_collecter
    autostart=true
    autorestart=true
    redirect_stderr=true
    stdout_logfile=/var/log/supervisor/%(program_name)s.log
    stdout_logfile_maxbytes=100MB
    stdout_logfile_backups=10
  ```
  * **sudo ln -s /var/log_collecter/kibana-5.4.1/supervisor_kibana.conf /etc/supervisor/conf.d/, 创建配置文件软连接到 /etc/supervisor/conf.d**
  * sudo supervisorctl update, 更新启动程序
  * 同样这里使用supervisor等方式也只是一种启动的方式，像 systemctl 等方式也可以实现相同的作用

  至此，日志收集的工具都已经安装齐全。

4. 修改 fluentd 的配置文件，只收集指定的日志, 建议只绑定到特定的内部地址，不要使用默认的端口号，注释掉原来默认的 @type forward 配置
```
<source>
    @type forward
    bind 10.10.10.10
    port 10000
</source>
<match lifemg.**>
    type elasticsearch
    host 127.0.0.1
    port 9200
    include_tag_key true
    tag_key @log_name
    logstash_format true
    flush_interval 10s
</match>
```

5. flask 相关的配置
  * 安装 fluentd 的第三方包 fluent
  * 配置相关的log
  ```python
    import logging
    import fluent.handler

    #class FilterObject(logging.Filter):
    #    def __init__(self, filterFunc):
    #        self.filterFunc = filterFunc

    #    def filter(self, record):
    #        return self.filterFunc(record)

    logger = logging.getLogger("log_prefix")
    fluentHandler = fluent.handler.FluentHandler(
        "log_prefix",
        host=self.fluent_host,
        port=self.fluent_port
    )
    fluent_format = {
        'host': '%(hostname)s',
        'where': '%(filename)s.%(lineno)d',  #具体到文件、函数
        'type': '%(levelname)s',
        'func': '%(funcName)s',
        'stack_trace': '%(exc_text)s',
        'real_log_name':'%(name)s',
        'process_id':'%(process)d',
    }
    fluentHandler.setFormatter(fluent.handler.FluentRecordFormatter(fluent_format))
    #fluentHandler.addFilter(FilterObject(
    #    lambda x: self.fluent_host and self.fluent_port
    #))
    logger.addHandler(fluentHandler)
    logger.propagate = False
  ```

6. 启动flask程序，打开kibana地址，就可以看到收集到的log

