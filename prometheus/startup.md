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
生成local和remote的Storage对象

```
localStorage  = &readyStorage{stats: tsdb.NewDBStats()}
scraper       = &readyScrapeManager{}
remoteStorage = remote.NewStorage(log.With(logger, "component", "remote"), prometheus.DefaultRegisterer, localStorage.StartTime, localStoragePath, time.Duration(cfg.RemoteFlushDeadline), scraper)
fanoutStorage = storage.NewFanout(logger, localStorage, remoteStorage)
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

生成scrape manager和tracing manager对象

```
scrapeManager  = scrape.NewManager(&cfg.scrape, log.With(logger, "component", "scrape manager"), fanoutStorage)
tracingManager = tracing.NewManager(logger)
```

在Prometheus开启auto-gomaxprocs的feature情况下，动态设置GOMAXPROCS

```
if cfg.enableAutoGOMAXPROCS {
    l := func(format string, a ...interface{}) {
    level.Info(logger).Log("component", "automaxprocs", "msg", fmt.Sprintf(strings.TrimPrefix(format, "maxprocs: "), a...))
    }
    if _, err := maxprocs.Set(maxprocs.Logger(l)); err != nil {
        level.Warn(logger).Log("component", "automaxprocs", "msg", "Failed to set GOMAXPROCS automatically", "err", err)
    }
}
```

在以server模式启动Prometheus的情况下，生成queryEngine对象和ruleManager对象

```
opts := promql.EngineOpts{
    Logger:                   log.With(logger, "component", "query engine"),
    Reg:                      prometheus.DefaultRegisterer,
    MaxSamples:               cfg.queryMaxSamples,
    Timeout:                  time.Duration(cfg.queryTimeout),
    ActiveQueryTracker:       promql.NewActiveQueryTracker(localStoragePath, cfg.queryConcurrency, log.With(logger, "component", "activeQueryTracker")),
    LookbackDelta:            time.Duration(cfg.lookbackDelta),
    NoStepSubqueryIntervalFn: noStepSubqueryInterval.Get,
    // EnableAtModifier and EnableNegativeOffset have to be
    // always on for regular PromQL as of Prometheus v2.33.
    EnableAtModifier:     true,
    EnableNegativeOffset: true,
    EnablePerStepStats:   cfg.enablePerStepStats,
}

queryEngine = promql.NewEngine(opts)

ruleManager = rules.NewManager(&rules.ManagerOptions{
    Appendable:      fanoutStorage,
    Queryable:       localStorage,
    QueryFunc:       rules.EngineQueryFunc(queryEngine, fanoutStorage),
    NotifyFunc:      sendAlerts(notifierManager, cfg.web.ExternalURL.String()),
    Context:         ctxRule,
    ExternalURL:     cfg.web.ExternalURL,
    Registerer:      prometheus.DefaultRegisterer,
    Logger:          log.With(logger, "component", "rule manager"),
    OutageTolerance: time.Duration(cfg.outageTolerance),
    ForGracePeriod:  time.Duration(cfg.forGracePeriod),
    ResendDelay:     time.Duration(cfg.resendDelay),
})

