### Prometheus TSDB源码解析，Index索引存储格式分析

当prometheus判断Head中存储的Series的时间范围超过3*chunkRange/2（storage.tsdb.min-block-duration参数）时，Prometheus将会执行Compact流程，将Head中的chunkRange范围的Series持久化到Block中。代码参考（tsdb/db.go Compact 方法）

```
func (h *Head) compactable() bool {
	return h.MaxTime()-h.MinTime() > h.chunkRange.Load()/2*3
}
```

Prometheus 整体的文件结构如下

```
├── 01G2K64976PG4ZN58D4WA3GZYB
├── 01G2K649VGZMA5T4BFTKASSRE6
├── chunks_head
├── lock
├── queries.active
└── wal
```

其中的01G2K64976PG4ZN58D4WA3GZYB和01G2K649VGZMA5T4BFTKASSRE6(ULID 可以参考 [ULID](https://github.com/oklog/ulid))是一个完整的Block，里面存储一定时间范围的Series的Samples。

其中每一个block的结构如下, 01G2K64976PG4ZN58D4WA3GZYB目录下的文件结构。

```
├── chunks
│   └── 000001
├── index
├── meta.json
└── tombstones
```

其中chunks目录下存储每一个Block中的所有的Chunk, Index文件是该Chunk的倒排索引，meta.json记录了该Block的元数据，tombstones记录的是删除信息。

本文中我们重点分析Index文件的存储格式，了解该格式是明白Prometheus查询流程的前提

#### Prometheus的Index存储格式

```
┌────────────────────────────┬─────────────────────┐
│ magic(0xBAAAD700) <4b>     │ version(1) <1 byte> │
├────────────────────────────┴─────────────────────┤
│ ┌──────────────────────────────────────────────┐ │
│ │                 Symbol Table                 │ │
│ ├──────────────────────────────────────────────┤ │
│ │                    Series                    │ │
│ ├──────────────────────────────────────────────┤ │
│ │                 Label Indices                │ │
│ ├──────────────────────────────────────────────┤ │
│ │                   Postings                   │ │
│ ├──────────────────────────────────────────────┤ │
│ │               Label Offset Table             │ │
│ ├──────────────────────────────────────────────┤ │
│ │             Postings Offset Table            │ │
│ ├──────────────────────────────────────────────┤ │
│ │                      TOC                     │ │
│ └──────────────────────────────────────────────┘ │
└──────────────────────────────────────────────────┘
```
TOC(table of content)记录的是该文件中Symbol Table，Series，Label Indices，Postings，Label Offset Table，Postings Offset Table在文件中的位置，该部分为固定长度52B，解析该文件的时候，通过解析文件末尾的52Bytes即可获取Index中各个部分在文件中的位置，该部分的详细结构为

```
┌──────────────────────────────────┐
│ ref<symbols> <8B>                |
├──────────────────────────────────┤
│ ref<Series>  <8B>                │ 
├──────────────────────────────────┤
│ ref<Label Indices>  <8B>         │ 
├──────────────────────────────────┤
│ ref<Label Offset Table> <8B>     │ 
├──────────────────────────────────┤
│ ref<Postings>  <8B>              │ 
├──────────────────────────────────┤
│ ref<Postings Offset Table> <8B>  │ 
├──────────────────────────────────┤
│ ref<Postings Offset Table> <8B>  |
└──────────────────────────────────┘

```

首先我们分析Symbol Table部分的详细结构，该部分记录的是Series中的所有的label-value中的值。
假如我们在Head中存在如下Series

```
metrics_1{label_1="value_1", label_2="value_2"}
metrics_2{label_1="value_1", label_3="value_3"}
```

该部分存储的是__name__(内部的特殊label name)，metrics_1，label_1，value_1，label_2，value_2，metrics_2，label_3，value_3，需要对这些字符串进行升序排序。


```
┌────────────────────┬─────────────────────┐
│ len <4B>           │ #symbols <4B>       │
├────────────────────┴─────────────────────┤
│ ┌──────────────────────┬───────────────┐ │
│ │ len(str_1) <uvarint> │ str_1 <bytes> │ │
│ ├──────────────────────┴───────────────┤ │
│ │                . . .                 │ │
│ ├──────────────────────┬───────────────┤ │
│ │ len(str_n) <uvarint> │ str_n <bytes> │ │
│ └──────────────────────┴───────────────┘ │
├──────────────────────────────────────────┤
│ CRC32 <4B>                               │
└──────────────────────────────────────────┘
```

其中len记录的是Series部分的长度(不包括len<4B>部分), #symbols 记录的是symbol的个数，后面是#symbols个symbol，其中每一个首先记录的是该symbol的长度，后面跟随symbol的实际的字节数组。最后的部分是对#symbols和所有的symbols的CRC32的校验值(从#symbols开始到CRC32之前的部分，不包括len部分)。



















































































