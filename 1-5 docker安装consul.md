# Docker本地部署consul集群

微服务涉及到服务发现，可用的有etcd consul zp等，先看看consul怎么在docker搭建，在网上找了一下相关的内容，先将其用起来再慢慢学习

## 集群安装

- 拉取镜像

```shell
docker pull consul
Using default tag: latest
latest: Pulling from library/consul
540db60ca938: Already exists
98d468a8c3b4: Pull complete
105d8515b221: Pull complete
c0ed6bc2e594: Pull complete
a1981893a8c5: Pull complete
72b39d63083e: Pull complete
Digest: sha256:c58e1f8d4f13eb4e2155a2830d55380ab3f66d8588e21bb135a75d803975da91
Status: Downloaded newer image for consul:latest
docker.io/library/consul:latest
```

- 运行服务

```shell
docker run --name consul1 -d -p 8500:8500 -p 8300:8300 -p 8301:8301 -p 8302:8302 -p 8600:8600 consul:1.2.2 agent -server -bootstrap-expect 2 -ui -bind=0.0.0.0 -client=0.0.0.0
Unable to find image 'consul:1.2.2' locally
1.2.2: Pulling from library/consul
c67f3896b22c: Pull complete
51cfb00081e2: Pull complete
0a92afe431c9: Pull complete
87262d12c26b: Pull complete
3144a0a0b402: Pull complete
4f48afb7673f: Pull complete
Digest: sha256:8603f0d1b2278364ecb7c11068a477b1ea648df735eda8791362063aba99656a
Status: Downloaded newer image for consul:1.2.2
29d53392f4a253f758f50993225ce6498f291dc4c17668e92758c1496e05ff44
```

```shell
docker inspect --format '{{ .NetworkSettings.IPAddress }}' consul1
172.17.0.3
```

- 启动多个consul服务部署集群

```shell
docker run --name consul3 -d -p 8502:8500 consul agent -server -ui -bind=0.0.0.0 -client=0.0.0.0 -join 172.17.0.3
docker run --name consul2 -d -p 8503:8500 consul agent -server -ui -bind=0.0.0.0 -client=0.0.0.0 -join 172.17.0.3
docker run --name consul4 -d -p 8504:8500 consul agent -server -ui -bind=0.0.0.0 -client=0.0.0.0 -join 172.17.0.3
docker run --name consul5 -d -p 8505:8500 consul agent -server -ui -bind=0.0.0.0 -client=0.0.0.0 -join 172.17.0.3
```

```shell
docker ps -a

CONTAINER ID   IMAGE               COMMAND                  CREATED         STATUS                    PORTS                                                                                                                                                                          NAMES
389275ab2c11   consul              "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes              8300-8302/tcp, 8301-8302/udp, 8600/tcp, 8600/udp, 0.0.0.0:8504->8500/tcp, :::8504->8500/tcp                                                                                    consul5
48251b38e5d0   consul              "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes              8300-8302/tcp, 8301-8302/udp, 8600/tcp, 8600/udp, 0.0.0.0:8503->8500/tcp, :::8503->8500/tcp                                                                                    consul4
c17c3d3272fa   consul              "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes              8300-8302/tcp, 8301-8302/udp, 8600/tcp, 8600/udp, 0.0.0.0:8502->8500/tcp, :::8502->8500/tcp                                                                                    consul2
2a70c342fda6   consul              "docker-entrypoint.s…"   4 minutes ago   Up 3 minutes              8300-8302/tcp, 8301-8302/udp, 8600/tcp, 8600/udp, 0.0.0.0:8501->8500/tcp, :::8501->8500/tcp                                                                                    consul3
29d53392f4a2   consul:1.2.2        "docker-entrypoint.s…"   6 minutes ago   Up 6 minutes              0.0.0.0:8300-8302->8300-8302/tcp, :::8300-8302->8300-8302/tcp, 8301-8302/udp, 0.0.0.0:8500->8500/tcp, :::8500->8500/tcp, 0.0.0.0:8600->8600/tcp, :::8600->8600/tcp, 8600/udp   consul1
```

- web页面查看consul service
<http://localhost:8500>
web页面将可以查看各个服务的运行状态，总共启动了5个服务
