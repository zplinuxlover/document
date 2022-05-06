#### 1. nohup启动

> nohup sleep 2 > /dev/null 2>&1 &

#### 2. Linux系统查看端口占用的进程

```
yum -y install psmisc
fuser 9100/tcp 
```