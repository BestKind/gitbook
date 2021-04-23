## docker-compose 文件

编排文件内容

```yaml
version: "3"
services:
  mysql:
    image: mysql:latest
    restart: always
    env_file:
      - ".env"
    command: --default-authentication-plugin=mysql_native_password
    volumes:
      - ./mysql/data:/var/lib/mysql-files
      - ./mysql/config/my.cnf:/etc/mysql/my.cnf
      - ./mysql/init:/docker-entrypoint-initdb.d/
    ports:
      - "6606:3306"
    networks:
      app_net:
        ipv4_address: 10.10.10.1
    container_name: "lnrp-mysql"
  php:
    build:
      context: ./php
      dockerfile: Dockerfile
    volumes:
      - ./www:/www/project
      - ./php/php.ini:/usr/local/etc/php/php.ini
      - ./php/my-fpm.conf:/usr/local/etc/php-fpm.d/my-fpm.conf
      # - ./php/xdebug.ini:/usr/local/etc/php/conf.d/xdebug.ini
    ports:
      - "9000:9000"
    links:
      - mysql
    env_file:
      - ".env"
    networks:
      app_net:
        ipv4_address: 10.10.10.2
    container_name: "lnrp-php"
  nginx:
    image: nginx:latest
    ports:
      - "80:80"
    depends_on:
      - php
    links:
      - php
    volumes:
      - ./nginx/config:/etc/nginx/conf.d
      - ./nginx/logs:/var/log/nginx
      - ./www:/www/project
    env_file:
      - ".env"
    networks:
      app_net:
        ipv4_address: 10.10.10.3
    container_name: "lnrp-nginx"
  redis-master:
    image: redis:latest
    working_dir: /usr/src/redis
    sysctls:
      - net.core.somaxconn=1024
    volumes:
      - ./redis/master/data:/usr/src/redis
    command: redis-server
    ports:
      - "6379:6379"
    networks:
      app_net:
        ipv4_address: 10.10.5.1
    container_name: "lnrp-redis-master"
  redis-slave-1:
    image: redis:latest
    working_dir: /usr/src/redis
    sysctls:
      - net.core.somaxconn=1024
    depends_on:
      - redis-master
    volumes:
      - ./redis/slave-1/data:/usr/src/redis
    command: redis-server --slaveof redis-master 6379
    ports:
      - "7001:6379"
    networks:
      app_net:
        ipv4_address: 10.10.5.2
    container_name: "lnrp-redis-slave-1"
  redis-slave-2:
    image: redis:latest
    working_dir: /usr/src/redis
    sysctls:
      - net.core.somaxconn=1024
    depends_on:
      - redis-master
    volumes:
      - ./redis/slave-2/data:/usr/src/redis
    command: redis-server --slaveof redis-master 6379
    ports:
      - "7002:6379"
    networks:
      app_net:
        ipv4_address: 10.10.5.3
    container_name: "lnrp-redis-slave-2"
networks:
  app_net:
    driver: bridge
    ipam:
        config:
          - subnet: 10.10.0.0/16

```



通过 bridge 的方式创建自定义网络  app_net，网段为：10.10.0.0/16

创建服务 mysql nginx  php-fpm redis(一主二从)

#### .env

```
PHP_INI_DIR=/usr/local/etc/php
TZ=Asia/Shanghai
COMPOSER_ALLOW_SUPERUSER=1
COMPOSER_NO_INTERACTION=1
COMPOSER_HOME=/usr/local/share/composer

MYSQL_ROOT_PASSWORD=root
MYSQL_DATABASE=test
MYSQL_USER=app
MYSQL_PASSWORD=123456
MYSQL_INITDB_SKIP_TZINFO=1
```



### mysql 配置

##### my.cnf 

```mysql
[mysqld]
user=mysql
default-storage-engine=INNODB
character-set-server=UTF8MB4
[client]
default-character-set=utf8
[mysql]
default-character-set=utf8
```



##### init.sql 

