---
title: python 使用 logging
date: 2017-06-01 12:12:53
tags:
---

首先是json格式的配置文件, 其中配置的有 DatagramHandler ，udp形式的日志
```json
{
    "version": 1,
    "disable_existing_loggers": false,
    "formatters": {
        "simple": {
            "format": "%(asctime)s - %(name)s - %(levelname)s - %(message)s"
        }
    },

    "handlers": {
        "console": {
            "class": "logging.StreamHandler",
            "level": "DEBUG",
            "formatter": "simple",
            "stream": "ext://sys.stdout"
        },

        "info_file_handler": {
            "class": "logging.handlers.RotatingFileHandler",
            "level": "INFO",
            "formatter": "simple",
            "filename": "info.log",
            "maxBytes": 10485760,
            "backupCount": 20,
            "encoding": "utf8"
        },

        "error_file_handler": {
            "class": "logging.handlers.RotatingFileHandler",
            "level": "ERROR",
            "formatter": "simple",
            "filename": "errors.log",
            "maxBytes": 10485760,
            "backupCount": 20,
            "encoding": "utf8"
        },

        "udp_handler": {
            "class": "logging.handlers.DatagramHandler",
            "level": "INFO",
            "formatter": "simple",
            "host": "localhost",
            "port": 11111
        }
    },

    "loggers": {
        "lifemg": {
            "level": "INFO",
            "handlers": ["udp_handler"],
            "propagate": "no"
        }
    },

    "root": {
        "level": "INFO",
        "handlers": ["console"]
    }
}
```

程序中配置log，并使用
```python
import json
import logging
import logging.config
import os

def setup_logging(
    default_path='logging.json', 
    default_level=logging.INFO,
    env_key='LOG_CFG'
):
    """Setup logging configuration

    """
    path = default_path
    value = os.getenv(env_key, None)
    if value:
        path = value
    if os.path.exists(path):
        with open(path, 'rt') as f:
            config = json.load(f)
        logging.config.dictConfig(config)
    else:
        logging.basicConfig(level=default_level)

if __name__ == "__main__":
    setup_logging()
    logger = logging.getLogger("lifemg")
    logger.debug("this is a debugmsg!")
    logger.info("this is a infomsg!")
    logger.warn("this is a warn msg!")
    logger.error("this is a error msg!")
    logger.critical("this is a critical msg!")
```

udp的服务端接受log, 只是接收，并未做任何处理
```python
import socket  

if __name__ == "__main__":
    address = ('localhost', 11111)  
    s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)  
    s.bind(address)  
    print "start udp: localhost:11111"
    while True:
        data, addr = s.recvfrom(2048)
        if not data:  
            print "client has exist"  
            break  
        print "received:", len(data), "from", addr, "type:", type(data)
    
    s.close()
```