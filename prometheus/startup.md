#### Prometheus启动流程

Prometheus启动过程中首先通过[Kingpin](https://github.com/alecthomas/kingpin)来解析命令行参数
```
a := kingpin.New(filepath.Base(os.Args[0]), "The Prometheus monitoring server").UsageWriter(os.Stdout)
```

命令行主要参数包括

| 命令行参数  | 命令说明  | 默认值  |
|:----------|:----------|:----------|
| config.file | Prometheus配置文件路径 | prometheus.yml |
| web.listen-address | Prometheus监听的地址 | 0.0.0.0:9090 |
| web.read-timeout | Maximum duration before timing out read of the request, and closing idle connections. | 5m |
| web.max-connections | 最大的并发链接数 | 512 |
| web.external-url | Prometheus配置外部可访问地址，比如Prometheus通过reverse proxy提供服务的情况下 | 无 |
| web.route-prefix | Prefix for the internal routes of web endpoints. Defaults to path of --web.external-url. | 无 |
| web.user-assets | Path to static asset directory, available at /user. | 无 |
| web.enable-lifecycle | 是否开启通过HTTP端口的方式停止Prometheus服务和重新加载配置文件 | false |
| web.enable-admin-api | 是否开启通过HTTP的方式调用Prometheus的管理接口，比如/admin/tsdb/delete_series | false |
| web.enable-remote-write-receiver | 是否开启通过HTTP的方式调用Prometheus的接口写入Metrics | false |
| web.console.templates | Path to the console template directory, available at /consoles. | consoles |
| web.console.libraries | Path to the console library directory. | console_libraries |
| web.page-title | Prometheus文档页面的title | Prometheus Time Series Collection and Processing Server |
| web.cors.origin | Prometheus跨域访问配置 | .* |
| storage.tsdb.path | Prometheus metrics storage的路径 | data/ |
| storage.tsdb.min-block-duration | Minimum duration of a data block before being persisted. For use in testing. | 2h |
| storage.tsdb.max-block-duration | Maximum duration compacted blocks may span. For use in testing. (Defaults to 10% of the retention period.) | 无 |
| storage.tsdb.max-block-chunk-segment-size | The maximum size for a single chunk segment in a block. Example: 512MB | 无 |
| storage.tsdb.wal-segment-size | Size at which to split the tsdb WAL segment files. Example: 100MB | 无 |
| storage.tsdb.retention | Prometheus Metrics保留时间（该选项已经废弃，需要使用storage.tsdb.retention.time） | 无 |
| storage.tsdb.retention.time | Prometheus Metrics保留最大时间 | 无 |
| storage.tsdb.retention.size | Prometheus Metrics保留最大存储大小 | 无 |
| storage.tsdb.no-lockfile | Prometheus是否不在数据存储目录生成lock file | false |
| storage.tsdb.allow-overlapping-blocks | Allow overlapping blocks, which in turn enables vertical compaction and vertical query merge. | false |
| storage.tsdb.wal-compression | Compress the tsdb WAL. | true |
| storage.tsdb.head-chunks-write-queue-size | Size of the queue through which head chunks are written to the disk to be m-mapped, 0 disables the queue completely. Experimental. | 0 |
| storage.agent.path | Base path for metrics storage. | data-agent/ |
| storage.agent.wal-segment-size | Size at which to split WAL segment files. Example: 100MB | 无 |
| storage.agent.wal-compression | Compress the agent WAL. | true |
| storage.agent.wal-truncate-frequency | The frequency at which to truncate the WAL and remove old data. | 无 |
| storage.agent.retention.min-time | Minimum age samples may be before being considered for deletion when the WAL is truncated | 无 |
| storage.agent.retention.max-time | Maximum age samples may be before being forcibly deleted when the WAL is truncated | 无 |
| storage.agent.no-lockfile | Do not create lockfile in data directory | false |
| storage.remote.flush-deadline | How long to wait flushing sample on shutdown or config reload. | 1m |
| storage.remote.read-sample-limit | Maximum overall number of samples to return via the remote read interface, in a single query. 0 means no limit. This limit is ignored for streamed response types. | 5e7 |
| storage.remote.read-concurrent-limit | Maximum number of concurrent remote read calls. 0 means no limit. | 10 |
| storage.remote.read-max-bytes-in-frame | Maximum number of bytes in a single frame for streaming remote read response types before marshalling. Note that client might have limit on frame size as well. 1MB as recommended by protobuf by default. | 1048576 |
| rules.alert.for-outage-tolerance | Max time to tolerate prometheus outage for restoring "for" state of alert. |1h |
| rules.alert.for-grace-period | Minimum duration between alert and restored "for" state. This is maintained only for alerts with configured "for" time greater than grace period. | 10m |
| rules.alert.resend-delay | Minimum amount of time to wait before resending an alert to Alertmanager. | 1m |
| scrape.adjust-timestamps | Adjust scrape timestamps by up to `scrape.timestamp-tolerance` to align them to the intended schedule. See https://github.com/prometheus/prometheus/issues/7846 for more context. Experimental. This flag will be removed in a future release. | true |
| scrape.timestamp-tolerance | Timestamp tolerance. See https://github.com/prometheus/prometheus/issues/7846 for more context. Experimental. This flag will be removed in a future release. | 2ms |
| alertmanager.notification-queue-capacity | The capacity of the queue for pending Alertmanager notifications. | 10000 |
| query.lookback-delta | The maximum lookback duration for retrieving metrics during expression evaluations and federation. | 5m |
| query.timeout | Maximum time a query may take before being aborted. | 2m |
| query.max-concurrency | Maximum number of queries executed concurrently. | 20 |
| query.max-samples | Maximum number of samples a single query can load into memory. Note that queries will fail if they try to load more samples than this into memory, so this also limits the number of samples a query can return. | 50000000 |
| enable-feature | Comma separated feature names to enable. Valid options: agent, exemplar-storage, expand-external-labels, memory-snapshot-on-shutdown, promql-at-modifier, promql-negative-offset, promql-per-step-stats, remote-write-receiver (DEPRECATED), extra-scrape-metrics, new-service-discovery-manager, auto-gomaxprocs. See https://prometheus.io/docs/prometheus/latest/feature_flags/ for more details. | "" |
| log.level | log日志级别 | info |
| log.format | log日志格式 | logfmt |
































