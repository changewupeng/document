# docker

## 安装

- 下载：https://download.docker.com/linux/static/stable/

- 安装

  ```
  tar -zxvf xxx.tar.gz
  sudo cp docker/* /usr/bin/
  ```

- 启动docker的守护进程

  ```
  sudo dockerd &
  ```

- 判断是否安装成功

  ```
  sudo docker run hello-world
  ```

  

```
## List Docker CLI commands
docker
docker container --help

## Display Docker version and info
docker --version
docker version
docker info

## Execute Docker image
docker run hello-world

## List Docker images
docker image ls

## List Docker containers (running, all, all in quiet mode)
docker container ls
docker container ls --all
docker container ls -aq
```

## demo

- Build the app

```
docker build --tag=friendlyhello .
```

- run app

  ```
  docker run -p 4000:80 friendlyhello
  ## 在后台运行
  docker run -d -p 4000:80 friendlyhello
  ```

- 现在在运行的container

  ```
  docker container ls
  ```

- 关闭指定的container

  ```
  docker container stop 1fa4ab2cf395
  ```

## 分享image

- 需要有一个docker hub的帐号，没有的去https://cloud.docker.com注册

- 在本机登录docker hub registries

  ```
  $ docker login -u -p
  ```

- 为镜像设置标签，格式是：username/repository:tag

  ```
  $ docker tag image username/repository:tag
  ```

- 展示本机的镜像

  ```
  docker image ls
  ```

- 发布镜像

  ```
  docker push username/repository:tag
  ```

  发布完毕后，在自己的docker hub上就可以看到这个镜像。

- 从远端仓库下载和运行镜像

  ```
  docker run -p 4000:80 username/repository:tag
  ```

  如果本机没有，会从docker hub去下载。

本章的命令：

```
docker build -t friendlyhello .  # Create image using this directory's Dockerfile
docker run -p 4000:80 friendlyhello  # Run "friendlyname" mapping port 4000 to 80
docker run -d -p 4000:80 friendlyhello         # Same thing, but in detached mode
docker container ls                                # List all running containers
docker container ls -a             # List all containers, even those not running
docker container stop <hash>           # Gracefully stop the specified container
docker container kill <hash>         # Force shutdown of the specified container
docker container rm <hash>        # Remove specified container from this machine
docker container rm $(docker container ls -a -q)         # Remove all containers
docker image ls -a                             # List all images on this machine
docker image rm <image id>            # Remove specified image from this machine
docker image rm $(docker image ls -a -q)   # Remove all images from this machine
docker login             # Log in this CLI session using your Docker credentials
docker tag <image> username/repository:tag  # Tag <image> for upload to registry
docker push username/repository:tag            # Upload tagged image to registry
docker run username/repository:tag                   # Run image from a registry
```

