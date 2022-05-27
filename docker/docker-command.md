#### docker命令

##### 1. docker拉取镜像命令

> docker pull centos:7.9.2009

##### 2. docker运行指定的镜像

> docker run -ti -v $HOME/workspace:/data centos:7.9.2009 bash

##### 3. go交叉编译

> CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build

##### 4. docker build

> docker build -t provider-a:1.0 .