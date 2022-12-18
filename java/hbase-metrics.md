## hbase metrics (2.4.13)

### hbase Put操作相关的监控

HBase在进行Put写入数据的时候，在源码中的org.apache.hadoop.hbase.regionserver.HRegion#batchMutate方法中会对Put操作进行统计，该统计不仅统计RegionServer的操作也统计，也涉及到对table的统计

在"Hadoop:service=HBase,name=RegionServer,sub=Server"这个mbean中，统计对RegionServer中所有Put的操作,通过com.codahale.metrics.Meter进行统计

```
{
  "ServerWriteQueryPerSecond_count": 13,
  "ServerWriteQueryPerSecond_mean_rate": 0.0023949377135419865,
  "ServerWriteQueryPerSecond_1min_rate": 0.019316872083132333,
  "ServerWriteQueryPerSecond_5min_rate": 0.014375177183927602,
  "ServerWriteQueryPerSecond_15min_rate": 0.009365322022879442
}
```

在"Hadoop:service=HBase,name=RegionServer,sub=TableLatencies"这个mbean中，统计该RegionServer上所有表的Put操作,通过com.codahale.metrics.Meter进行统计,其中每一条记录namespace下某一个表的写入速率

```
{
  "Namespace_hbase_table_namespace_metric_tableWriteQueryPerSecond_count": 2,
  "Namespace_hbase_table_namespace_metric_tableWriteQueryPerSecond_mean_rate": 3.687751526857381E-4,
  "Namespace_hbase_table_namespace_metric_tableWriteQueryPerSecond_1min_rate": 2.552601379224452E-40,
  "Namespace_hbase_table_namespace_metric_tableWriteQueryPerSecond_5min_rate": 5.794881947088224E-9,
  "Namespace_hbase_table_namespace_metric_tableWriteQueryPerSecond_15min_rate": 9.751128027398012E-4,
  "Namespace_hbase_table_meta_metric_tableWriteQueryPerSecond_count": 10,
  "Namespace_hbase_table_meta_metric_tableWriteQueryPerSecond_mean_rate": 0.0018436189727881736,
  "Namespace_hbase_table_meta_metric_tableWriteQueryPerSecond_1min_rate": 0.016414054742347445,
  "Namespace_hbase_table_meta_metric_tableWriteQueryPerSecond_5min_rate": 0.012042276424808542,
  "Namespace_hbase_table_meta_metric_tableWriteQueryPerSecond_15min_rate": 0.007422946401663329,
  "Namespace_default_table_debug_table_metric_tableWriteQueryPerSecond_count": 1,
  "Namespace_default_table_debug_table_metric_tableWriteQueryPerSecond_mean_rate": 0.011805880984427133,
  "Namespace_default_table_debug_table_metric_tableWriteQueryPerSecond_1min_rate": 0.05730095937203805,
  "Namespace_default_table_debug_table_metric_tableWriteQueryPerSecond_5min_rate": 0.15576015661428097,
  "Namespace_default_table_debug_table_metric_tableWriteQueryPerSecond_15min_rate": 0.18400888292586462
}
```

HBase在org.apache.hadoop.hbase.regionserver.RSRpcServices#put方法中通过如下代码对Put操作的时间进行统计

```
metricsRegionServer.updatePut(region.getRegionInfo().getTable(), after - before);
```

在该方法中，对每一个table的Put操作时间进行统计，统计记录在"name": "Hadoop:service=HBase,name=RegionServer,sub=TableLatencies"中

