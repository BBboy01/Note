## 安装

```shell
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum install docker-ce docker-ce-cli containerd.io -y

mkdir /etc/docker

tee /etc/docker/daemon.json << 'EOF'
{
	"registry-mirrors": ["https://registry.docker-cn.com"]
}
EOF

systemctl daemon-reload
systemctl restart docker
```

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
  - `-P 容器端口` 将容器端口随机映射到宿主机上的某一个端口
  - `--name 容器名称` 指定运行的容器名称
  - `/bin/sh` 覆盖容器的初始命令
  - `-v 宿主机目录：容器目录 / 宿主机文件：容器文件` 将宿主机目录挂载到容器中（目录不存在则创建，参数类型要一样）
    - 当第一个参数不是一个目录时，例如`-v monkey:/usr/share/nginx/html`，则会创建一个`volume`。
    - `docker volume ls` 查看当前所有的卷
    - 使用`docker volume inspect monkey`查看该容器的详细信息，其中`"Mountpoint"`为该卷挂载在宿主机上的位置。
    - 当相关容器删除后，该卷依旧存在
  - `-e "NAME=VALUE"` 为容器添加环境变量
  - `-h 名字` 指定进入容器时显示的主机名
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
- 查看容器的端口映射`docker port 容器id`
- 将本地的文件拷贝到容器中`docker cp 本地文件 容器名称:目录`

docker容器内的第一个进程（初始命令）必须一直处于前台运行的状态，否则这个容器就会出现退出状态

## 重启docker容器

当docker重启之后没有设置重启的容器全部都会退出

- 重启指定容器`docker run -d --restart=always nginx`

- 设置所有容器都自动重启`vim /etc/docker/daemon.json` 但是当容器重新启动后容器虽然启动着，**但是关闭docker的时候容器也一直处于启动状态，所以不建议使用**

  ```json
  {
    "live-restore": true
  }
  ```

## 小案例

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