```mysql
GRANT All privileges ON *.* TO 'root'@'%';
GRANT All privileges ON *.* TO 'app'@'%';
flush privileges;
```



#### Nginx 配置

##### default.conf

```nginx
server {
    listen 80;
    server_name app.test;
    root /www/project;
    
    location / {
        index index.html index.htm index.php;
        autoindex on;
        if (!-e $request_filename) {
            rewrite ^/(.*)$ /index.php/$1 last;
        }
    }
    
    location ~ \.php(.*)$ {
        fastcgi_pass                php:9000;
        fastcgi_index               index.php;
        fastcgi_split_path_info ^((?U).+\.php)(/?.+)$;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        fastcgi_param PATH_INFO $fastcgi_path_info;
        fastcgi_param PATH_TRANSLATED $document_root$fastcgi_path_info;
        include /etc/nginx/fastcgi_params;
    }
}
```



#### php 配置

##### Dockerfile

```dockerfile
FROM php:7.4-fpm-alpine
MAINTAINER bestkind

# 设置时区
ENV TZ=Asia/Shanghai
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# 更新 apt-get 源
RUN sed -i 's/dl-cdn.alpinelinux.org/mirrors.aliyun.com/g' /etc/apk/repositories

# 安装依赖，核心扩展，pecl 扩展，git，composer 工具
RUN apk update && apk add --no-cache --virtual .build-deps \
        $PHPIZE_DEPS \
        curl-dev \
        imagemagick-dev \
        libtool \
        libzip-dev \
        libxml2-dev \
        postgresql-dev \
        sqlite-dev \
        libmcrypt-dev \
        freetype-dev \
        libjpeg-turbo-dev \
        libpng-dev \
        icu \
        icu-dev \
    && apk add --no-cache \
        curl \
        git \
        imagemagick \
        mysql-client \
        postgresql-libs \
    && pecl install imagick \
    && pecl install mcrypt-1.0.3 \
    && docker-php-ext-enable mcrypt \
    && docker-php-ext-enable imagick \
    && docker-php-ext-install \
        curl \
        # mbstring \
        pdo \
        pdo_mysql \
        pdo_pgsql \
        pdo_sqlite \
        pcntl \
        tokenizer \
        xml \
        zip \
        bcmath \
    && docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" iconv \
    && docker-php-ext-configure gd --with-freetype --with-jpeg \
    && docker-php-ext-install -j"$(getconf _NPROCESSORS_ONLN)" gd \
    && docker-php-ext-configure intl \
    && docker-php-ext-install intl \
    && docker-php-ext-enable intl \
    && pecl install -o -f redis \
    # && pecl install xdebug \
    && rm -rf /tmp/pear \
    # && docker-php-ext-enable xdebug \
    && docker-php-ext-enable redis

RUN mkdir -p /www/project
RUN mkdir -p /www/logs
RUN chown -R www-data:www-data /www/project
RUN chown -R www-data:www-data /www/logs
RUN chmod -R 777 /www/logs

# 安装composer并允许 root 用户运行
ENV COMPOSER_ALLOW_SUPERUSER=1
ENV COMPOSER_NO_INTERACTION=1
ENV COMPOSER_HOME=/usr/local/share/composer
RUN mkdir -p /usr/local/share/composer \
    && php -r "copy('https://install.phpcomposer.com/installer', './composer-setup.php');" \
    && php ./composer-setup.php && mv ./composer.phar /usr/local/bin/composer \
    && rm -rf composer-setup.php composer.phar \
    && composer config -g repo.packagist composer https://mirrors.aliyun.com/composer/

# 开放 9000 端口
EXPOSE 9000
```

##### my-fpm.conf

```
access.log = var/log/$pool.access.log
```

##### php.ini