```
{
  "Namespace_default_table_debug_table_metric_putTime_num_ops": 1,
  "Namespace_default_table_debug_table_metric_putTime_min": 0,
  "Namespace_default_table_debug_table_metric_putTime_max": 0,
  "Namespace_default_table_debug_table_metric_putTime_mean": 0,
  "Namespace_default_table_debug_table_metric_putTime_25th_percentile": 0,
  "Namespace_default_table_debug_table_metric_putTime_median": 0,
  "Namespace_default_table_debug_table_metric_putTime_75th_percentile": 0,
  "Namespace_default_table_debug_table_metric_putTime_90th_percentile": 0,
  "Namespace_default_table_debug_table_metric_putTime_95th_percentile": 0,
  "Namespace_default_table_debug_table_metric_putTime_98th_percentile": 0,
  "Namespace_default_table_debug_table_metric_putTime_99th_percentile": 0,
  "Namespace_default_table_debug_table_metric_putTime_99.9th_percentile": 0,
}
```

如果Put操作的时间超过阈值（通过hbase.ipc.slow.metric.time配置，默认值为1000ms），则进行慢请求操作进行统计，统计结果记录在"Hadoop:service=HBase,name=RegionServer,sub=Server"中

```
{
  "slowPutCount": 0
}
```

统计该RegionServer上所有Put请求的分位值

```
{
  "Put_num_ops": 13,
  "Put_min": 0,
  "Put_max": 0,
  "Put_mean": 0,
  "Put_25th_percentile": 0,
  "Put_median": 0,
  "Put_75th_percentile": 0,
  "Put_90th_percentile": 0,
  "Put_95th_percentile": 0,
  "Put_98th_percentile": 0,
  "Put_99th_percentile": 0,
  "Put_99.9th_percentile": 0
}
```

如果开启hbase.regionserver.user.metrics.enabled参数（默认没有开启），则统计每一个client用户的PUT操作

### hbase append操作相关的监控

### hbase increment操作相关的监控

### hbase delete操作相关的监控

### hbase get操作的监控

hbase在org.apache.hadoop.hbase.regionserver.RSRpcServices#get方法中对get请求的统计操作，该统计不仅统计RegionServer的操作也统计，也涉及到对table的统计

在"Hadoop:service=HBase,name=RegionServer,sub=Server"这个mbean中，统计对RegionServer中所有Put的操作,通过com.codahale.metrics.Meter进行统计

```
{
      "ServerReadQueryPerSecond_count": 332,
      "ServerReadQueryPerSecond_mean_rate": 0.013143889653968838,
      "ServerReadQueryPerSecond_1min_rate": 0.04552697901167321,
      "ServerReadQueryPerSecond_5min_rate": 0.023731426674196317,
      "ServerReadQueryPerSecond_15min_rate": 0.017636400256602167  
}
```

在"Hadoop:service=HBase,name=RegionServer,sub=TableLatencies" mbean中统计get请求的分位值

```
{
      "Namespace_default_table_debug_table_metric_getTime_num_ops": 2,
      "Namespace_default_table_debug_table_metric_getTime_min": 1,
      "Namespace_default_table_debug_table_metric_getTime_max": 1,
      "Namespace_default_table_debug_table_metric_getTime_mean": 1,
      "Namespace_default_table_debug_table_metric_getTime_25th_percentile": 1,
      "Namespace_default_table_debug_table_metric_getTime_median": 1,
      "Namespace_default_table_debug_table_metric_getTime_75th_percentile": 1,
      "Namespace_default_table_debug_table_metric_getTime_90th_percentile": 1,
      "Namespace_default_table_debug_table_metric_getTime_95th_percentile": 1,
      "Namespace_default_table_debug_table_metric_getTime_98th_percentile": 1,
      "Namespace_default_table_debug_table_metric_getTime_99th_percentile": 1,
      "Namespace_default_table_debug_table_metric_getTime_99.9th_percentile": 1,
      "Namespace_default_table_debug_table_metric_getTime_TimeRangeCount_0-1": 1
}
```

如果get操作的时间超过阈值（通过hbase.ipc.slow.metric.time配置，默认值为1000ms），则进行慢请求操作进行统计，统计结果记录在"Hadoop:service=HBase,name=RegionServer,sub=Server"中

```
{
  "slowPutCount": 0
}
```

在"Hadoop:service=HBase,name=RegionServer,sub=Server"中对该RegionServer上的所有的get请求时间进行统计

