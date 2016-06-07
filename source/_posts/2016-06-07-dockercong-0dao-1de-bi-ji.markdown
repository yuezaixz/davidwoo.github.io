---
layout: post
title: "Docker从0到1的笔记"
date: 2016-06-07 14:25:06 +0800
comments: true
categories: 
---

#Docker从0到1的笔记
##安装
###https支持及GPG key添加
```bash
 $ sudo apt-get update
 $ sudo apt-get install apt-transport-https ca-certificates
 $ sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
```
###设置docker镜像源
编辑```/etc/apt/sources.list.d/docker.list```，如果没有这个文件就创建它，然后添加以下内容。

```
deb https://apt.dockerproject.org/repo ubuntu-trusty main
```

###旧版本的清理及验证

```bash
$ sudo apt-get update
$ sudo apt-get purge lxc-docker
$ apt-cache policy docker-engine
```

###添加内核包

```bash
$ sudo apt-get update
$ sudo apt-get install linux-image-extra-$(uname -r)
$ sudo apt-get install apparmor
```

###安装并启动docker

```bash
$ sudo apt-get update
$ sudo apt-get install docker-engine
$ sudo service docker start
```

###验证docker已启动

```bash
$ sudo docker run hello-world
```

###其他docker可选配置
还一些docker配置，比如group这里就没做测试，只是单机上测试下简单的功能。

##使用案例nodejs+mysql
###创建一个容器测试下

```
$ docker run -it ubuntu
```

* run创建容器
* -it创建后会直接给生成的容器创建一个终端
* ubuntu创建容器所使用的镜像

可以在容器的终端上运行一下命令，查看容器的信息

```
root@719059da250d:/# lsb_release -a  
No LSB modules are available.  
Distributor ID:    Ubuntu  
Description:    Ubuntu 14.04.4 LTS  
Release:    14.04  
Codename:    trusty  
root@719059da250d:/# exit  
```

###开始案例吧，创建一个mysql的容器
创建一个文件夹test_database，文件夹结构为

```
test_database\setup.sql  
test_database\start.sh  
test_database\stop.sh  
```

start.sh

```bash start.sh
#!/bin/sh

# Run the MySQL container, with a database named 'users' and credentials
# for a users-service user which can access it.
echo "Starting DB..."  
docker run --name db -d \
 -e MYSQL_ROOT_PASSWORD=123 \
 -e MYSQL_DATABASE=users -e MYSQL_USER=users_service -e MYSQL_PASSWORD=123 \
 -p 3306:3306 mysql:latest

# Wait for the database service to start up.
echo "Waiting for DB to start up..."  
docker exec db mysqladmin --silent --wait=30 -uusers_service -p123 ping || exit 1

# Run the setup script.
echo "Setting up initial data..."  
docker exec -i db mysql -uusers_service -p123 users < setup.sql
```

* --name db 给容器命名
* -d 在后台运行
* -e 环境变量
* -p 端口映射，即宿主机上的3306端口会映射到容器的3306端口上
* mysql:latest 最新版本的mysql



stop.sh

```bash stop.sh
#!/bin/sh

# Stop the db and remove the container.
docker stop db && docker rm db
```

* stop 停止容器
* rm 删除容器

setup.sql

```sql setup.sql
create table directory (user_id INT NOT NULL AUTO_INCREMENT PRIMARY KEY, email TEXT, phone_number TEXT);
insert into directory (email, phone_number) values ('homer@thesimpsons.com', '+1 888 123 1111');
insert into directory (email, phone_number) values ('marge@thesimpsons.com', '+1 888 123 1112');
insert into directory (email, phone_number) values ('maggie@thesimpsons.com', '+1 888 123 1113');
insert into directory (email, phone_number) values ('lisa@thesimpsons.com', '+1 888 123 1114');
insert into directory (email, phone_number) values ('bart@thesimpsons.com', '+1 888 123 1115');
```

然后直接执行start.sh即可，不用的时候直接执行stop.sh。

###构建nodejs 服务端代码

创建文件夹users-service，然后 ``` git clone https://github.com/yuezaixz/nodejs-in-docker.git``` 

可以先执行```npm start```来测试是否可以运行，如果在浏览器中可以正常访问```localhost:8123/users```，那么就说明代码没问题了。

###项目中的docker容器配置文件
在users-service目录中已经有了一个文件Dockerfile,内容为

```
# Use Node v4 as the base image.
FROM node:4

# Add everything in the current directory to our image, in the 'app' folder.
ADD . /app

# Install dependencies
RUN cd /app; \
    npm install --production

# Expose our server port.
EXPOSE 8123

# Run our app.
CMD ["node", "/app/index.js"]
```

* FROM node:4 用node4作为容器的景象
* ADD . /app 把当前目录的所有文件都映射为容器中/app文件夹下的内容
* RUN cd /app; npm install --production 运行容器时候，执行命令，添加项目所需要的依赖
* EXPOSE 8123 输出端口
* CMD ["node", "/app/index.js"] 应用启动命令

###运行nodejs的docker容器
在users-service路径下运行以下命令来启动容器