```dockerfile
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
  - EXPOSE 声明容器运行的服务端口
  - CMD 指定容器启动后要干的事情（容易被替换）
- Dockerfile其他指令
  - COPY 与ADD相同但不会自动解压tar
  - ENV 环境变量
  - ENTRYPOINT 容器启动后执行的命令（无法被替换），并把启动时指定的自己指定的命令作为其参数
- 构建的时候可以对一些缓存进行清理来减少构建后的镜像的大小`yum clean all && rm -rf /var/cache/yum/*`

使用 ENRTYPOINT 指定的命令需要在运行容器时通过 --entrypoint 来覆盖, 通常使用 ENTRYPOINT 指定固定命令, CMD 指定参数

## 文件的分层

- docker在生成新的镜像的时候是分层生成的，分层则是根据之前对镜像进行文件大小变化的记录进行分层，无文件大小变化的操作作为容器的属性不在分层中，使用`docker history 镜像id或名称`查看历史对该镜像的所有操作。

- 分层是除了最原始的镜像，后面添加的镜像都会有一个parent属性标记着父层的目录

- 分层后的镜像都是复用上一次镜像内容，因此只保留改变的数据，当运行该镜像为容器的时候，则会merge为一个整体

- 分层的好处便是可以复用，节省磁盘空间，相同的内容只需加载一份到内存里。

- 更改dockerfile时尽量在最后加新内容，这样可以尽可能少的改变分层的结构，没被破坏的地方直接走上一次打包镜像的缓存

## 容器的互联（--link 单向）

docker中每建立一个容器都会创建两个虚拟网卡，一个为容器的，一个为桥接到`docker0`的，因此主机可以`ping`通该容器。多个容器也都会通过桥接`docker0`实现互相访问，但是只能通过ip地址进行访问。而使用`--link`可以使用名称来实现容器之间的访问

`docker run -d --name WEB-p 80:80 nginx`

`docker run -it --link 容器名称:别名 centos-ssh /bin/bash`

`ping WEB`

实际上是在后面的容器的`/etc/hosts`文件中添加了一条`--link`后面的容器的 ip ，可以通过别名、容器名称、容器id来快捷连接

## 上传镜像到私有仓库

如果上传或下载的私有仓库地址不是HTTPS的，则需要更改`/etc/docker/daemon.json`，添加

```json
{
	"insecure-registries": ["10.0.0.11:5000"]
}
```

之后`systemctl restart docker`

---

- 创建私有仓库(10.0.0.11)

  - `docker run -d -p 5000:5000 --restart=always --name=registry -v /opt/myregistry:/var/lib/registry registry`

- 给镜像打标签

  - `docker tag nginx:v2 10.0.0.11:5000/nginx:v2`

- 上传镜像

  - `docker push 10.0.0.11:5000/nginx:v2`

- 拉取镜像

  - `docker pull 10.0.0.11:5000/nginx:v2`

- 创建带basic认证的私有仓库

  - ```shell
    yum install httpd-tools -y
    mkdir /opt/registry-var/auth/ -p
    htpasswd -Bbn oldboy 123456 » /opt/registry-var/auth/htpasswd
    
    docker run -d -p 5000:5000 -v /opt/registry-var/auth/:/auth/ -v /opt/myregistry:/var/lib/registry -e "REGISTRY_AUTH=htpasswd" -e "REGISTRY_AUTH_HTPASSWD_REALH=Registry Realm" -e "REGISTRY_AUTH_HTPASSWD_PATH=/auth/htpasswd" registry
    ```
    登录`docker login -uoldboy -p123456 10.0.0.11:5000`

## Docker网络类型

查看docker启动的网段`docker network ls`

查看网段详情`docker network inspect 网段名称或id`，里面会显示这个网段中每个容器的IP信息

- none 不为容器配置任何网络功能  `--network none`
- container 与另一个运行中的容器共享主机名、IP、端口 `--network container:容器id`
- host 与宿主机共享主机名、IP、端口 `--network host`
- bridge （默认）桥接的方式通过NAT转换使用网络功能

## 自己创建网络

- `docekr network create --dirver bridge --subnet 192.168.0.0/16 --gateway 192.168.0.1 myNet`

- 在自定义的网络下启动容器

  - ```shell
    docker run -d -P --name tomcat-net-01 --net myNet tomcat
    docker run -d -P --name tomcat-net-02 --net myNet tomcat
    ```

- 此时`docker network inspect myNet`会出现tomcat-net-01和tomcat-net-02的信息

- 并且此时两个容器**可以使用名称互相ping通**，推荐在不同的集群中使用不同的网络，保证集群是安全和健康的

## 不同网络间网络的连通

- `docker connect myNet tomcat01`
- 之后docker便会将tomcat01放到myNet网络下

## docker compose批量启动容器

- 编写`docker-compose.yaml`

  ```shell
  version: '3'
  
  services:
    db:
      image: mysql:5.7
      volumes:
        - db_data:/var/lib/mysql
      restart: always
      environment:
        MYSQL_ROOT_PASSWORD: somewordpress
        MYSQL_DATABASE: wordpress
        MYSQL_USER: wordpress
        MYSQL_PASSWORD: wordpress
  
    wordpress:
      depends_on:
        - db
      image: wordpress:latest
      volumes:
        - web_data:/var/www/html
      ports:
        - "80"
      restart: always
      environment:
        WORDPRESS_DB_HOST: db:3306
        WORDPRESS_DB_USER: wordpress
        WORDPRESS_DB_PASSWORD: wordpress
    volumes:
      db_data:
      web_data:
  ```

- 启动`docker compose up` 不指定 -f 默认启动文件为 `docker-compose.yaml`

- 后台启动`docker compose up -d`

- 某个容器启动多个`docker compose up -d --scale wordpress=3`

- 启动后的容器名称为：`当前所处的文件夹名称 + services中起的名称`，如果启动多个则会在后面拼上`_1、_2`

- 启动后会为这些批量创建的容器创建一个docker网段

此时使用`netstat -lntup`查看占用的端口便会看到3个wordpress服务占用了宿主机3个随机的端口映射到wordpress的80口上

![image-20210708131427253](https://gitee.com/RealBBboy/mark-down-images-repo/raw/master/NoteImg/image-20210708131427253.png)

此时便可以使用**Nginx**做负载均衡`vim /etc/nginx/nginx.conf`

```nginx
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    upstream wordpress {
        server 10.0.0.11:32768;
        server 10.0.0.11:32769;
        server 10.0.0.11:32770;
    }
    server {
        listen       80;
        server_name  localhost;
        location / {
            proxy_pass http://wordpress;
            proxy_set_header Host $host;
        }
    }
}
```