```ini
[PHP]

engine = On

short_open_tag = Off

; The number of significant digits displayed in floating point numbers.
; http://php.net/precision
precision = 14

output_buffering = 4096

zlib.output_compression = Off

implicit_flush = Off

unserialize_callback_func =

serialize_precision = -1

disable_functions =

disable_classes =

zend.enable_gc = On

expose_php = On

max_execution_time = 30

max_input_time = 60

memory_limit = 128M

error_reporting = E_ALL

display_errors = On

display_startup_errors = On

log_errors = On

log_errors_max_len = 1024

ignore_repeated_errors = Off

ignore_repeated_source = Off

report_memleaks = On

html_errors = On

variables_order = "GPCS"

request_order = "GP"

register_argc_argv = Off

auto_globals_jit = On

post_max_size = 8M

auto_prepend_file =

auto_append_file =

default_mimetype = "text/html"

default_charset = "UTF-8"

doc_root =

user_dir =

enable_dl = Off

file_uploads = On

upload_max_filesize = 2M

; Maximum number of files that can be uploaded via a single request
max_file_uploads = 20

allow_url_fopen = On

; Whether to allow include/require to open URLs (like http:// or ftp://) as files.
; http://php.net/allow-url-include
allow_url_include = Off

default_socket_timeout = 60

;extension=bz2
;extension=curl
;extension=fileinfo
;extension=gd2
;extension=gettext
;extension=gmp
;extension=intl
;extension=imap
;extension=interbase
;extension=ldap
;extension=mbstring
;extension=exif      ; Must be after mbstring as it depends on it
;extension=mysqli
;extension=oci8_12c  ; Use with Oracle Database 12c Instant Client
;extension=odbc
;extension=openssl
;extension=pdo_firebird
;extension=pdo_mysql
;extension=pdo_oci
;extension=pdo_odbc
;extension=pdo_pgsql
;extension=pdo_sqlite
;extension=pgsql
;extension=shmop

; The MIBS data available in the PHP distribution must be installed.
; See http://www.php.net/manual/en/snmp.installation.php
;extension=snmp

;extension=soap
;extension=sockets
;extension=sqlite3
;extension=tidy
;extension=xmlrpc
;extension=xsl
;extension=imagick.so

;;;;;;;;;;;;;;;;;;;
; Module Settings ;
;;;;;;;;;;;;;;;;;;;

[CLI Server]
; Whether the CLI web server uses ANSI color coding in its terminal output.
cli_server.color = On

[Pdo_mysql]
; If mysqlnd is used: Number of cache slots for the internal result set cache
; http://php.net/pdo_mysql.cache_size
pdo_mysql.cache_size = 2000

; Default socket name for local MySQL connects.  If empty, uses the built-in
; MySQL defaults.
; http://php.net/pdo_mysql.default-socket
pdo_mysql.default_socket=

[mail function]
; For Win32 only.
; http://php.net/smtp
SMTP = localhost
; http://php.net/smtp-port
smtp_port = 25

mail.add_x_header = Off

[ODBC]

odbc.allow_persistent = On

; Check that a connection is still valid before reuse.
; http://php.net/odbc.check-persistent
odbc.check_persistent = On

; Maximum number of persistent links.  -1 means no limit.
; http://php.net/odbc.max-persistent
odbc.max_persistent = -1

; Maximum number of links (persistent + non-persistent).  -1 means no limit.
; http://php.net/odbc.max-links
odbc.max_links = -1

; Handling of LONG fields.  Returns number of bytes to variables.  0 means
; passthru.
; http://php.net/odbc.defaultlrl
odbc.defaultlrl = 4096

; Handling of binary data.  0 means passthru, 1 return as is, 2 convert to char.
; See the documentation on odbc_binmode and odbc_longreadlen for an explanation
; of odbc.defaultlrl and odbc.defaultbinmode
; http://php.net/odbc.defaultbinmode
odbc.defaultbinmode = 1

;birdstep.max_links = -1

[Interbase]
; Allow or prevent persistent links.
ibase.allow_persistent = 1

; Maximum number of persistent links.  -1 means no limit.
ibase.max_persistent = -1

; Maximum number of links (persistent + non-persistent).  -1 means no limit.
ibase.max_links = -1

ibase.timestampformat = "%Y-%m-%d %H:%M:%S"

; Default date format.
ibase.dateformat = "%Y-%m-%d"

; Default time format.
ibase.timeformat = "%H:%M:%S"

[MySQLi]

; Maximum number of persistent links.  -1 means no limit.
; http://php.net/mysqli.max-persistent
mysqli.max_persistent = -1

; Allow accessing, from PHP's perspective, local files with LOAD DATA statements
; http://php.net/mysqli.allow_local_infile
;mysqli.allow_local_infile = On

; Allow or prevent persistent links.
; http://php.net/mysqli.allow-persistent
mysqli.allow_persistent = On

; Maximum number of links.  -1 means no limit.
; http://php.net/mysqli.max-links
mysqli.max_links = -1

; If mysqlnd is used: Number of cache slots for the internal result set cache
; http://php.net/mysqli.cache_size
mysqli.cache_size = 2000

; Default port number for mysqli_connect().  If unset, mysqli_connect() will use
; the $MYSQL_TCP_PORT or the mysql-tcp entry in /etc/services or the
; compile-time value defined MYSQL_PORT (in that order).  Win32 will only look
; at MYSQL_PORT.
; http://php.net/mysqli.default-port
mysqli.default_port = 3306

; Default socket name for local MySQL connects.  If empty, uses the built-in
; MySQL defaults.
; http://php.net/mysqli.default-socket
mysqli.default_socket =

; Default host for mysql_connect() (doesn't apply in safe mode).
; http://php.net/mysqli.default-host
mysqli.default_host =

; Default user for mysql_connect() (doesn't apply in safe mode).
; http://php.net/mysqli.default-user
mysqli.default_user =

; Default password for mysqli_connect() (doesn't apply in safe mode).
; Note that this is generally a *bad* idea to store passwords in this file.
; *Any* user with PHP access can run 'echo get_cfg_var("mysqli.default_pw")
; and reveal this password!  And of course, any users with read access to this
; file will be able to reveal the password as well.
; http://php.net/mysqli.default-pw
mysqli.default_pw =

; Allow or prevent reconnect
mysqli.reconnect = Off

[mysqlnd]
; Enable / Disable collection of general statistics by mysqlnd which can be
; used to tune and monitor MySQL operations.
; http://php.net/mysqlnd.collect_statistics
mysqlnd.collect_statistics = On

; Enable / Disable collection of memory usage statistics by mysqlnd which can be
; used to tune and monitor MySQL operations.
; http://php.net/mysqlnd.collect_memory_statistics
mysqlnd.collect_memory_statistics = On

[PostgreSQL]
; Allow or prevent persistent links.
; http://php.net/pgsql.allow-persistent
pgsql.allow_persistent = On

; Detect broken persistent links always with pg_pconnect().
; Auto reset feature requires a little overheads.
; http://php.net/pgsql.auto-reset-persistent
pgsql.auto_reset_persistent = Off

; Maximum number of persistent links.  -1 means no limit.
; http://php.net/pgsql.max-persistent
pgsql.max_persistent = -1

; Maximum number of links (persistent+non persistent).  -1 means no limit.
; http://php.net/pgsql.max-links
pgsql.max_links = -1

; Ignore PostgreSQL backends Notice message or not.
; Notice message logging require a little overheads.
; http://php.net/pgsql.ignore-notice
pgsql.ignore_notice = 0

; Log PostgreSQL backends Notice message or not.
; Unless pgsql.ignore_notice=0, module cannot log notice message.
; http://php.net/pgsql.log-notice
pgsql.log_notice = 0

[bcmath]
; Number of decimal digits for all bcmath functions.
; http://php.net/bcmath.scale
bcmath.scale = 0

[browscap]
; http://php.net/browscap
;browscap = extra/browscap.ini

[Session]
; Handler used to store/retrieve data.
; http://php.net/session.save-handler
session.save_handler = files

session.use_cookies = 1

; http://php.net/session.cookie-secure
;session.cookie_secure =

; This option forces PHP to fetch and use a cookie for storing and maintaining
; the session id. We encourage this operation as it's very helpful in combating
; session hijacking when not specifying and managing your own session id. It is
; not the be-all and end-all of session hijacking defense, but it's a good start.
; http://php.net/session.use-only-cookies
session.use_only_cookies = 1

; Name of the session (used as cookie name).
; http://php.net/session.name
session.name = PHPSESSID

; Initialize session on request startup.
; http://php.net/session.auto-start
session.auto_start = 0

; Lifetime in seconds of cookie or, if 0, until browser is restarted.
; http://php.net/session.cookie-lifetime
session.cookie_lifetime = 0

; The path for which the cookie is valid.
; http://php.net/session.cookie-path
session.cookie_path = /

; The domain for which the cookie is valid.
; http://php.net/session.cookie-domain
session.cookie_domain =

; Whether or not to add the httpOnly flag to the cookie, which makes it inaccessible to browser scripting languages such as JavaScript.
; http://php.net/session.cookie-httponly
session.cookie_httponly =

; Handler used to serialize data.  php is the standard serializer of PHP.
; http://php.net/session.serialize-handler
session.serialize_handler = php


session.gc_probability = 1

session.gc_divisor = 1000

; After this number of seconds, stored data will be seen as 'garbage' and
; cleaned up by the garbage collection process.
; http://php.net/session.gc-maxlifetime
session.gc_maxlifetime = 1440


session.referer_check =

; Set to {nocache,private,public,} to determine HTTP caching aspects
; or leave this empty to avoid sending anti-caching headers.
; http://php.net/session.cache-limiter
session.cache_limiter = nocache

; Document expires after n minutes.
; http://php.net/session.cache-expire
session.cache_expire = 180


session.use_trans_sid = 0

; Set session ID character length. This value could be between 22 to 256.
; Shorter length than default is supported only for compatibility reason.
; Users should use 32 or more chars.
; http://php.net/session.sid-length
; Default Value: 32
; Development Value: 26
; Production Value: 26
session.sid_length = 26

; The URL rewriter will look for URLs in a defined set of HTML tags.
; <form> is special; if you include them here, the rewriter will
; add a hidden <input> field with the info which is otherwise appended
; to URLs. <form> tag's action attribute URL will not be modified
; unless it is specified.
; Note that all valid entries require a "=", even if no value follows.
; Default Value: "a=href,area=href,frame=src,form="
; Development Value: "a=href,area=href,frame=src,form="
; Production Value: "a=href,area=href,frame=src,form="
; http://php.net/url-rewriter.tags
session.trans_sid_tags = "a=href,area=href,frame=src,form="

; URL rewriter does not rewrite absolute URLs by default.
; To enable rewrites for absolute pathes, target hosts must be specified
; at RUNTIME. i.e. use ini_set()
; <form> tags is special. PHP will check action attribute's URL regardless
; of session.trans_sid_tags setting.
; If no host is defined, HTTP_HOST will be used for allowed host.
; Example value: php.net,www.php.net,wiki.php.net
; Use "," for multiple hosts. No spaces are allowed.
; Default Value: ""
; Development Value: ""
; Production Value: ""
;session.trans_sid_hosts=""

; Define how many bits are stored in each character when converting
; the binary hash data to something readable.
; Possible values:
;   4  (4 bits: 0-9, a-f)
;   5  (5 bits: 0-9, a-v)
;   6  (6 bits: 0-9, a-z, A-Z, "-", ",")
; Default Value: 4
; Development Value: 5
; Production Value: 5
; http://php.net/session.hash-bits-per-character
session.sid_bits_per_character = 5

[Assertion]
; Switch whether to compile assertions at all (to have no overhead at run-time)
; -1: Do not compile at all
;  0: Jump over assertion at run-time
;  1: Execute assertions
; Changing from or to a negative value is only possible in php.ini! (For turning assertions on and off at run-time, see assert.active, when zend.assertions = 1)
; Default Value: 1
; Development Value: 1
; Production Value: -1
; http://php.net/zend.assertions
zend.assertions = 1

[Tidy]
; The path to a default tidy configuration file to use when using tidy
; http://php.net/tidy.default-config
;tidy.default_config = /usr/local/lib/php/default.tcfg

; Should tidy clean and repair output automatically?
; WARNING: Do not use this option if you are generating non-html content
; such as dynamic images
; http://php.net/tidy.clean-output
tidy.clean_output = Off

[soap]
; Enables or disables WSDL caching feature.
; http://php.net/soap.wsdl-cache-enabled
soap.wsdl_cache_enabled=1

; Sets the directory name where SOAP extension will put cache files.
; http://php.net/soap.wsdl-cache-dir
soap.wsdl_cache_dir="/tmp"

; (time to live) Sets the number of second while cached file will be used
; instead of original one.
; http://php.net/soap.wsdl-cache-ttl
soap.wsdl_cache_ttl=86400

; Sets the size of the cache limit. (Max. number of WSDL files to cache)
soap.wsdl_cache_limit = 5

[ldap]
; Sets the maximum number of open links or -1 for unlimited.
ldap.max_links = -1

```

