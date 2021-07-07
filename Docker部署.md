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
- 查看容器启动后的输出内容（排错）`docker logs 容器id或名称`

docker容器内的第一个进程（初始命令）必须一直处于前台运行的状态，否则这个容器就会出现退出状态

---



## 小案例

---

将宿主机80端口映射到容器80，81端口映射到容器81，并用自己的配置启动nginx

```shell
docker run -d -p 80:80 -p 81:81 -v /opt/Bird:/data -v /opt/Bird.conf:/etc/nginx/conf.d/Bird.conf nginx:last
```

---

配置apache代理php启动kodexplore

```shell
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/centos-vault/
yum install php unzip php-gd php-mbstring -y
cd /var/www/html/
curl -o kodexplorer4.40.zip http://static.kodcloud.com/update/download/kodexplorer4.40.zip
unzip kodexplorer4.40.zip
chown -R apache:apache .
```

改为Dockerfile

```shell
FROM centos:6.9
RUN curl -o /etc/yum.repos.d/CentOS-Base.repo https://static.lty.fun/%E5%85%B6%E4%BB%96%E8%B5%84%E6%BA%90/SourcesList/Centos-6-Vault-Aliyun.repo
RUN yum install php unzip php-gd php-mbstring -y
WORKDIR /var/www/html/
ADD kodexplorer4.40.zip .
RUN unzip kodexplorer4.40.zip
RUN chown -R apache:apache .
ADD init.sh /init.sh
CMD ["/bin/bash","/init.sh"]
```

初始化脚本

```shell
#!/bin/bash

service httpd start
tail -F /var/log/httpd/access_log
```

构建镜像`docker build -t kod:v1 .`

启动容器`docker run -d -p 88:80 kod:v1`

## 手动制作镜像

1. 启动一个基础容器`docker run -it centos:6.9`
2. 在容器中安装服务
3. 把已经安装好服务的容器提交为镜像`docker commit 容器id 输出的镜像名字:标签`

## Dockerfile自动构建docker镜像

- 用Dockerfile构建镜像`docker build -t 构建后的容器名称:标签 Dockerfile的目录`
- 使用Dockerfile构建的过程中，每一个 RUN 中的命令执行的时候都会构建一个临时的容器，执行完后基于当前的临时容器执行下一条 RUN 命令并一个新的临时容器同时删除之前的临时容器，而每次创建容器的时候为了变更容器主机名都会覆盖`/etc/resolv.conf、/etc/hostname、/etc/hosts`这三个文件，因此如果后续 RUN 中的命令要用到这三个文件被之前的 RUN 修改后的数据，需要把后续命令用`&&`连接为一条命令执行
- Dockerfile主要组成部分:
  - 基础镜像信息               FROM    centos:6.9
  - 制作镜像操作指令        RUN    yum install openssh-server -y
  - 容器启动时执行指令    CMD    ["/bin/bash", "/init.sh"]
- Dockerfile常用指令
  - FROM 这个镜像的基础镜像
  - MANITAINER 指定维护者信息，可选
  - LABLE 描述，标签
  - RUN 制作镜像时需要做的事情
  - ADD 将宿主机上的某个文件copy到容器里(tar 自动解压) 
  - WORKDIR 设置当前工作目录，不设置默认是`/`开始
  - VOLUME 设置卷，挂载宿主机目录
  - EXPOSE 指定对外的端口
  - CMD 指定容器启动后要干的事情（容易被替换）
- Dockerfile其他指令
  - COPY 与ADD相同但不会自动解压tar
  - ENV 环境变量
  - ENTRYPOINT 容器启动后执行的命令（无法被替换），并把启动时指定的自己指定的命令作为其参数