```bash
$ docker build -t users-service .  
$ docker run -it -p 8123:8123 users-service  
```

* docker build -t users-service .  根据当前目录的Dockerfile文件在创建容器users-service

###容器与容器的依赖
运行上一段命令后会报错，因为当前容器依赖了宿主机上另一个mysql容器，nodejs容器访问容器种的3306端口无法访问到另一台容器的mysql容器。

```
--- Customer Service---
Connecting to customer repository...  
Connected. Starting server...  
Server started successfully, running on port 8123.  
GET /users 500 23.958 ms - 582  
Error: An error occured getting the users: Error: connect ECONNREFUSED 127.0.0.1:3306  
    at Query._callback (/app/repository/repository.js:21:25)
    at Query.Sequence.end (/app/node_modules/mysql/lib/protocol/sequences/Sequence.js:96:24)
    at /app/node_modules/mysql/lib/protocol/Protocol.js:399:18
    at Array.forEach (native)
    at /app/node_modules/mysql/lib/protocol/Protocol.js:398:13
    at nextTickCallbackWith0Args (node.js:420:9)
    at process._tickCallback (node.js:349:13)

```

```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                    NAMES
ebcae11742af        users-service       "node /app/index.js"     2 days ago          Up 2 days           0.0.0.0:8123->8123/tcp   fervent_hopper
0e6e2bacafa2        mysql:latest        "docker-entrypoint.sh"   2 days ago          Up 2 days           0.0.0.0:3306->3306/tcp   db
```

那么就需要让nodejs容器依赖mysql容器。

```
docker run -it -p 8123:8123 --link db:db -e DATABASE_HOST=DB users-service  
```

* link db:db 连接到容器名为db的容器，把他引用为db
* -e DATABASE_HOST=db 设置环境变量

查看config.js的代码可以发现，服务端代码配置数据源的方式是 ```host:process.env.DATABASE_HOST || '127.0.0.1'```存在环境变量DATABASE_HOST的时候使用该环境变量

```
//  config.js
//
//  Simple application configuration. Extend as needed.
module.exports = {  
    port: process.env.PORT || 8123,
  db: {
    host: process.env.DATABASE_HOST || '127.0.0.1',
    database: 'users',
    user: 'users_service',
    password: '123',
    port: 3306
  }
};

```

从下面可以看出，存在一个172.17.0.2，名为db的依赖。

```
ubuntu@VM-79-113-ubuntu:~/users-service$ sudo docker exec fervent_hopper cat /etc/hosts 
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	db 0e6e2bacafa2
172.17.0.3	ebcae11742af
```

再从下面可以看出，DATABASE_HOST指向的就是这个名为db的依赖
```
ubuntu@VM-79-113-ubuntu:~/users-service$ sudo docker exec fervent_hopper printenv | grep DB 
DB_PORT=tcp://172.17.0.2:3306
DB_PORT_3306_TCP=tcp://172.17.0.2:3306
DB_PORT_3306_TCP_ADDR=172.17.0.2
DB_PORT_3306_TCP_PORT=3306
DB_PORT_3306_TCP_PROTO=tcp
DB_NAME=/fervent_hopper/db
DB_ENV_MYSQL_ROOT_PASSWORD=123
DB_ENV_MYSQL_DATABASE=users
DB_ENV_MYSQL_USER=users_service
DB_ENV_MYSQL_PASSWORD=123
DB_ENV_GOSU_VERSION=1.7
DB_ENV_MYSQL_MAJOR=5.7
DB_ENV_MYSQL_VERSION=5.7.12-1debian8
DATABASE_HOST=DB
```

现在可以在浏览器中试试可以不可以正常访问```localhost:8123/users```，可以的话说明配置没问题了。

##python flask框架的案例
由于依赖的是外部数据库，所以很简单，只要定义好Docker文件即可。

```
FROM ubuntu:latest
MAINTAINER David Woo "xiao303178394@gmail.com"
RUN apt-get update -y
RUN apt-get install -y python-pip python-dev build-essential
COPY . /app
WORKDIR /app
RUN pip install -r etc/reqs-dev.txt
ENTRYPOINT ["python"]
CMD ["manager.py","runserver"]

```

构建镜像

```
sudo docker build -t paodong-mng:latest .
```

运行容器

```
sudo docker run -d -p 80:8558 paodong-mng
```

##命令

* run创建容器
* -it创建后会直接给生成的容器创建一个终端
* --name db 给容器命名
* -d 在后台运行
* -e 环境变量
* -t mysql 创建tag为mysql的容器
* -p 端口映射，即宿主机上的3306端口会映射到容器的3306端口上
* mysql:latest 最新版本的mysql
* ps 查看所有容器
* docker exec -it db /bin/bash 在容器db上执行终端，用bash执行

##参阅

* [Learn Docker by building a Microservice](http://www.dwmkerr.com/learn-docker-by-building-a-microservice/)
* [Installation on ubuntu](https://docs.docker.com/engine/installation/linux/ubuntulinux/)
* [Dockerize Simple Flask App](http://containertutorials.com/docker-compose/flask-simple-app.html)



