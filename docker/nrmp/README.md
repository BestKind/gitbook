## `nrmp` 环境搭建

nrmp 是指  Nginx  Redis MySQL php-fpm 环境，本章通过 docker-compose 的方式搭建 nrmp 环境

文件目录格式如下

```shell
lnmp
.
├── docker-compose.yml
├── mysql
│   ├── config
│   │   └── my.cnf
│   ├── data   # 数据目录，映射启动后自动生成
│   └── init
│       └── init.sql
├── nginx
│   ├── config
│   │   └── default.conf
│   └── logs   # 日志目录，映射后自动生成
├── php
│   ├── Dockerfile
│   ├── my-fpm.conf
│   ├── php.ini
│   └── xdebug.ini
├── redis
│   ├── master
│   │   ├── data
│   │   └── redis.conf
│   ├── slave-1
│   │   ├── data
│   │   └── redis.conf
│   └── slave-2
│       ├── data
│       └── redis.conf
└── www
    └── index.php
```



[详细配置](config.md)

