#### 1. nohup启动

> nohup sleep 2 > /dev/null 2>&1 &

#### 2. Linux系统查看端口占用的进程

```
yum -y install psmisc
fuser 9100/tcp 
```

#### 3. Linux查看命令的路径

> which -a bash
> command -v openresty

#### 4. grep 命令正则表达式

> grep -E "[0-9]+\\.[0-9]+" -o 

#### 5. 查看Linux进程打开的文件

> lsof -p pid

#### 6. linux按照find命令

##### 按照文件的扩展名进行查找

> find ~/develop/hadoop/share/hadoop -type f -name "*.xml"