```
{
      "Get_num_ops": 8,
      "Get_min": 1,
      "Get_max": 1,
      "Get_mean": 1,
      "Get_25th_percentile": 1,
      "Get_median": 1,
      "Get_75th_percentile": 1,
      "Get_90th_percentile": 1,
      "Get_95th_percentile": 1,
      "Get_98th_percentile": 1,
      "Get_99th_percentile": 1,
      "Get_99.9th_percentile": 1,
      "Get_TimeRangeCount_0-1": 1
}
```

如果开启hbase.regionserver.user.metrics.enabled参数（默认没有开启），则统计每一个client用户的GET操作

### hbase wal写操作监控

### hbase flush操作的监控

hbase在org.apache.hadoop.hbase.regionserver.MemStoreFlusher.FlushHandler#run执行flush操作

在"Hadoop:service=HBase,name=RegionServer,sub=Server"这个mbean中统计flush操作耗时分位值

```
{
      "FlushTime_num_ops": 4,
      "FlushTime_min": 0,
      "FlushTime_max": 0,
      "FlushTime_mean": 0,
      "FlushTime_25th_percentile": 0,
      "FlushTime_median": 0,
      "FlushTime_75th_percentile": 0,
      "FlushTime_90th_percentile": 0,
      "FlushTime_95th_percentile": 0,
      "FlushTime_98th_percentile": 0,
      "FlushTime_99th_percentile": 0,
      "FlushTime_99.9th_percentile": 0
}
```

统计flush操作的字节数,以及自启动总的flush字节数, flush操作生成的file size,以及自启动总的flush生成file的字节数

```
{
      "FlushMemstoreSize_num_ops": 4,
      "FlushMemstoreSize_min": 0,
      "FlushMemstoreSize_max": 0,
      "FlushMemstoreSize_mean": 0,
      "FlushMemstoreSize_25th_percentile": 0,
      "FlushMemstoreSize_median": 0,
      "FlushMemstoreSize_75th_percentile": 0,
      "FlushMemstoreSize_90th_percentile": 0,
      "FlushMemstoreSize_95th_percentile": 0,
      "FlushMemstoreSize_98th_percentile": 0,
      "FlushMemstoreSize_99th_percentile": 0,
      "FlushMemstoreSize_99.9th_percentile": 0,
      "flushedMemstoreBytes": 2590,
      "FlushOutputSize_num_ops": 4,
      "FlushOutputSize_min": 0,
      "FlushOutputSize_max": 0,
      "FlushOutputSize_mean": 0,
      "FlushOutputSize_25th_percentile": 0,
      "FlushOutputSize_median": 0,
      "FlushOutputSize_75th_percentile": 0,
      "FlushOutputSize_90th_percentile": 0,
      "FlushOutputSize_95th_percentile": 0,
      "FlushOutputSize_98th_percentile": 0,
      "FlushOutputSize_99th_percentile": 0,
      "FlushOutputSize_99.9th_percentile": 0
}
```

在"Hadoop:service=HBase,name=RegionServer,sub=Tables"这个mbean中统计每一个table的flush操作耗时分位值

```
{
      "Namespace_default_table_debug_table_metric_flushTime_num_ops": 1,
      "Namespace_default_table_debug_table_metric_flushTime_min": 0,
      "Namespace_default_table_debug_table_metric_flushTime_max": 0,
      "Namespace_default_table_debug_table_metric_flushTime_mean": 0,
      "Namespace_default_table_debug_table_metric_flushTime_25th_percentile": 0,
      "Namespace_default_table_debug_table_metric_flushTime_median": 0,
      "Namespace_default_table_debug_table_metric_flushTime_75th_percentile": 0,
      "Namespace_default_table_debug_table_metric_flushTime_90th_percentile": 0,
      "Namespace_default_table_debug_table_metric_flushTime_95th_percentile": 0,
      "Namespace_default_table_debug_table_metric_flushTime_98th_percentile": 0,
      "Namespace_default_table_debug_table_metric_flushTime_99th_percentile": 0,
      "Namespace_default_table_debug_table_metric_flushTime_99.9th_percentile": 0
}
```

