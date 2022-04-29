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

解析程序选项

```
_, err := a.Parse(os.Args[1:])
```

设置Prometheus开启的feature

```
if err := cfg.setFeatureListOptions(logger); err != nil {
    fmt.Fprintln(os.Stderr, errors.Wrapf(err, "Error parsing feature list"))
    os.Exit(1)
}
```

在Prometheus以agent和server模式启动的时候判断开启的feature是否合法

```
if agentMode && len(serverOnlyFlags) > 0 {
    fmt.Fprintf(os.Stderr, "The following flag(s) can not be used in agent mode: %q", serverOnlyFlags)
    os.Exit(3)
}

if !agentMode && len(agentOnlyFlags) > 0 {
    fmt.Fprintf(os.Stderr, "The following flag(s) can only be used in agent mode: %q", agentOnlyFlags)
    os.Exit(3)
}
```
获取Prometheus的数据存储目录，在Prometheus以server模式启动的时候，使用cfg.serverStoragePath，以agent模式启动的时候，使用cfg.agentStoragePath

```
localStoragePath := cfg.serverStoragePath
if agentMode {
    localStoragePath = cfg.agentStoragePath
}
```

计算Prometheus的ExternalURL

```
cfg.web.ExternalURL, err = computeExternalURL(cfg.prometheusURL, cfg.web.ListenAddress)
if err != nil {
    fmt.Fprintln(os.Stderr, errors.Wrapf(err, "parse external URL %q", cfg.prometheusURL))
    os.Exit(2)
}
```

计算CORS跨域配置

```
cfg.web.CORSOrigin, err = compileCORSRegexString(cfg.corsRegexString)
if err != nil {
    fmt.Fprintln(os.Stderr, errors.Wrapf(err, "could not compile CORS regex string %q", cfg.corsRegexString))
    os.Exit(2)
}
```
加载并且解析配置文件

```
var cfgFile *config.Config
if cfgFile, err = config.LoadFile(cfg.configFile, agentMode, false, log.NewNopLogger()); err != nil {
    absPath, pathErr := filepath.Abs(cfg.configFile)
    if pathErr != nil {
        absPath = cfg.configFile
    }
    level.Error(logger).Log("msg", fmt.Sprintf("Error loading config (--config.file=%s)", cfg.configFile), "file", absPath, "err", err)
    os.Exit(2)
}
```

~~TODO~~

```
if cfg.tsdb.EnableExemplarStorage {
    if cfgFile.StorageConfig.ExemplarsConfig == nil {
        cfgFile.StorageConfig.ExemplarsConfig = &config.DefaultExemplarsConfig
    }
    cfg.tsdb.MaxExemplars = int64(cfgFile.StorageConfig.ExemplarsConfig.MaxExemplars)
}
```
Prometheus在以server模式启动的时候，计算Prometheus TSDB数据保留最大的时间cfg.tsdb.RetentionDuration，如果cfg.tsdb.RetentionDuration和cfg.tsdb.MaxBytes在都没有设置的逻辑下，采取默认的配置（15d），当cfg.tsdb.RetentionDuration的配置为负数的情况下，最大保留100year的数据。

```

// Time retention settings.
if oldFlagRetentionDuration != 0 {
    level.Warn(logger).Log("deprecation_notice", "'storage.tsdb.retention' flag is deprecated use 'storage.tsdb.retention.time' instead.")
    cfg.tsdb.RetentionDuration = oldFlagRetentionDuration
}

// When the new flag is set it takes precedence.
if newFlagRetentionDuration != 0 {
    cfg.tsdb.RetentionDuration = newFlagRetentionDuration
}

if cfg.tsdb.RetentionDuration == 0 && cfg.tsdb.MaxBytes == 0 {
    cfg.tsdb.RetentionDuration = defaultRetentionDuration
    level.Info(logger).Log("msg", "No time or size retention was set so using the default time retention", "duration", defaultRetentionDuration)
}

// Check for overflows. This limits our max retention to 100y.
if cfg.tsdb.RetentionDuration < 0 {
    y, err := model.ParseDuration("100y")
    if err != nil {
        panic(err)
    }
    cfg.tsdb.RetentionDuration = y
    level.Warn(logger).Log("msg", "Time retention value is too high. Limiting to: "+y.String())
}

```
Prometheus在没有设置cfg.tsdb.MaxBlockDuration的情况下，通过默认值（31d）和cfg.tsdb.RetentionDuration计算最终的值
```
if cfg.tsdb.MaxBlockDuration == 0 {
    maxBlockDuration, err := model.ParseDuration("31d")
    if err != nil {
        panic(err)
    }
    // When the time retention is set and not too big use to define the max block duration.
    if cfg.tsdb.RetentionDuration != 0 && cfg.tsdb.RetentionDuration/10 < maxBlockDuration {
        maxBlockDuration = cfg.tsdb.RetentionDuration / 10
    }

    cfg.tsdb.MaxBlockDuration = maxBlockDuration
}
```

在开启new-service-discovery-manager的情况下，生成HTTP SD，否则生成File-Based SD，两者的区别参考文档[HTTP SD和File-Based SD的对比](https://prometheus.io/docs/prometheus/latest/http_sd/)

```
if cfg.enableNewSDManager {
    discovery.RegisterMetrics()
    discoveryManagerScrape = discovery.NewManager(ctxScrape, log.With(logger, "component", "discovery manager scrape"), discovery.Name("scrape"))
    discoveryManagerNotify = discovery.NewManager(ctxNotify, log.With(logger, "component", "discovery manager notify"), discovery.Name("notify"))
} else {
    legacymanager.RegisterMetrics()
    discoveryManagerScrape = legacymanager.NewManager(ctxScrape, log.With(logger, "component", "discovery manager scrape"), legacymanager.Name("scrape"))
    discoveryManagerNotify = legacymanager.NewManager(ctxNotify, log.With(logger, "component", "discovery manager notify"), legacymanager.Name("notify"))
}

```































