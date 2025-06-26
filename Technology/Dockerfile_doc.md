# Dockerfile

参考：https://docs.docker.com/reference/dockerfile

**Dockerfile** 是一个文本文件，用于定义如何构建 Docker 镜像。它包含一系列指令（Instructions），这些指令会按顺序执行，最终生成一个可运行的容器镜像。当定义好一个 `Dockerfile` 文件的时候，运行:

 ```
docker build -t <镜像名>:<标签> <PATH>
 ```

程序会自动在 `<PATH>` 中找到 `Dockerfile` 文件，并按照内容创建名为 `<镜像名>:<标签>` 的镜像。

## 1. Overview

`Dokerfile` 中的常用命令包括 `ADD`、`ARG`、`CMD`、`COPY`、`ENTRYPOINT`、`ENV`、`EXPOSE`、`FROM`、`LABEL`、`RUN`、`SHELL`、`USER`、`volume`、`workdir` 等。

`Dokerfile` 的语法是 “大小写不敏感” 的，但是为了更好的阅读性，通常指定用大写字母表示，参数用小写字母表示：

```dockerfile
# Comment
RUN echo 'we are running some # of cool things'
```

## 2. 环境变量

`Dockerfile` 中，环境变量的格式是  `$variable_name` or `${variable_name}`，并且支持标准的 `bash` 语法：

- `${variable:-word}`：当 `variable` 不存在时返回 `word`，否则返回 `variable`；
- `${variable:+word}`：当 `variable` 存在时返回 `result`，否则返回空字符串。

## 3. `.dockerignore`

在构建镜像的时候，在 Docker 构建上下文目录中制作一个 `.dockerignore` 文件，可以在构建过程中用来排除不需要的文件或目录的配置文件，这些文件将不会被拷贝到镜像内。

## 4. 命令解析

### Shell 和 exec

`RUN`, `CMD`, and `ENTRYPOINT` 三个命令都支持 2 中格式：

- exec格式：`INSTRUCTION ["executable","param1","param2"]`，Docker 不会通过 shell 解释命令，而是直接调用可执行程序。如：`CMD ["python3", "app.py", "--config", "config.yaml"]`
- Shell 格式：`INSTRUCTION command param1 param2`，Docker 会使用默认 shell（Linux 是 `/bin/sh -c`，Windows 是 `cmd /S /C`）执行命令。如：`RUN echo "Hello World"`

#### shell 命令的串联写法

- 方法1：通过 `&&` 进行串联：

  ```Docker
  RUN apt-get update && \
      apt-get install -y curl && \
      rm -rf /var/lib/apt/lists/*
  ```

- 方法2：使用 `<<EOF ... EOF`.

  ```dockerfile
  RUN <<EOF
  apt-get update
  apt-get install -y curl
  EOF
  ```

### `RUN`, `CMD` 和 `ENTRYPOINT`

- `RUN` 命令主要用于构建镜像时执行安装、配置命令。

- `CMD` 命令主要作用于容器运行阶段，定义一个**默认命令/参数**，如果 `docker run` 没有指定命令，则执行它。

- `ENTRYPOINT` 命令也是主要作用于容器运行阶段，不会被  `docker run` 指定的命令覆盖。除非通过`--entrypoint` 命令修改。

通常 `RUN` 命令使用 shell 格式，`CMD` 和 `ENTRYPOINT` 使用 exec 格式。

通常的做法是，用 `ENTRYPOINT` 定义容器运行时的应用程序，用 `CMD` 定义传入的默认参数，如：

```dockerfile
ENTRYPOINT ["python3", "app.py"]
CMD ["--config=default.yaml"]
```

这里的 `CMD` 可以被 `docker run myimage --debug=true ` 指定的参数覆盖。

### FROM

```dockerfile
FROM [--platform=<platform>] <image> [AS <name>]
# Or
FROM [--platform=<platform>] <image>[:<tag>] [AS <name>]
# Or
FROM [--platform=<platform>] <image>[@<digest>] [AS <name>]
```

### LABEL

`LABEL` 是一个用于为镜像添加**元数据（metadata）**的指令。它以键值对（`key=value`）的形式存储信息，例如镜像的作者、版本、描述等。这些元数据不会影响容器的运行，但可以帮助开发者或工具更好地管理和理解镜像的用途。

```dockerfile
LABEL maintainer="admin@example.com"
LABEL version="1.0" description="This is a demo image"
```

可以通过 `docker inspect <镜像名>` 查看一个镜像的标签内容。

### EXPOSE

`EXPOSE` 命令用于指定容器监听的断口，默认是是 `TCP` 协议下的端口。

```dockerfile
EXPOSE 80
# 指定 UDP 协议端口
EXPOSE 80/udp
```

### ARG 和 ENV

- `ARG` ：定义构建镜像过程中使用的变量（**构建时变量**），在构建镜像的时候可以通过 `--build-arg` 传入；
- `ENF`：主要用于定义镜像和容器运行时可用的**环境变量**（**运行时变量**），在运行容器时可以通过 `-e` 传入。

两者也可以联合使用，如：

```dockerfile
ARG VERSION=1.0
ENV APP_VERSION=$VERSION
```

### VOLUME

`VOLUME` 用于指定卷的挂在路径。当运行容器时，如果没有指定该目录的挂载，docker 就会在每次运行容器的时候都自动生成一个卷，这些卷并不会被自动删除。

```shell
$ docker volume ls
DRIVER    VOLUME NAME
local     4b253aed2ff61473bf2eb5056299621c269f9e70554388c7325157fd02708517
local     6f491caffe96f3f04ea2a809ec097e9e2f11802aef0fb4bc3c44034fa181863d
local     22d91133b0a8b61fafd6c04ad68bb731501b3d7f659ac67a3e983860fdee7ea2
```

### USER

```docker
USER <username>[:<groupname>]
```

> `USER` 指定接下来 Dockerfile 中所有指令（如 `RUN`、`CMD`、`ENTRYPOINT`）以及容器运行时的默认用户。

默认情况下，容器中的进程会以 **`root` 用户**运行，而 `USER` 允许你切换为普通用户，从而提升安全性。

```dockerfile
FROM node:20

# 创建一个非 root 用户
RUN useradd -m appuser

# 切换为非 root 用户
USER appuser

# 之后的 RUN、CMD、ENTRYPOINT 都以 appuser 身份运行
CMD ["node", "app.js"]
```

### ONBUILD

`ONBUILD` 命令主要在父镜像中定义命令，调用该父镜像生成子镜像的时候才会被执行。

### STOPSIGNAL

`STOPSIGNAL` 告诉容器在接收到什么信号的时候停止运行

```
STOPSIGNAL signal
```

### SHELL

`SHELL` 命令指定用于解析命令的 shell 程序，如：

```dockerfile
SHELL ["/bin/bash", "-c"]
```

`SHELL` 指令会影响**后续所有**的 `RUN`、`CMD` 和 `ENTRYPOINT` 命令，直到遇到下一个 `SHELL` 指令。