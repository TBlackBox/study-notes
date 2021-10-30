# docker 客户端
查看docker客户端的所有命令`docker`
```shell
[root@iZbp17nybdeu4tpvlzfbjbZ ~]# docker

Usage:  docker [OPTIONS] COMMAND

A self-sufficient runtime for containers

Options:
      --config string      Location of client config files (default "/root/.docker")
  -c, --context string     Name of the context to use to connect to the daemon (overrides DOCKER_HOST env var and default context set
                           with "docker context use")
  -D, --debug              Enable debug mode
  -H, --host list          Daemon socket(s) to connect to
  -l, --log-level string   Set the logging level ("debug"|"info"|"warn"|"error"|"fatal") (default "info")
      --tls                Use TLS; implied by --tlsverify
      --tlscacert string   Trust certs signed only by this CA (default "/root/.docker/ca.pem")
      --tlscert string     Path to TLS certificate file (default "/root/.docker/cert.pem")
      --tlskey string      Path to TLS key file (default "/root/.docker/key.pem")
      --tlsverify          Use TLS and verify the remote
  -v, --version            Print version information and quit

Management Commands:
  app*        Docker App (Docker Inc., v0.9.1-beta3)
  builder     Manage builds
  buildx*     Build with BuildKit (Docker Inc., v0.6.1-docker)
  config      Manage Docker configs
  container   Manage containers
  context     Manage contexts
  image       Manage images
  manifest    Manage Docker image manifests and manifest lists
  network     Manage networks
  node        Manage Swarm nodes
  plugin      Manage plugins
  scan*       Docker Scan (Docker Inc., v0.8.0)
  secret      Manage Docker secrets
  service     Manage services
  stack       Manage Docker stacks
  swarm       Manage Swarm
  system      Manage Docker
  trust       Manage trust on Docker images
  volume      Manage volumes

Commands:
  attach      Attach local standard input, output, and error streams to a running container
  build       Build an image from a Dockerfile
  commit      Create a new image from a container's changes
  cp          Copy files/folders between a container and the local filesystem
  create      Create a new container
  diff        Inspect changes to files or directories on a container's filesystem
  events      Get real time events from the server
  exec        Run a command in a running container
  export      Export a container's filesystem as a tar archive
  history     Show the history of an image
  images      List images
  import      Import the contents from a tarball to create a filesystem image
  info        Display system-wide information
  inspect     Return low-level information on Docker objects
  kill        Kill one or more running containers
  load        Load an image from a tar archive or STDIN
  login       Log in to a Docker registry
  logout      Log out from a Docker registry
  logs        Fetch the logs of a container
  pause       Pause all processes within one or more containers
  port        List port mappings or a specific mapping for the container
  ps          List containers
  pull        Pull an image or a repository from a registry
  push        Push an image or a repository to a registry
  rename      Rename a container
  restart     Restart one or more containers
  rm          Remove one or more containers
  rmi         Remove one or more images
  run         Run a command in a new container
  save        Save one or more images to a tar archive (streamed to STDOUT by default)
  search      Search the Docker Hub for images
  start       Start one or more stopped containers
  stats       Display a live stream of container(s) resource usage statistics
  stop        Stop one or more running containers
  tag         Create a tag TARGET_IMAGE that refers to SOURCE_IMAGE
  top         Display the running processes of a container
  unpause     Unpause all processes within one or more containers
  update      Update configuration of one or more containers
  version     Show the Docker version information
  wait        Block until one or more containers stop, then print their exit codes

Run 'docker COMMAND --help' for more information on a command.

To get more help with docker, check out our guides at https://docs.docker.com/go/guides/
```

具体命令的帮助

```shell
 docker command --help 
 //例如
 docker stats --help
```



# 使用

### 查看容器和镜像
```shell
docker ps

//也可以使用
docker ps -a

//查询最后一次创建的容器
docker ps -l 
```

输出详情介绍：

**CONTAINER ID:** 容器 ID。

**IMAGE:** 使用的镜像。

**COMMAND:** 启动容器时运行的命令。

**CREATED:** 容器的创建时间。

**STATUS:** 容器状态。

状态有7种：

- created（已创建）
- restarting（重启中）
- running 或 Up（运行中）
- removing（迁移中）
- paused（暂停）
- exited（停止）
- dead（死亡）

**PORTS:** 容器的端口信息和使用的连接类型（tcp\udp）。

**NAMES:** 自动分配的容器名称。

在宿主主机内使用 **docker logs** 命令，查看容器内的标准输出，可以用来看日志。

```shell
dockers logs 容器的id或者容器的名字
```

### 获取镜像

- 如果我们本地没有 ubuntu 镜像，我们可以使用 docker pull 命令来载入 ubuntu 镜像

  ```
  docker pull ubuntu
  ```
### 启动镜像
使用 ubuntu 镜像启动一个容器，参数为以命令行模式进入该容器：
```
docker run -i -t ubuntu /bin/bash
```

参数说明：

- **-i**: 交互式操作。
- **-t**: 终端。
- **ubuntu**: ubuntu 镜像。
- **/bin/bash**：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 /bin/bash。

**通过输入exit**

### 启动停止容器
```shell
//启动容器
docker start 容器ID
//后台运行 -d指定容器的模式
docker run -itd --name ubuntu-test ubuntu /bin/bash

//停止容器
docker stop <容器 ID>

//重启容器
docker restart <容器 ID>
```

### 进入容器

在使用 **-d** 参数时，容器启动后会进入后台。此时想要进入容器，可以通过以下指令进入：

- **docker attach**
- **docker exec**：推荐大家使用 docker exec 命令，因为此退出容器终端，不会导致容器的停止。

```shell
//这种方式进入容器 注意： 如果从这个容器退出，会导致容器的停止。
docker attach 容器ID

//这种方式进入容器  exit 退出容器 这种方式不会导致容器停止  推荐使用这是
docker exec -it 容器ID 对应的命令

```

### 导入导出容器
```shell
//导出容器
docker export 容器ID 
//例如
//导出容器 1e560fca3906 快照到本地文件 ubuntu.tar    这样将导出容器快照到本地文件。
docker export 1e560fca3906 > ubuntu.tar


//导入容器快照
//可以使用 docker import 从容器快照文件中再导入为镜像
以下实例将快照文件 ubuntu.tar 导入到镜像 test/ubuntu:v1:
cat docker/ubuntu.tar | docker import - test/ubuntu:v1

//指定url或者目录导入
docker import http://example.com/exampleimage.tgz example/imagerepo
```

### 删除容器
```shell
docker rm -f  容器ID

//清除所有处于终止状态的容器
docker container prune
```

## 检查 WEB 应用程序

使用 **docker inspect** 来查看 Docker 的底层信息。它会返回一个 JSON 文件记录着 Docker 容器的配置和状态信息。

```shell
docker inspect 容器id或者名字
```

## 查看WEB应用程序容器的进程

我们还可以使用 docker top 来查看容器内部运行的进程

```shell
docker top 容器id或者名字
```

### 查看容器的网络端口

```
docker port 容器id或者名字
```

### 查看容器应用程序的日志

```shell
docker logs -f 容器ID或名字

//-f: 让 docker logs 像使用 tail -f 一样来输出容器内部的标准输出。
```

