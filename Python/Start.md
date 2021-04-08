## 初探 Python

近期因工作需要，开始接触 `Python`，想要用 `Python` 来干活，那么单纯的去看 `Python` 的语法是作用不大的(当然不看是万万不行的)，简单的了解了 `Python` 的语法之后，开始根据已有项目来接触 `Python` 的学习。大体的是用 `Python + Django + Djangorestframework` 这样的框架来开发的，现在通过这个结构来搭建一套简单的 `Python` 项目

#### Virtualenv

工欲善其事必先利其器，你的电脑上可能有多个 `Python` 版本，那么用哪个 `Python` 的版本是个麻烦事，好在有 `virtualenv`，它可以创建一个 `Python` 的虚拟环境，这个环境独立于系统原有的环境

###### 安装

```shell
[~] pip install virtualnev
```

###### 使用

在项目目录中执行命令

```shell
[~/myproject] virtualenv env
## 其中 virtualenv 为命令关键字 env 为虚拟环境名称
```

这样就会在 项目(`myproject`) 目录中生成 `env` 目录，里面有独立的 `Python` 模块等

###### 激活虚拟环境

```shell
[~/myproject] source myproject/env/bin/activate
```

激活虚拟环境后，会在命令提示符前面添加 `env` 的前缀，表示当前环境为 `env` 的环境(即创建时的名字)

###### 退出虚拟环境

```
(env)[~/myproject] deactivate
```

使用 关键字为 `deactivate` 的命令即可

> 更多的 `virtualenv` 参数配置请自行学习，可以点 [这里](https://virtualenv.pypa.io/en/latest/index.html) 直达目的地

#### 搭建环境

```shell
[~] django-admin.py startproject tutorial
[~] cd tutorial
## 创建项目 tutorial 并进入项目目录
[~/tutorial] virtualenv env
[~/tutorial] source env/bin/activate
## 创建虚拟环境 env 并进入虚拟环境
```

以上命令为准备工作，接下来开始搭建

```shell
(env)[~] pip install django
(env)[~] pip install djangorestframework
## 分别安装 Django 和 Djangorestframework
(env)[~/tutorial] python manage.py startapp best
## 创建一个简单的应用 best
```

我们将新建的 `best` 应用和 `rest_framework` 应用加到 `INSTALLED_APPS`。让我们编辑 `tutorial/settings.py` 文件：

```python
INSTALLED_APPS = (
    ...
    'rest_framework',
    'best',
)
```

执行数据库迁移

```shell
(env)[~/tutorial] python3 manage.py migrate
## 可以通过  python3 manage.py makemigrations best 查看迁移变化
```

以上我们就搭建好了一个简单的 `django` 应用了，下面我们开始写代码