统计table的flush操作的字节数,以及自启动总的flush字节数, flush操作生成的file size,以及自启动总的flush生成file的字节数

```
{
      "Namespace_default_table_debug_table_metric_flushMemstoreSize_num_ops": 1,
      "Namespace_default_table_debug_table_metric_flushMemstoreSize_min": 0,
      "Namespace_default_table_debug_table_metric_flushMemstoreSize_max": 0,
      "Namespace_default_table_debug_table_metric_flushMemstoreSize_mean": 0,
      "Namespace_default_table_debug_table_metric_flushMemstoreSize_25th_percentile": 0,
      "Namespace_default_table_debug_table_metric_flushMemstoreSize_median": 0,
      "Namespace_default_table_debug_table_metric_flushMemstoreSize_75th_percentile": 0,
      "Namespace_default_table_debug_table_metric_flushMemstoreSize_90th_percentile": 0,
      "Namespace_default_table_debug_table_metric_flushMemstoreSize_95th_percentile": 0,
      "Namespace_default_table_debug_table_metric_flushMemstoreSize_98th_percentile": 0,
      "Namespace_default_table_debug_table_metric_flushMemstoreSize_99th_percentile": 0,
      "Namespace_default_table_debug_table_metric_flushMemstoreSize_99.9th_percentile": 0,
      "Namespace_default_table_debug_table_metric_flushedMemstoreBytes": 32,
      "Namespace_default_table_debug_table_metric_flushOutputSize_num_ops": 1,
      "Namespace_default_table_debug_table_metric_flushOutputSize_min": 0,
      "Namespace_default_table_debug_table_metric_flushOutputSize_max": 0,
      "Namespace_default_table_debug_table_metric_flushOutputSize_mean": 0,
      "Namespace_default_table_debug_table_metric_flushOutputSize_25th_percentile": 0,
      "Namespace_default_table_debug_table_metric_flushOutputSize_median": 0,
      "Namespace_default_table_debug_table_metric_flushOutputSize_75th_percentile": 0,
      "Namespace_default_table_debug_table_metric_flushOutputSize_90th_percentile": 0,
      "Namespace_default_table_debug_table_metric_flushOutputSize_95th_percentile": 0,
      "Namespace_default_table_debug_table_metric_flushOutputSize_98th_percentile": 0,
      "Namespace_default_table_debug_table_metric_flushOutputSize_99th_percentile": 0,
      "Namespace_default_table_debug_table_metric_flushOutputSize_99.9th_percentile": 0,
      "Namespace_default_table_debug_table_metric_flushedOutputBytes": 4861
      
}
```

### hbase compaction操作的监控

hbase在org.apache.hadoop.hbase.regionserver.HStore#doCompaction方法中进行compact操作，compact分为minor compact和major compact。

在"Hadoop:service=HBase,name=RegionServer,sub=Server"这个mbean中统计RegionServer进行的compact操作的执行时间, compact文件大小的分位值统计, compact过程写文件的分位统计

