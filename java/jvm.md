#### 1. Java dump内存

> jmap -dump:format=b,file=/data/91317.dump 91317

#### 2. OQL Syntax

##### 2.1 查询字符串长度大于指定字符

```
select * 
FROM org.t3.hbase.controller.DebugController s
where s.name.value.@length > 4
```

