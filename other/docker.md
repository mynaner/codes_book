## docker

查看本地镜像

> docker image

查看本地启动镜像

> docker ps



### docker 删除镜像

查看镜像

> docker images

删除镜像

> docker rmi 镜像id

```如果被容器引用需要先删除容器```

容器查询

>docker ps -a

删除容器

> Docker rm 容器id

再执行上面的删除镜像操作







### mongodb

查看mongo可用版本

> docker  search mongo

获取最新mongo镜像

> docker pull mongo:latest

运行容器

```sh
docker run -itd --name mongo -p 27017:27017 mongo --auth
/*
-p 27017:27017 ：映射容器服务的 27017 端口到宿主机的 27017 端口。外部可以直接通过 宿主机 ip:27017 访问到 mongo 的服务。
--auth：需要密码才能访问容器服务。
*/

# 进入容器
docker exec -it 6bc1d4ce9136 /bin/sh;

```

停止容器

> docker stop mongo

启动容器

> docker start mongo







# docker 搭建生产环境

## docker 的安装

>https://docker_practice.gitee.io/zh-cn/install/centos.html

```bash
# 运行容器：
docker run -it 镜像名 /bin/bash

# 列出镜像
docker image ls

# 安装最新mysql镜像
docker pull mysql:latest
# 启动镜像
docker run -itd --name mysql3306 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=root1234 mysql
# mysql3306 镜像名
# root1234 mysql密码


# 安装redis
docker pull redis:latest
# 启动redis镜像
docker run -itd --name redis6379 -p 6379:6379 redis
#进入容器： 
docker exec -it 容器ID /bin/bash  
# redis-cli 连接 redis
docker exec -it 容器ID redis-cli
docker exec -it e1556b96ba1d redis-cli
auth 密码
# 设置redis连接密码
config set re quirepass 密码

# 退出容器：
exit 
# 查看容器：
docker ps -a
# 查看运行的容器：
docker ps
# 重启容器：
docker restart 容器ID
# 重启容器后进入交互式：
docker start -i 5c6ce895b979

```