```
{
      "CompactionTime_num_ops": 0,
      "CompactionTime_min": 0,
      "CompactionTime_max": 0,
      "CompactionTime_mean": 0,
      "CompactionTime_25th_percentile": 0,
      "CompactionTime_median": 0,
      "CompactionTime_75th_percentile": 0,
      "CompactionTime_90th_percentile": 0,
      "CompactionTime_95th_percentile": 0,
      "CompactionTime_98th_percentile": 0,
      "CompactionTime_99th_percentile": 0,
      "CompactionTime_99.9th_percentile": 0,
      
      "CompactionInputFileCount_num_ops": 0,
      "CompactionInputFileCount_min": 0,
      "CompactionInputFileCount_max": 0,
      "CompactionInputFileCount_mean": 0,
      "CompactionInputFileCount_25th_percentile": 0,
      "CompactionInputFileCount_median": 0,
      "CompactionInputFileCount_75th_percentile": 0,
      "CompactionInputFileCount_90th_percentile": 0,
      "CompactionInputFileCount_95th_percentile": 0,
      "CompactionInputFileCount_98th_percentile": 0,
      "CompactionInputFileCount_99th_percentile": 0,
      "CompactionInputFileCount_99.9th_percentile": 0,
      
      "CompactionOutputFileCount_num_ops": 0,
      "CompactionOutputFileCount_min": 0,
      "CompactionOutputFileCount_max": 0,
      "CompactionOutputFileCount_mean": 0,
      "CompactionOutputFileCount_25th_percentile": 0,
      "CompactionOutputFileCount_median": 0,
      "CompactionOutputFileCount_75th_percentile": 0,
      "CompactionOutputFileCount_90th_percentile": 0,
      "CompactionOutputFileCount_95th_percentile": 0,
      "CompactionOutputFileCount_98th_percentile": 0,
      "CompactionOutputFileCount_99th_percentile": 0,
      "CompactionOutputFileCount_99.9th_percentile": 0,
      
      "CompactionInputSize_num_ops": 0,
      "CompactionInputSize_min": 0,
      "CompactionInputSize_max": 0,
      "CompactionInputSize_mean": 0,
      "CompactionInputSize_25th_percentile": 0,
      "CompactionInputSize_median": 0,
      "CompactionInputSize_75th_percentile": 0,
      "CompactionInputSize_90th_percentile": 0,
      "CompactionInputSize_95th_percentile": 0,
      "CompactionInputSize_98th_percentile": 0,
      "CompactionInputSize_99th_percentile": 0,
      "CompactionInputSize_99.9th_percentile": 0,
      
      "CompactionOutputSize_num_ops": 0,
      "CompactionOutputSize_min": 0,
      "CompactionOutputSize_max": 0,
      "CompactionOutputSize_mean": 0,
      "CompactionOutputSize_25th_percentile": 0,
      "CompactionOutputSize_median": 0,
      "CompactionOutputSize_75th_percentile": 0,
      "CompactionOutputSize_90th_percentile": 0,
      "CompactionOutputSize_95th_percentile": 0,
      "CompactionOutputSize_98th_percentile": 0,
      "CompactionOutputSize_99th_percentile": 0,
      "CompactionOutputSize_99.9th_percentile": 0,
      
      "MajorCompactionTime_num_ops": 0,
      "MajorCompactionTime_min": 0,
      "MajorCompactionTime_max": 0,
      "MajorCompactionTime_mean": 0,
      "MajorCompactionTime_25th_percentile": 0,
      "MajorCompactionTime_median": 0,
      "MajorCompactionTime_75th_percentile": 0,
      "MajorCompactionTime_90th_percentile": 0,
      "MajorCompactionTime_95th_percentile": 0,
      "MajorCompactionTime_98th_percentile": 0,
      "MajorCompactionTime_99th_percentile": 0,
      "MajorCompactionTime_99.9th_percentile": 0,
      
      "MajorCompactionInputFileCount_num_ops": 0,
      "MajorCompactionInputFileCount_min": 0,
      "MajorCompactionInputFileCount_max": 0,
      "MajorCompactionInputFileCount_mean": 0,
      "MajorCompactionInputFileCount_25th_percentile": 0,
      "MajorCompactionInputFileCount_median": 0,
      "MajorCompactionInputFileCount_75th_percentile": 0,
      "MajorCompactionInputFileCount_90th_percentile": 0,
      "MajorCompactionInputFileCount_95th_percentile": 0,
      "MajorCompactionInputFileCount_98th_percentile": 0,
      "MajorCompactionInputFileCount_99th_percentile": 0,
      "MajorCompactionInputFileCount_99.9th_percentile": 0,
      
      "MajorCompactionOutputSize_num_ops": 0,
      "MajorCompactionOutputSize_min": 0,
      "MajorCompactionOutputSize_max": 0,
      "MajorCompactionOutputSize_mean": 0,
      "MajorCompactionOutputSize_25th_percentile": 0,
      "MajorCompactionOutputSize_median": 0,
      "MajorCompactionOutputSize_75th_percentile": 0,
      "MajorCompactionOutputSize_90th_percentile": 0,
      "MajorCompactionOutputSize_95th_percentile": 0,
      "MajorCompactionOutputSize_98th_percentile": 0,
      "MajorCompactionOutputSize_99th_percentile": 0,
      "MajorCompactionOutputSize_99.9th_percentile": 0,  
      
      "MajorCompactionInputSize_num_ops": 0,
      "MajorCompactionInputSize_min": 0,
      "MajorCompactionInputSize_max": 0,
      "MajorCompactionInputSize_mean": 0,
      "MajorCompactionInputSize_25th_percentile": 0,
      "MajorCompactionInputSize_median": 0,
      "MajorCompactionInputSize_75th_percentile": 0,
      "MajorCompactionInputSize_90th_percentile": 0,
      "MajorCompactionInputSize_95th_percentile": 0,
      "MajorCompactionInputSize_98th_percentile": 0,
      "MajorCompactionInputSize_99th_percentile": 0,
      "MajorCompactionInputSize_99.9th_percentile": 0,
      
      "MajorCompactionOutputFileCount_num_ops": 0,
      "MajorCompactionOutputFileCount_min": 0,
      "MajorCompactionOutputFileCount_max": 0,
      "MajorCompactionOutputFileCount_mean": 0,
      "MajorCompactionOutputFileCount_25th_percentile": 0,
      "MajorCompactionOutputFileCount_median": 0,
      "MajorCompactionOutputFileCount_75th_percentile": 0,
      "MajorCompactionOutputFileCount_90th_percentile": 0,
      "MajorCompactionOutputFileCount_95th_percentile": 0,
      "MajorCompactionOutputFileCount_98th_percentile": 0,
      "MajorCompactionOutputFileCount_99th_percentile": 0,
      "MajorCompactionOutputFileCount_99.9th_percentile": 0,      
}      
```