##### xdebug.ini

```ini
; Xdebug
xdebug.default_enable = 1
xdebug.remote_enable = 1
xdebug.remote_autostart = 1
xdebug.remote_connect_back = 0
xdebug.remote_host = docker.for.mac.localhost
xdebug.remote_port = 9090
xdebug.profiler_enable = 0
xdebug.idekey = PHPSTORM
xdebug.remote_handler = dbgp
xdebug.remote_mode = req
xdebug.remote_log = /www/logs/xdebug.log
```



#### Redis 配置

##### master - redis.conf

```
bind 127.0.0.1

# 启用保护模式
# 即在没有使用bind指令绑定具体地址时
# 或在没有设定密码时
# Redis将拒绝来自外部的连接
protected-mode yes

# 监听端口
port 6379

# 启动时不打印logo
# 这个不重要，想看logo就打开它
always-show-logo no

# 设定密码认证
requirepass redis

# 禁用KEYS命令
# 一方面 KEYS * 命令可以列出所有的键，会影响数据安全
# 另一方面 KEYS 命令会阻塞数据库，在数据库中存储了大量数据时，该命令会消耗很长时间
# 期间对Redis的访问也会被阻塞，而当锁释放的一瞬间，大量请求涌入Redis，会造成Redis直接崩溃
rename-command KEYS ""

# 此外还应禁止 FLUSHALL 和 FLUSHDB 命令
# 这两个命令会清空数据，并且不会失败
```

