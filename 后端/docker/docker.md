# 使用模板
1. 服务器安装docker
   
    ```shell
    # 安装 最新 docker
    sudo yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
      
    # 启动& 开机启动docker； enable + start 二合一
    systemctl enable docker --now
      
    # 配置加速
    sudo mkdir -p /etc/docker
    sudo tee /etc/docker/daemon.json <<-'EOF'
    {
    "registry-mirrors": ["https://82m9ar63.mirror.aliyuncs.com"]
    }
    EOF
    sudo systemctl daemon-reload
    sudo systemctl restart docker
    ```

2. 运行
    1. 本地创建好镜像之后save
    ```shell
    docker save -o light-server.tar light-server
    ```
    2. 将tar上传到服务器

    3. 加载tar到服务器的image
    ```shell
    docker load -i light-server.tar
    ```
    4. 运行容器
    docker run -p 8080:8080 light-server

3. 调整安全组策略 
    0.0.0.0/0的入站为all



from b站 尚硅谷

## docker 简介

跨平台
    快速运行应用
    快速构建应用
    快速分享应用

### 容器化

images镜像 - 软件包
containers容器 - 运行的软件包 应用

理解容器
    ![alt text](https://zilong-blog-butterfly.oss-cn-shanghai.aliyuncs.com/article/image-1.png)
    **应用隔离** 的同时 不会拥有自己的操作系统，**共享操作系统内核** 
    类似轻量化的虚拟机，容器**拥有自己的文件系统**、内存、进程空间

## 安装

### 服务器

这里选择 腾讯云
连接工具选择 mobaXterm
###

```shell
# 移除旧版本docker
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine

# 配置docker yum源。
sudo yum install -y yum-utils
sudo yum-config-manager \
--add-repo \
http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo


# 安装 最新 docker
sudo yum install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# 启动& 开机启动docker； enable + start 二合一
systemctl enable docker --now

# 配置加速
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://82m9ar63.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
```

## 操作 与 实验 
1. 下载镜像
2. 启动容器
3. 修改页面
4. 保存镜像
5. 分享镜像

### 启动一个Nginx

- 下载命令： 
    镜像images相关
    - docker search 检索
    - docker pull 拉取镜像
        `docker pull nginx`
        特定版本，去dockerhub搜索
    - docker images 检查所有镜像
    - docker rmi 删除

### - 容器命令：
    container容器运行之后，需要用他的**id来进行指代**
    - `docker run` 运行 第一次启动
    - docker ps 查看运行容器
        ps -a 可以看到停止的
    - docker stop 停止
    - docker start 启动
    - docker restart 重启
    - docker stats 状态
        查看占用的cpu资源等
    - docker logs 日志
    - docker exec 进入容器
    - docker rm 删除

`docker run`补充
>docker为类虚拟机，需要使用 端口映射 将外部主机端口 映射到 内网主机 端口，这里为 -p `命令`
-d 为后台启动，放置占用当前窗口
`docker run -d --name mynginx -p 80:80 nginx`
外部主机端口不能重复，而内部可以重复，因为内部是独立的

`docker exec` 补充
> 操作容器
-it 交互模式，分配伪终端 /bin/bash 执行bash
`docker exec -it mynginx /bin/bash`

### - 保存镜像
docker commit 提交
    将容器运行内容提交为镜像image
    `docker commit -m "update index" mynginx mynginx:v1.0`
    这里将mynginx容器打包为mynginx:v1.0镜像
docker save 保存
    -o 表示输出到文件
    `docker save -o mynginx.tar mynginx:v1.0`
docker load 加载
    -i 加载文件
    `docker load -i mynginx.tar`

### 分享镜像
`docker login` 登录
`docker tag`  官方通过tag来区别
    用户名/名字:tag，
    这里tag如果写latest方便下载
`docker push` 即可

## 存储
### 目录挂载
由于确实基本编辑工具等，并且数据容易丢失
在容器内进行相关操作有诸多不便

目录挂载 将主机的目录加载到容器目录
将/data/html映射到nginx/html
外部目录会自动创建，都为空内容
```bash
docker run --name=mynginx   \
-d  --restart=always \
-p  88:80 -v /data/html:/usr/share/nginx/html:ro  \
nginx

# 修改页面只需要去 主机的 /data/html
```

### 卷映射
映射时复制内部内容，修改时二者同步
```bash
# docker 经常修改nginx配置文件
docker run -d -p 80:80 \
-v /data/html:/usr/share/nginx/html \
-v ngconfig:/etc/nginx \
--name mynginx-02 \
nginx
```
外部为ngconfig，在docker volume ls中可以查看
在删除容器时，外部的卷内容不会消失

## docker 网络
容器在内部之前有一个通用网络，默认为docker0
docker为容器分配唯一ip，使用容器ip+端口互相访问

由于ip各种原因可能会变化，使用域名来解决，类似baidu等
但是docker0不支持域名

- docker network 网络相关操作
    `docker netwoek create mynet`
    创建mynet自定义网络 -p 外部：内部
    `docker run -d -p 88:80 --name app1 --network mynet nginx`
    `docker run -d -p 99:80 --name app2 --network mynet nginx`

    现在使用容器名+端口即可访问

### redis集群模拟

![alt text](https://zilong-blog-butterfly.oss-cn-shanghai.aliyuncs.com/article/image-2.png)

```shell
#自定义网络
docker network create mynet
#主节点
docker run -d -p 6379:6379 \
-v /app/rd1:/bitnami/redis/data \
-e REDIS_REPLICATION_MODE=master \
-e REDIS_PASSWORD=123456 \
--network mynet --name redis01 \
bitnami/redis

#从节点
docker run -d -p 6380:6379 \
-v /app/rd2:/bitnami/redis/data \
-e REDIS_REPLICATION_MODE=slave \
-e REDIS_MASTER_HOST=redis01 \
-e REDIS_MASTER_PORT_NUMBER=6379 \
-e REDIS_MASTER_PASSWORD=123456 \
-e REDIS_PASSWORD=123456 \
--network mynet --name redis02 \
bitnami/redis
```

其他信息：[尚硅谷笔记](https://www.yuque.com/leifengyang/sutong/au0lv3sv3eldsmn8)

[compose官方文件](https://docs.docker.com/compose/compose-file/)



```dockerfile
FROM golang:1.21
LABEL authors="wang2"

WORKDIR /mp_server

COPY go.mod go.sum ./

RUN go mod download

COPY . /mp_server

RUN CGO_ENABLED=0 GOOS=linux go build -o /docker_mp_server

CMD ["/docker_mp_server","api"]

```

轻量化后版本
```dockerfile 
# 第一阶段：构建阶段
FROM golang:1.21 AS builder

LABEL authors="wang2"

WORKDIR /mp_server

# 复制依赖文件并下载依赖项
COPY go.mod go.sum ./
RUN go mod download

# 复制源代码并构建
COPY . .
RUN CGO_ENABLED=0 GOOS=linux go build -o /docker_mp_server

# 第二阶段：运行阶段
FROM alpine:latest

WORKDIR /root/

# 复制构建阶段的可执行文件到运行阶段
COPY --from=builder /docker_mp_server .
COPY --from=builder /mp_server/config/config.dev.toml ./config/config.dev.toml

# 设置运行命令
CMD ["./docker_mp_server", "api"]

```

## compose
在 项目目录下 新建 compose.yaml
```yaml
name: mp_server

services:
  server:
    image: light-server
    ports:
      - "8080:8080"

```