在 "Hadoop:service=HBase,name=RegionServer,sub=Tables" mbean中统计每一个table的compact操作

```
{
      "Namespace_default_table_debug_table_metric_compactionTime_num_ops": 0,
      "Namespace_default_table_debug_table_metric_compactionTime_min": 0,
      "Namespace_default_table_debug_table_metric_compactionTime_max": 0,
      "Namespace_default_table_debug_table_metric_compactionTime_mean": 0,
      "Namespace_default_table_debug_table_metric_compactionTime_25th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionTime_median": 0,
      "Namespace_default_table_debug_table_metric_compactionTime_75th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionTime_90th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionTime_95th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionTime_98th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionTime_99th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionTime_99.9th_percentile": 0,
      
      "Namespace_default_table_debug_table_metric_compactionInputFileCount_num_ops": 0,
      "Namespace_default_table_debug_table_metric_compactionInputFileCount_min": 0,
      "Namespace_default_table_debug_table_metric_compactionInputFileCount_max": 0,
      "Namespace_default_table_debug_table_metric_compactionInputFileCount_mean": 0,
      "Namespace_default_table_debug_table_metric_compactionInputFileCount_25th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionInputFileCount_median": 0,
      "Namespace_default_table_debug_table_metric_compactionInputFileCount_75th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionInputFileCount_90th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionInputFileCount_95th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionInputFileCount_98th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionInputFileCount_99th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionInputFileCount_99.9th_percentile": 0,
      
      "Namespace_default_table_debug_table_metric_compactionInputSize_num_ops": 0,
      "Namespace_default_table_debug_table_metric_compactionInputSize_min": 0,
      "Namespace_default_table_debug_table_metric_compactionInputSize_max": 0,
      "Namespace_default_table_debug_table_metric_compactionInputSize_mean": 0,
      "Namespace_default_table_debug_table_metric_compactionInputSize_25th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionInputSize_median": 0,
      "Namespace_default_table_debug_table_metric_compactionInputSize_75th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionInputSize_90th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionInputSize_95th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionInputSize_98th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionInputSize_99th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionInputSize_99.9th_percentile": 0,
      
      "Namespace_default_table_debug_table_metric_compactionOutputFileCount_num_ops": 0,
      "Namespace_default_table_debug_table_metric_compactionOutputFileCount_min": 0,
      "Namespace_default_table_debug_table_metric_compactionOutputFileCount_max": 0,
      "Namespace_default_table_debug_table_metric_compactionOutputFileCount_mean": 0,
      "Namespace_default_table_debug_table_metric_compactionOutputFileCount_25th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionOutputFileCount_median": 0,
      "Namespace_default_table_debug_table_metric_compactionOutputFileCount_75th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionOutputFileCount_90th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionOutputFileCount_95th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionOutputFileCount_98th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionOutputFileCount_99th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionOutputFileCount_99.9th_percentile": 0,
      
      "Namespace_default_table_debug_table_metric_compactionOutputSize_num_ops": 0,
      "Namespace_default_table_debug_table_metric_compactionOutputSize_min": 0,
      "Namespace_default_table_debug_table_metric_compactionOutputSize_max": 0,
      "Namespace_default_table_debug_table_metric_compactionOutputSize_mean": 0,
      "Namespace_default_table_debug_table_metric_compactionOutputSize_25th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionOutputSize_median": 0,
      "Namespace_default_table_debug_table_metric_compactionOutputSize_75th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionOutputSize_90th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionOutputSize_95th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionOutputSize_98th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionOutputSize_99th_percentile": 0,
      "Namespace_default_table_debug_table_metric_compactionOutputSize_99.9th_percentile": 0,   
      
      "Namespace_hbase_table_namespace_metric_majorCompactionTime_num_ops": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionTime_min": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionTime_max": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionTime_mean": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionTime_25th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionTime_median": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionTime_75th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionTime_90th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionTime_95th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionTime_98th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionTime_99th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionTime_99.9th_percentile": 0,
      
      "Namespace_hbase_table_namespace_metric_majorCompactionInputFileCount_num_ops": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionInputFileCount_min": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionInputFileCount_max": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionInputFileCount_mean": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionInputFileCount_25th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionInputFileCount_median": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionInputFileCount_75th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionInputFileCount_90th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionInputFileCount_95th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionInputFileCount_98th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionInputFileCount_99th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionInputFileCount_99.9th_percentile": 0,
      
      "Namespace_hbase_table_namespace_metric_majorCompactionInputSize_num_ops": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionInputSize_min": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionInputSize_max": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionInputSize_mean": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionInputSize_25th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionInputSize_median": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionInputSize_75th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionInputSize_90th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionInputSize_95th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionInputSize_98th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionInputSize_99th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionInputSize_99.9th_percentile": 0,
      
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputFileCount_num_ops": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputFileCount_min": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputFileCount_max": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputFileCount_mean": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputFileCount_25th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputFileCount_median": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputFileCount_75th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputFileCount_90th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputFileCount_95th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputFileCount_98th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputFileCount_99th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputFileCount_99.9th_percentile": 0,
      
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputSize_num_ops": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputSize_min": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputSize_max": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputSize_mean": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputSize_25th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputSize_median": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputSize_75th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputSize_90th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputSize_95th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputSize_98th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputSize_99th_percentile": 0,
      "Namespace_hbase_table_namespace_metric_majorCompactionOutputSize_99.9th_percentile": 0,      
}

```

### hbase的split操作的统计