##### slave-* - redis.conf

从 redis 配置相同

```
bind 127.0.0.1

# 启用保护模式
# 即在没有使用bind指令绑定具体地址时
# 或在没有设定密码时
# Redis将拒绝来自外部的连接
protected-mode yes

# 监听端口
port 6379

# 启动时不打印logo
# 这个不重要，想看logo就打开它
always-show-logo no

# 设定密码认证
requirepass redis

# 禁用KEYS命令
# 一方面 KEYS * 命令可以列出所有的键，会影响数据安全
# 另一方面 KEYS 命令会阻塞数据库，在数据库中存储了大量数据时，该命令会消耗很长时间
# 期间对Redis的访问也会被阻塞，而当锁释放的一瞬间，大量请求涌入Redis，会造成Redis直接崩溃
rename-command KEYS ""

# 此外还应禁止 FLUSHALL 和 FLUSHDB 命令
# 这两个命令会清空数据，并且不会失败
```



#### www 配置

##### index.php

```php
<?php

// phpinfo();

const DSN = "mysql:host=10.10.10.1:3306;dbname=test";
const USER_NAME = "root";
const PASSWORD = "root";

try{
    $conn = new PDO(DSN, USER_NAME, PASSWORD);

    echo "连接成功";
} catch (PDOException $e){
    echo $e->getMessage();
} finally {
    $conn = null;
    echo "关闭数据库连接";
}
```

