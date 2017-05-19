---
title: Dockerfile demo
date: 2017-05-19 16:13:49
tags:
---

一个简单的Dockerfile的demo

```Dockerfile
FROM python:2.7-slim
MAINTAINER XX <XX@yy.com>

RUN apt-get update && \
    apt-get install -y supervisor \
    libmysqlclient-dev unzip libaio1 \
    build-essential libssl-dev libffi-dev curl && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/* \
        /tmp/* \
        /var/tmp/*

ENV MYSQL_DATABASE tmp
ENV MYSQL_USER tmp
ENV MYSQL_PASSWORD tmp
ENV MYSQL_HOST mysql_db
ENV MYSQL_PORT 3306
ENV HOST_IP '127.0.0.1'
ENV HOST_PORT 5000
ENV GIT_BRANCH_NAME 'UNKOWN'
ENV ES_HOST es_db
ENV ES_PORT 9200
ENV REDIS_HOST redis_db
ENV REDIS_PORT 6379
ENV FLASK_ENV docker

WORKDIR /opt/oracle/
ADD instantclient-basic-linux.x64-12.2.0.1.0.zip .
ADD instantclient-sdk-linux.x64-12.2.0.1.0.zip .
RUN unzip instantclient-basic-linux.x64-12.2.0.1.0.zip && \
    unzip instantclient-sdk-linux.x64-12.2.0.1.0.zip && \
    rm instantclient-basic-linux.x64-12.2.0.1.0.zip && \
    rm instantclient-sdk-linux.x64-12.2.0.1.0.zip && \
    cd instantclient_12_2 && \
    ln -s libclntsh.so.12.1 libclntsh.so && \
    ln -s libocci.so.12.1 libocci.so

ENV ORACLE_HOME "/opt/oracle/instantclient_12_2"
ENV DYLD_LIBRARY_PATH "$ORACLE_HOME:$ORACLE_HOME/sdk/oracle/ott:$DYLD_LIBRARY_PATH"
ENV LD_LIBRARY_PATH "$ORACLE_HOME/sdk/include:$LD_LIBRARY_PATH"

RUN curl https://raw.githubusercontent.com/creationix/nvm/v0.29.0/install.sh -o - | bash && \
    export NVM_DIR="/root/.nvm" && . "$NVM_DIR/nvm.sh" && \
    nvm install v0.12.7 && \
    nvm use v0.12.7 && \
    npm config set user 0 && npm config set unsafe-perm true && \
    npm install -g fis && \
    npm install -g fis-postprocessor-require-async && \
    npm install -g  fis-parser-ejs && \
    npm install -g  fis-parser-sass && \
    npm install -g  fis-postpackager-autoload

WORKDIR /opt/lifemg
ADD requirements.txt .
RUN pip --no-cache-dir --disable-pip-version-check install -r requirements.txt

ADD manage.py .
ADD scripts scripts
ADD configs configs
ADD lws lws

RUN ls -l && pybabel compile -d ./lws/translations && \
    export NVM_DIR="/root/.nvm" && . "$NVM_DIR/nvm.sh" && \
    nvm use v0.12.7 && \
    fis release --root ./lws/frontend/  -pcmd ./lws/ && \
    rm -rf $NVM_DIR

# RUN ln -s ./configs/supervisor_docker_app.conf /etc/supervisor/conf.d/supervisor_docker_app.conf

EXPOSE 443
EXPOSE 80

# CMD supervisorctl start lifemg && tail -f /dev/null
CMD gunicorn -w 2 -b 0.0.0.0:80 -k sync --log-level debug manage:app && tail -f /dev/null
```