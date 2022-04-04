# Docker命令学习笔记

构建并运行漏洞环境

```
docker-compose up -d
```

## 容器

### 列出所有的容器

```javascript
docker ps -a
```

### 列出所有的容器 ID

```javascript
docker ps -aq
```

### 杀死所有的容器

```javascript
docker kill $(docker ps -aq)
```

### 停止所有的容器

```javascript
docker stop $(docker ps -aq)
```

### 删除所有的容器

```javascript
docker rm $(docker ps -aq)
```

### 停止所有异常的容器

```javascript
docker ps -a | grep "Exited" | awk '{print $1}' | xargs docker stop
```

### 删除所有异常停止的容器

```javascript
docker ps -a | grep "Exited" | awk '{print $1}' | xargs docker rm
```



## 镜像

### 通过标签删除镜像

通过如下两个都可以删除镜像：

```javascript
docker rmi [image]
```

或者：

```javascript
docker image rm [image]
```

支持的子命令如下：

- `-f, -force`: 强制删除镜像，即便有容器引用该镜像；
- `-no-prune`: 不要删除未带标签的父镜像；

### 通过 ID 删除镜像

除了通过标签名称来删除镜像，我们还可以通过指定镜像 ID, 来删除镜像，如：

```javascript
docker rmi b7b28af77ffe
```

### 批量删除无 tag 标签镜像

在 Docker 构建镜像过程产生的临时镜像，或者遗留下来的垃圾镜像，可以通过以下命令删除。

```javascript
docker ps -a|grep "Exited" | awk '{print $1}' | xargs docker stop
docker ps -a|grep "Exited" | awk '{print $1}' | xargs docker rm
docker images|grep none|awk '{print $3}'|xargs docker rmi
```

### 删除所有镜像

```javascript
docker rmi $(docker images -q)
```

### 清理镜像

我们在使用 Docker 一段时间后，系统一般都会残存一些临时的、没有被使用的镜像文件，可以通过以下命令进行清理：

```javascript
docker image prune
```

它支持的子命令有：

- `-a, --all`: 删除所有没有用的镜像，而不仅仅是临时文件；
- `-f, --force`：强制删除镜像文件，无需弹出提示确认；