```

初始化Prometheus的web Handler对象，用于处理客户端请求

```
webHandler := web.New(log.With(logger, "component", "web"), &cfg.web)
```

Prometheus的配置可以热更新，生成reloader对象

```
	reloaders := []reloader{
		{
			name:     "db_storage",
			reloader: localStorage.ApplyConfig,
		}, {
			name:     "remote_storage",
			reloader: remoteStorage.ApplyConfig,
		}, {
			name:     "web_handler",
			reloader: webHandler.ApplyConfig,
		}, {
			name: "query_engine",
			reloader: func(cfg *config.Config) error {
				if agentMode {
					// No-op in Agent mode.
					return nil
				}

				if cfg.GlobalConfig.QueryLogFile == "" {
					queryEngine.SetQueryLogger(nil)
					return nil
				}

				l, err := logging.NewJSONFileLogger(cfg.GlobalConfig.QueryLogFile)
				if err != nil {
					return err
				}
				queryEngine.SetQueryLogger(l)
				return nil
			},
		}, {
			// The Scrape and notifier managers need to reload before the Discovery manager as
			// they need to read the most updated config when receiving the new targets list.
			name:     "scrape",
			reloader: scrapeManager.ApplyConfig,
		}, {
			name: "scrape_sd",
			reloader: func(cfg *config.Config) error {
				c := make(map[string]discovery.Configs)
				for _, v := range cfg.ScrapeConfigs {
					c[v.JobName] = v.ServiceDiscoveryConfigs
				}
				return discoveryManagerScrape.ApplyConfig(c)
			},
		}, {
			name:     "notify",
			reloader: notifierManager.ApplyConfig,
		}, {
			name: "notify_sd",
			reloader: func(cfg *config.Config) error {
				c := make(map[string]discovery.Configs)
				for k, v := range cfg.AlertingConfig.AlertmanagerConfigs.ToMap() {
					c[k] = v.ServiceDiscoveryConfigs
				}
				return discoveryManagerNotify.ApplyConfig(c)
			},
		}, {
			name: "rules",
			reloader: func(cfg *config.Config) error {
				if agentMode {
					// No-op in Agent mode
					return nil
				}

				// Get all rule files matching the configuration paths.
				var files []string
				for _, pat := range cfg.RuleFiles {
					fs, err := filepath.Glob(pat)
					if err != nil {
						// The only error can be a bad pattern.
						return errors.Wrapf(err, "error retrieving rule files for %s", pat)
					}
					files = append(files, fs...)
				}
				return ruleManager.Update(
					time.Duration(cfg.GlobalConfig.EvaluationInterval),
					files,
					cfg.GlobalConfig.ExternalLabels,
					externalURL,
					nil,
				)
			},
		}, {
			name:     "tracing",
			reloader: tracingManager.ApplyConfig,
		},
	}
```
用于控制Prometheus Components启动顺序的channel
```
	// Start all components while we wait for TSDB to open but only load
	// initial config and mark ourselves as ready after it completed.
	dbOpen := make(chan struct{})

	// sync.Once is used to make sure we can close the channel at different execution stages(SIGTERM or when the config is loaded).
	type closeOnce struct {
		C     chan struct{}
		once  sync.Once
		Close func()
	}
	// Wait until the server is ready to handle reloading.
	reloadReady := &closeOnce{
		C: make(chan struct{}),
	}
	reloadReady.Close = func() {
		reloadReady.once.Do(func() {
			close(reloadReady.C)
		})
	}
```

初始化Prometheus的http的listener，并且校验TLS配置是否合法

```
	listener, err := webHandler.Listener()
	if err != nil {
		level.Error(logger).Log("msg", "Unable to start web listener", "err", err)
		os.Exit(1)
	}

	err = toolkit_web.Validate(*webConfig)
	if err != nil {
		level.Error(logger).Log("msg", "Unable to validate web configuration file", "err", err)
		os.Exit(1)
	}
