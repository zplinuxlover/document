### server/etcdserver/bootstrap.go recoverSnapshot

## bootstrap 流程解析

### 1. bootstrapSnapshot解析

通过readWALNames读取wal文件列表，openWALFiles打开所有的wal文件，依次扫描每一个wal文件，读取所有的snapshotType(记录snapshot类型的entry)，最新的stateType(记录的HardState类型的entry)类型的记录。
遍历扫描到的snapshot的entry记录，返回Index值不大于HardState中Commit的记录。
读取所有的snapshot文件，按照降序排序，依次检查每一个snapshot文件,找出最新的一个满足snapshot的index和term和wal中扫描出的snapshot记录相同的snapshot文件。通过snapshot文件，反序列化恢复v2store.Store对象。

### 2. bootstrapBackend 解析

初始化ConsistentIndexer和BackendHooks对象。打开boltdb数据库。在wal存在的情况下，执行recoverSnapshot恢复snapshot。

### 3. bootstrapWALFromSnapshot 解析

openWALFromSnapshot 找出所有wal文件中Index小于snapshot中Index的最新的wal文件，并且加载所有seq大于等于该wal的所有wal文件。解析所有的wal文件，解析出所有的type类型为entryType、stateType、metadataType的entry。




#### 问题列表

1. HardState中的Commit什么时候更新持久化的。

