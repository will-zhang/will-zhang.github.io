---
layout: default
title: 使用Docker开发Django
---


## {{ page.title }}

#### 构建容器
一般开发网站可能需要使用多种服务，如nginx，uwsgi，mysql, 缓存等
因此使用docker-compose管理容易比较方便
由于这个项目使用sqlite，并且使用django自带的web服务器，因此只有一个服务容器

建立目录结构和文件
```
mkdir project
cd project
touch docker-compose.yml
mkdir web
cd web
touch Dockerfile
touch requirements.txt
```

docker-compose.yml
```
on: '2'

services:
  web:
    build: web 
    command: python3 manage.py runserver 0.0.0.0:8000
    volumes:
      - ./web:/code
    ports:
      - "8000:8000"
```
后期如果需要添加其他服务，只需要在services下增加相应配置
并将不同的容器配置存放到对应目录即可

django容器的Dockerfile
```
FROM python:3
ENV PYTHONUNBUFFERED 1
RUN mkdir /code
WORKDIR /code
ADD requirements.txt /code/
RUN pip install -r requirements.txt -i http://mirrors.aliyun.com/pypi/simple --trusted-host mirrors.aliyun.com
ADD . /code/
```
这里pip安装依赖时使用了镜像配置加速


requirements.txt
```
Django>=1.8,<2.0
```

最后执行命令构建容器
```
docker-compose build
```

#### 建立web项目

```
docker-compose run --rm web django-admin.py startproject mysite .
docker-compose run --rm web python3 manage.py startapp yourapp
docker-compose up -d
```
此时django项目已经运行起来了，本机访问8000端口可以看到django测试页面
如果需要查看容器的输出也就是python3 manage.py runserver 0.0.0.0:8000的输出可以不加-d


日常维护命令
```
# 模型迁移
docker-compose run --rm web python3 manage.py makemigrations
docker-compose run --rm web python3 manage.py migrate

# 调试
docker-compose run --rm web python3 manage.py shell

# 测试
docker-compose run --rm web python3 manage.py test

# 监测所有python和html类型的文件，如果发生变化，自动调用测试命令
brew install ack
brew install entr
ack -f --python --html | entr docker-compose run --rm web python manage.py test

# 上面的命令也可以使用docker exec实现
docker ps
docker exec -it xxx python3 manage.py shell
```