```


创建run.Group，并且添加一系列的actors，并且启动这些actor，这部分是Prometheus启动的核心代码部分，详细分析每一个actor实现的功能


```
	var g run.Group
	{
		// Termination handler.
		term := make(chan os.Signal, 1)
		signal.Notify(term, os.Interrupt, syscall.SIGTERM)
		cancel := make(chan struct{})
		g.Add(
			func() error {
				// Don't forget to release the reloadReady channel so that waiting blocks can exit normally.
				select {
				case <-term:
					level.Warn(logger).Log("msg", "Received SIGTERM, exiting gracefully...")
					reloadReady.Close()
				case <-webHandler.Quit():
					level.Warn(logger).Log("msg", "Received termination request via web service, exiting gracefully...")
				case <-cancel:
					reloadReady.Close()
				}
				return nil
			},
			func(err error) {
				close(cancel)
			},
		)
	}
	{
		// Scrape discovery manager.
		g.Add(
			func() error {
				err := discoveryManagerScrape.Run()
				level.Info(logger).Log("msg", "Scrape discovery manager stopped")
				return err
			},
			func(err error) {
				level.Info(logger).Log("msg", "Stopping scrape discovery manager...")
				cancelScrape()
			},
		)
	}
	{
		// Notify discovery manager.
		g.Add(
			func() error {
				err := discoveryManagerNotify.Run()
				level.Info(logger).Log("msg", "Notify discovery manager stopped")
				return err
			},
			func(err error) {
				level.Info(logger).Log("msg", "Stopping notify discovery manager...")
				cancelNotify()
			},
		)
	}
	{
		// Scrape manager.
		g.Add(
			func() error {
				// When the scrape manager receives a new targets list
				// it needs to read a valid config for each job.
				// It depends on the config being in sync with the discovery manager so
				// we wait until the config is fully loaded.
				<-reloadReady.C

				err := scrapeManager.Run(discoveryManagerScrape.SyncCh())
				level.Info(logger).Log("msg", "Scrape manager stopped")
				return err
			},
			func(err error) {
				// Scrape manager needs to be stopped before closing the local TSDB
				// so that it doesn't try to write samples to a closed storage.
				level.Info(logger).Log("msg", "Stopping scrape manager...")
				scrapeManager.Stop()
			},
		)
	}
	{
		// Tracing manager.
		g.Add(
			func() error {
				<-reloadReady.C
				tracingManager.Run()
				return nil
			},
			func(err error) {
				tracingManager.Stop()
			},
		)
	}
	{
		// Reload handler.

		// Make sure that sighup handler is registered with a redirect to the channel before the potentially
		// long and synchronous tsdb init.
		hup := make(chan os.Signal, 1)
		signal.Notify(hup, syscall.SIGHUP)
		cancel := make(chan struct{})
		g.Add(
			func() error {
				<-reloadReady.C

				for {
					select {
					case <-hup:
						if err := reloadConfig(cfg.configFile, cfg.enableExpandExternalLabels, cfg.tsdb.EnableExemplarStorage, logger, noStepSubqueryInterval, reloaders...); err != nil {
							level.Error(logger).Log("msg", "Error reloading config", "err", err)
						}
					case rc := <-webHandler.Reload():
						if err := reloadConfig(cfg.configFile, cfg.enableExpandExternalLabels, cfg.tsdb.EnableExemplarStorage, logger, noStepSubqueryInterval, reloaders...); err != nil {
							level.Error(logger).Log("msg", "Error reloading config", "err", err)
							rc <- err
						} else {
							rc <- nil
						}
					case <-cancel:
						return nil
					}
				}
			},
			func(err error) {
				// Wait for any in-progress reloads to complete to avoid
				// reloading things after they have been shutdown.
				cancel <- struct{}{}
			},
		)
	}
	{
		// Initial configuration loading.
		cancel := make(chan struct{})
		g.Add(
			func() error {
				select {
				case <-dbOpen:
				// In case a shutdown is initiated before the dbOpen is released
				case <-cancel:
					reloadReady.Close()
					return nil
				}

				if err := reloadConfig(cfg.configFile, cfg.enableExpandExternalLabels, cfg.tsdb.EnableExemplarStorage, logger, noStepSubqueryInterval, reloaders...); err != nil {
					return errors.Wrapf(err, "error loading config from %q", cfg.configFile)
				}

				reloadReady.Close()

				webHandler.Ready()
				level.Info(logger).Log("msg", "Server is ready to receive web requests.")
				<-cancel
				return nil
			},
			func(err error) {
				close(cancel)
			},
		)
	}
	if !agentMode {
		// Rule manager.
		g.Add(
			func() error {
				<-reloadReady.C
				ruleManager.Run()
				return nil
			},
			func(err error) {
				ruleManager.Stop()
			},
		)

		// TSDB.
		opts := cfg.tsdb.ToTSDBOptions()
		cancel := make(chan struct{})
		g.Add(
			func() error {
				level.Info(logger).Log("msg", "Starting TSDB ...")
				if cfg.tsdb.WALSegmentSize != 0 {
					if cfg.tsdb.WALSegmentSize < 10*1024*1024 || cfg.tsdb.WALSegmentSize > 256*1024*1024 {
						return errors.New("flag 'storage.tsdb.wal-segment-size' must be set between 10MB and 256MB")
					}
				}
				if cfg.tsdb.MaxBlockChunkSegmentSize != 0 {
					if cfg.tsdb.MaxBlockChunkSegmentSize < 1024*1024 {
						return errors.New("flag 'storage.tsdb.max-block-chunk-segment-size' must be set over 1MB")
					}
				}

				db, err := openDBWithMetrics(localStoragePath, logger, prometheus.DefaultRegisterer, &opts, localStorage.getStats())
				if err != nil {
					return errors.Wrapf(err, "opening storage failed")
				}

				switch fsType := prom_runtime.Statfs(localStoragePath); fsType {
				case "NFS_SUPER_MAGIC":
					level.Warn(logger).Log("fs_type", fsType, "msg", "This filesystem is not supported and may lead to data corruption and data loss. Please carefully read https://prometheus.io/docs/prometheus/latest/storage/ to learn more about supported filesystems.")
				default:
					level.Info(logger).Log("fs_type", fsType)
				}

				level.Info(logger).Log("msg", "TSDB started")
				level.Debug(logger).Log("msg", "TSDB options",
					"MinBlockDuration", cfg.tsdb.MinBlockDuration,
					"MaxBlockDuration", cfg.tsdb.MaxBlockDuration,
					"MaxBytes", cfg.tsdb.MaxBytes,
					"NoLockfile", cfg.tsdb.NoLockfile,
					"RetentionDuration", cfg.tsdb.RetentionDuration,
					"WALSegmentSize", cfg.tsdb.WALSegmentSize,
					"AllowOverlappingBlocks", cfg.tsdb.AllowOverlappingBlocks,
					"WALCompression", cfg.tsdb.WALCompression,
				)

				startTimeMargin := int64(2 * time.Duration(cfg.tsdb.MinBlockDuration).Seconds() * 1000)
				localStorage.Set(db, startTimeMargin)
				close(dbOpen)
				<-cancel
				return nil
			},
			func(err error) {
				if err := fanoutStorage.Close(); err != nil {
					level.Error(logger).Log("msg", "Error stopping storage", "err", err)
				}
				close(cancel)
			},
		)
	}
	if agentMode {
		// WAL storage.
		opts := cfg.agent.ToAgentOptions()
		cancel := make(chan struct{})
		g.Add(
			func() error {
				level.Info(logger).Log("msg", "Starting WAL storage ...")
				if cfg.agent.WALSegmentSize != 0 {
					if cfg.agent.WALSegmentSize < 10*1024*1024 || cfg.agent.WALSegmentSize > 256*1024*1024 {
						return errors.New("flag 'storage.agent.wal-segment-size' must be set between 10MB and 256MB")
					}
				}
				db, err := agent.Open(
					logger,
					prometheus.DefaultRegisterer,
					remoteStorage,
					localStoragePath,
					&opts,
				)
				if err != nil {
					return errors.Wrap(err, "opening storage failed")
				}

				switch fsType := prom_runtime.Statfs(localStoragePath); fsType {
				case "NFS_SUPER_MAGIC":
					level.Warn(logger).Log("fs_type", fsType, "msg", "This filesystem is not supported and may lead to data corruption and data loss. Please carefully read https://prometheus.io/docs/prometheus/latest/storage/ to learn more about supported filesystems.")
				default:
					level.Info(logger).Log("fs_type", fsType)
				}

				level.Info(logger).Log("msg", "Agent WAL storage started")
				level.Debug(logger).Log("msg", "Agent WAL storage options",
					"WALSegmentSize", cfg.agent.WALSegmentSize,
					"WALCompression", cfg.agent.WALCompression,
					"StripeSize", cfg.agent.StripeSize,
					"TruncateFrequency", cfg.agent.TruncateFrequency,
					"MinWALTime", cfg.agent.MinWALTime,
					"MaxWALTime", cfg.agent.MaxWALTime,
				)

				localStorage.Set(db, 0)
				close(dbOpen)
				<-cancel
				return nil
			},
			func(e error) {
				if err := fanoutStorage.Close(); err != nil {
					level.Error(logger).Log("msg", "Error stopping storage", "err", err)
				}
				close(cancel)
			},
		)
	}
	{
		// Web handler.
		g.Add(
			func() error {
				if err := webHandler.Run(ctxWeb, listener, *webConfig); err != nil {
					return errors.Wrapf(err, "error starting web server")
				}
				return nil
			},
			func(err error) {
				cancelWeb()
			},
		)
	}
	{
		// Notifier.

		// Calling notifier.Stop() before ruleManager.Stop() will cause a panic if the ruleManager isn't running,
		// so keep this interrupt after the ruleManager.Stop().
		g.Add(
			func() error {
				// When the notifier manager receives a new targets list
				// it needs to read a valid config for each job.
				// It depends on the config being in sync with the discovery manager
				// so we wait until the config is fully loaded.
				<-reloadReady.C

				notifierManager.Run(discoveryManagerNotify.SyncCh())
				level.Info(logger).Log("msg", "Notifier manager stopped")
				return nil
			},
			func(err error) {
				notifierManager.Stop()
			},
		)
	}
	if err := g.Run(); err != nil {
		level.Error(logger).Log("err", err)
		os.Exit(1)
	}
