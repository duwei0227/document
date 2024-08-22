---
layout: post
title: Docker使用错误汇总
categories: [Docker]
permalink: docker/errors.html
---


### 一、Permission Denied
`Ubuntu`系统使用`snap`或`apt`安装完成后，在当前用户通过`docker`命令检查镜像等操作时，会提示 `permission denied`，此时可以通过增加 `sudo` 提升权限进行操作，如果不需要每次通过`sudo`操作，可以参考以下方式调整：

错误提示：
```
permission denied while trying to connect to the Docker daemon socket at unix:///var/run/docker.sock: Get "http://%2Fvar%2Frun%2Fdocker.sock/v1.24/images/json": dial unix /var/run/docker.sock: connect: permission denied
```

操作步骤(基于`ubuntu 24.04`)：
1、安装：

```
sudo apt install docker.io
```

2、添加当前用户到`docker`组

```
sudo gpasswd -a 用户名 docker
```

3、更新`docker`用户组

```
newgrp docker
```

4、完成，检查

```
docker version 或 docker images
```

