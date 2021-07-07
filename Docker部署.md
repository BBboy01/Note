## 容器的常用管理命令

1. docker的镜像管理

- 搜索镜像`docker search xxx`

- 获取镜像`docker pull xxx:xx`
  - 不指定版本时默认安装`latest`
  - 官方pull`docker pull centos:6.8`
  - 私有仓库pull`docker pull daoclub.io/huangzhichong/alpine-cn:latest`

- 查看镜像列表`docker images`、`docker image ls`
- 删除镜像`docker image rm centos:latest`
- 导出镜像`docker image save centos -o docker-centos7.4.tar.gz`
- 导入镜像`docker image load -i docker-centos7.4.tar.gz`

---

2. docker的容器管理

- 创建并运行一个容器`docker run -d -p 80:456 --name myNginx nginx:latest /bin/sh`
  - `-d` 放在后台 `-it` 分配交互式的终端
  - `-p 宿主机端口：容器端口` 将容器端口映射到宿主机上
  - `--name 容器名称` 指定运行的容器名称
  - `/bin/sh` 覆盖容器的初始命令
  - `-v 宿主机目录：容器目录 / 宿主机文件：容器文件` 将宿主机目录挂载到容器中（目录不存在则创建，参数类型要一样）
    - 当第一个参数不是一个目录时，例如`-v monkey:/usr/share/nginx/html`，则会创建一个`volume`。
    - `docker volume ls` 查看当前所有的卷
    - 使用`docker volume inspect monkey`查看该容器的详细信息，其中`"Mountpoint"`为该卷挂载在宿主机上的位置。
    - 当相关容器删除后，该卷依旧存在
  - `-e "NAME=VALUE"` 为容器添加环境变量
- 启动容器
  - `docker run 容器名称`
  - `docker run -it 容器名称 初始命令`
  - `docker run = docker create + docker start`
- 停止容器`docker stop 容器id或名称`
- 杀死容器`docker kill 容器名称`
- 查看容器列表`docker ps`、`docker ps -a`
- 查看最新运行的容器`docker ps -a -l`
- 进入容器（目的：调试、排错）
  - `docker exec -it 容器id或名称 初始命令` 不共享终端
  - `docker attach 容器id或名称` 共享终端
- 删除容器`docker rm 容器id或名称`
  - `-f` 连运行中的容器也一同删除
- 批量删除容器 docker rm -f  &#96; docker ps -a -q &#96;

docker容器内的第一个进程（初始命令）必须一直处于前台运行的状态，否则这个容器就会出现退出状态

---



## 小案例

将宿主机80端口映射到容器80，81端口映射到容器81，并用自己的配置启动nginx

```shell
docker run -d -p 80:80 -p 81:81 -v /opt/Bird:/data -v /opt/Bird.conf:/etc/nginx/conf.d/Bird.conf nginx:last
```

## 手动制作镜像

1. 启动一个基础容器`docker run -it centos:6.9`
2. 在容器中安装服务
3. 把已经安装好服务的容器提交为镜像`docker commit 容器id 输出的镜像名字:标签`

## Dockerfile自动构建docker镜像

- 用Dockerfile构建镜像`docker build -t 构建后的容器名称:标签 Dockerfile的目录`
- 使用Dockerfile构建的过程中，每一个 RUN 中的命令执行的时候都会构建一个临时的容器，执行完后基于当前的临时容器执行下一条 RUN 命令并一个新的临时容器同时删除之前的临时容器，而每次创建容器的时候为了变更容器主机名都会覆盖`/etc/resolv.conf、/etc/hostname、/etc/hosts`这三个文件，因此如果后续 RUN 中的命令要用到这三个文件被之前的 RUN 修改后的数据，需要把后续命令用`&&`连接为一条命令执行
- Dockerfile主要组成部分:
  - 基础镜像信息            FROM    centos:6.9
  - 制作镜像操作指令     RUN    yum install openssh-server -y
  - 容器启动时执行指令  CMD    ["/bin/bash"]