```

监听操作系统的退出信号和通过web接口的退出信号，优雅退出

```

	{
		// Termination handler.
		term := make(chan os.Signal, 1)
		signal.Notify(term, os.Interrupt, syscall.SIGTERM)
		cancel := make(chan struct{})
		g.Add(
			func() error {
				// Don't forget to release the reloadReady channel so that waiting blocks can exit normally.
				select {
				case <-term:
					level.Warn(logger).Log("msg", "Received SIGTERM, exiting gracefully...")
					reloadReady.Close()
				case <-webHandler.Quit():
					level.Warn(logger).Log("msg", "Received termination request via web service, exiting gracefully...")
				case <-cancel:
					reloadReady.Close()
				}
				return nil
			},
			func(err error) {
				close(cancel)
			},
		)
	}

```

启动Prometheusd的web服务

```
		g.Add(
			func() error {
				if err := webHandler.Run(ctxWeb, listener, *webConfig); err != nil {
					return errors.Wrapf(err, "error starting web server")
				}
				return nil
			},
			func(err error) {
				cancelWeb()
			},
		)
	}
```

启动TSDB数据存储组件

```

				level.Info(logger).Log("msg", "Starting TSDB ...")
				if cfg.tsdb.WALSegmentSize != 0 {
					if cfg.tsdb.WALSegmentSize < 10*1024*1024 || cfg.tsdb.WALSegmentSize > 256*1024*1024 {
						return errors.New("flag 'storage.tsdb.wal-segment-size' must be set between 10MB and 256MB")
					}
				}
				if cfg.tsdb.MaxBlockChunkSegmentSize != 0 {
					if cfg.tsdb.MaxBlockChunkSegmentSize < 1024*1024 {
						return errors.New("flag 'storage.tsdb.max-block-chunk-segment-size' must be set over 1MB")
					}
				}

				db, err := openDBWithMetrics(localStoragePath, logger, prometheus.DefaultRegisterer, &opts, localStorage.getStats())
				if err != nil {
					return errors.Wrapf(err, "opening storage failed")
				}

				switch fsType := prom_runtime.Statfs(localStoragePath); fsType {
				case "NFS_SUPER_MAGIC":
					level.Warn(logger).Log("fs_type", fsType, "msg", "This filesystem is not supported and may lead to data corruption and data loss. Please carefully read https://prometheus.io/docs/prometheus/latest/storage/ to learn more about supported filesystems.")
				default:
					level.Info(logger).Log("fs_type", fsType)
				}

				level.Info(logger).Log("msg", "TSDB started")
				level.Debug(logger).Log("msg", "TSDB options",
					"MinBlockDuration", cfg.tsdb.MinBlockDuration,
					"MaxBlockDuration", cfg.tsdb.MaxBlockDuration,
					"MaxBytes", cfg.tsdb.MaxBytes,
					"NoLockfile", cfg.tsdb.NoLockfile,
					"RetentionDuration", cfg.tsdb.RetentionDuration,
					"WALSegmentSize", cfg.tsdb.WALSegmentSize,
					"AllowOverlappingBlocks", cfg.tsdb.AllowOverlappingBlocks,
					"WALCompression", cfg.tsdb.WALCompression,
				)

				startTimeMargin := int64(2 * time.Duration(cfg.tsdb.MinBlockDuration).Seconds() * 1000)
				localStorage.Set(db, startTimeMargin)
				close(dbOpen)
				<-cancel
				return nil

```

启动初始配置加载组件，将配置应用到每一个component，成功应用每一个配置后，会关闭reloadReady.C，其他组件会并行启动运行

```
				select {
				case <-dbOpen:
				// In case a shutdown is initiated before the dbOpen is released
				case <-cancel:
					reloadReady.Close()
					return nil
				}

				if err := reloadConfig(cfg.configFile, cfg.enableExpandExternalLabels, cfg.tsdb.EnableExemplarStorage, logger, noStepSubqueryInterval, reloaders...); err != nil {
					return errors.Wrapf(err, "error loading config from %q", cfg.configFile)
				}

				reloadReady.Close()

				webHandler.Ready()
				level.Info(logger).Log("msg", "Server is ready to receive web requests.")
				<-cancel
				return nil
```





































