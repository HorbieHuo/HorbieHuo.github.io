---
title: docker中安装cx-Oracle
date: 2017-05-17 08:31:31
tags:
---

1. 下载下面两个压缩文件 [地址](http://www.oracle.com/technetwork/topics/linuxx86-64soft-092277.html)
  * instantclient-basic-linux.x64-12.2.0.1.0.zip
  * instantclient-sdk-linux.x64-12.2.0.1.0.zip 

2. 将下载的两个压缩包放至Dockerfile所在目录，或者镜像构建时可以访问的其他目录

3. docker 构建时使用的命令(部分)
```dockerfile
RUN apt-get update && \
    apt-get install -y  unzip libaio1 && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* \
        /tmp/* \
        /var/tmp/*
WORKDIR /opt/oracle/
ADD instantclient-basic-linux.x64-12.2.0.1.0.zip .
ADD instantclient-sdk-linux.x64-12.2.0.1.0.zip .
RUN unzip instantclient-basic-linux.x64-12.2.0.1.0.zip && \
    unzip instantclient-sdk-linux.x64-12.2.0.1.0.zip && \
    cd instantclient_12_2 && \
    ln -s libclntsh.so.12.1 libclntsh.so && \
    ln -s libocci.so.12.1 libocci.so

ENV ORACLE_HOME "/opt/oracle/instantclient_12_2"
ENV DYLD_LIBRARY_PATH "$ORACLE_HOME:$ORACLE_HOME/sdk/oracle/ott:$DYLD_LIBRARY_PATH"
ENV LD_LIBRARY_PATH "$ORACLE_HOME/sdk/include:$LD_LIBRARY_PATH"
```

4. 至此，在使用pip安装python中的cx-Oracle就正常了

** 在oracle的文件下载页面也有相关设置的说明 **