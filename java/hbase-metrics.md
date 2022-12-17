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
































