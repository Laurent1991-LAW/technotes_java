



# Docker指令

systemctl start docker	——启动docker

systemctl status docker	——查看运行状态

systemctl enable docker	——开机自启

systemctl disable docker	——禁用开机自启

systemctl restart docker	——重启

docker info		——查看docker信息

docker info | grep 'Docker Root Dir:'	——查看docker info中具体key的信息

systemctl stop docker		——关闭docker

docker pull 镜像名		—— 下载镜像，如docker pull centos:7

docker images 	——查看镜像文件

docker history 镜像名

docker image rm 镜像名

docker load < hello-world.tar.gz  

——镜像导入（要在hello-world.tar.gz 文件所在目录下执行）

docker inspect 镜像名		——查看镜像详情

docker run -it centos:7 bash		——创建并启动容器(Container)

docker run -d --name sentinel8181 -p 8181:8080 sentinel:8

```
-d  在后台运行容器
--name  命名为sentinel8181
-p端口映射
sentinel:8镜像名
```

docker ps	——查看docker运行中的容器

docker ps -a / --all			——查看所有容器

docker container logs 802		——查看容器日志（启动了，PS看不到=>查看日志）

docker container start 802		—— 802启动容器

docker container stop 802  	—— #802为容器自己的id

docker container restart 802 		—— #802位容器自己的id

docker exec -it 802 bash 		 ——进入指定容器，#802为容器id

exit		——退出容器

docker container rm 802 		——删除容器，#802为容器id

docker container rm -f 802 		——强制删除容器，#802为容器id

docker container prune	——清除所有停止运行的容器

docker exec -it redis01 redis-cli  进入redis容器打开redis客户端

docker volume create container-vol	——创建数据卷container-vol

docker volume rm container-vol	——创建数据卷container-vol

docker volume ls				——查看所有数据卷

docker volume inspect container-vol		——查看指定 数据卷 的信息

docker volume prune		——清理无主数据卷

docker inspect 91a 		—— #91a 为容器id，其中可查看"Mounts"，即容器挂载点

docker run -it --mount source=container-vol,target=/root centos:7 bash

-it 交互模式挂载	bash 终端

省略source宿主机 / target容器

docker run -it -v container-vol:/root centos:7 bash

touch hello.java	—— 直接创建文件hello.java

vim hello.java		 —— 不存在会创建，并进入hello.java	