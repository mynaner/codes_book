## docker

查看本地镜像

> docker image

查看本地启动镜像

> docker ps



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
```

停止容器

> docker stop mongo

启动容器

> docker start mongo

