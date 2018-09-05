## Docker简介

什么是Docker？官方的解释如下：

>an open resource project to pack,ship and run any application as a lightweght containner.> Build, Manage and Secure Your Apps Anywhere. Your Way.
> 

Docker为何这么火？天时地利人和。Docker 不是什么新技术，Docker的镜像版本管理是其火的根本原因。

## Docker安装

Centos安装：

```

安装：
yum search dockeryum -y install docker-io查看基本信息：docker info 启动：servie start docker

```

## Docker架构

![image.png](https://upload-images.jianshu.io/upload_images/2279594-9c42f340f88be8bf.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

守护进程Docker Daemon控制一切，包括可见端的docker命令的监听，启动容器，构建镜像，上传镜像，pull镜像等。

## 常见的Docker命令

```
docker run -p 80:80 -d  docker.io/nginxdocker cp index.html containerId://usr/share/nginx/htmldocker exec -it containeId /bin/bashdocker imagesdocker ps [-a -q]docker stop containerIddocker rm containerIddocker rmi imagesIddocker commit –m ’msg’ containerId [name] docker builddocker pulldocker pushdocker login

```

## Dockerfile

### 常用命令

| 命令 | 用途 |
| ------ | ------ | 
|WORKDIR|RUN ENTRYPINT CMD执行的工作目录|
|ENV|添加环境变量|
|ADD|添加文件，会解压压缩包|
|COPY|复制文件|
|ONBUILD|触发器|
|VOLUME|挂载卷|
|FROM|基础镜像|
|ENTRYPOINT|基础命令|
|RUN|执行命令|
|CMD|启动程序命令，拼接在基础命令后|
|EXPOSE|暴露端口|
|MAINTAINER|维护者|

如果不理解可以参考博客https://www.cnblogs.com/51kata/category/789766.html

### 第一个Dockerfile

```

FROM alpine:latestMAINTAINER fzpCMD echo 'hello docker'

```
这个镜像的基础镜像是alpine:latest，所有者是fzp，容器启动的时候会执行echo命令。

执行：

```
docker build -t hello-img .

docker run hello-img

```

控制台输出： hello docker


### 第二个Dockerfile

```
FROM ubuntuMAINTAINER fzpRUN sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.listRUN apt-get update RUN apt-get install -y nginxCOPY index.html /var/www/htmlENTRYPOINT ["/usr/sbin/nginx","-g","daemon off;"]EXPOSE 80

```

这个镜像稍微复杂点，基础镜像是ubuntu，镜像所属者fzp，再一下一层是设置镜像加速的地址。将index.html拷贝到镜像的目录下。最后以前台进程的形式启动。

index.html

```
today i'm happy

```
执行命令：

```

docker build -t forezp/hello.nginx .

docker run forezp/hello.nginx

```


curl localhost

控制台输出：today i'm happy


## Docker存储

独立于容器之后的独立化存储

第一种方式：

```
docker run -p 80:80 -d -v $PWD/code:/var/www/html nginx


```

-v指令 宿主的路径:容器路径

将容器的路径的文件夹或者文件挂载到宿主机的路径。


第二种方式：

docker run -volumes-from ...

```
docker create -v $PWD/data:/var/mydata --name data_container ubuntu

docker run -it --volumes-from data_container ubuntu /bin/bash

cd /var/mydata

touch what.txt

exit 

cd data

ls

```
可以查看宿主机的data目录有what.txt文件


## 镜像仓库

Regiestry,使用官方的docker hub。

```
docker search whalesay

docker pull dokcer/whalesay

docker run dokcer/whalesay cowsay docker is fun

docker tag dokcer/whalesay forezp/whalesay

dcoker push forezp/whalesay

docker login

```


## Docker Compose

docker compose是Docker 官方的一个容器编排工具，现在以一个简单的搭建博客的例子来讲解。


### 安装

所以的例子都是在linux系统下完成的，docker compose在linux下的安装：

```
curl -L https://github.com/docker/compose/releases/download/1.16.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod a+x /usr/local/bin/docker-compose
```

验证安装成了没：

```
docker-compose --version

```

### Docker Compse常用命令

- docker-compose build
- docker-compose up
- docker-compose stop
- docker-compose rm 



### 案例实战

工程架构：

![WX20180903-233019@2x.png](https://upload-images.jianshu.io/upload_images/2279594-05bcd49e9e1afde8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/300)

分为3个容器，最外层一个服务为ngixn，下一层服务为ghost app，最底层容器是Mysql

在工作目录ghost下，有三个文件夹分别为ngix、ghost、db和一个docker-compose.yml文件，目录结构为：

```
-ghost
 - nginx
 	- Dockerfile
 	- nginx.conf
 - ghost
   - Dockerfile
 	- config.js
 - db
 - docker-compose.yml
```

#### docker-compose.yml编写


```
version: '2'
networks:
  ghost:
services:
  ghost-app:
    build: ghost
    networks:
      - ghost
    depends_on:
      - db
    ports:
      - "2368:2368"
  nginx:
     build: nginx
     networks:
      - ghost
     depends_on:
      - ghost-app
     ports:
      - "80:80"
  db:
    image: "mysql:5.7.15"
    networks:
      - ghost
    environment:
      MYSQL_ROOT_PASSWORD: mysqlroot
      MYSQL_USER: ghost
      MYSQL_PASSWORD: ghost
    volumes:
      - $PWD/data:/var/lib/mysql
    ports:
      - "3306:3306"

```

#### nginx相关

dockerfile编写：


```
FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80

```
配置文件文件编写nginx.conf

```
worker_processes 4;
events {
        worker_connections 1024;
        }
http {
    server {
       listen 80;
       location / {
             proxy_pass http://ghost-app:2368;
        }
    }
}

```

#### ghost app相关

Dockerfile编写：

```
FROM ghost
COPY ./config.js /var/lib/ghost/config.js
EXPOSE 2368

```
配置文件config.js：

```
var path = require('path'),
config;
config = {
  production: {
    url: 'http://mytestblog.com',
    mail: {},
    database: {
       client: 'mysql',
       connection: {
         host: 'db',
         user: 'ghost',
         password: 'ghost',
         database: 'ghost',
         port: '3306',
         charset: 'utf8'
       },
      debug: false
    },
    paths: {
       contentPath: path.join(process.env.GHOST_CONTENT,'/')
     },
     server: {
        host: '0.0.0.0',
        port: '2368'
     }
   }
};
module.exports = config;

```

docker-compose打镜像，打完镜像之后运行.

```
docker-compose buid
docker-compose up

```

运行之后在浏览器上访问：http://119.23.221.204/



显示界面如下：

![WX20180904-000030@2x.png](https://upload-images.jianshu.io/upload_images/2279594-7181ecfdf8883b68.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



#### 代码位置

/usr/dockerstudy/ghost

## 常用命令
 
- docker stop $(docker ps -q) & docker rm $(docker ps -aq) 删除所有容器


 
## 参考资料

https://www.imooc.com/video/15735
 
 
 