---
title: "Prometheus_grafana"
date: 2025-07-09T10:40:58+08:00
author: ["loveyu"]
draft: false
categories: 
- go
- grpc
tags: 
- 可视化
- 链路追踪
- prometheus
- grafana
---

# 安装prometheus

>``brew install prometheus`` 

```shell
~ > brew install prometheus
==> Downloading https://formulae.brew.sh/api/formula.jws.json
################################################################################### 100.0%
==> Downloading https://formulae.brew.sh/api/cask.jws.json
################################################################################### 100.0%
==> Fetching prometheus
==> Downloading https://mirrors.aliyun.com/homebrew/homebrew-bottles/prometheus-2.44.0.arm
################################################################################### 100.0%
==> Pouring prometheus-2.44.0.arm64_ventura.bottle.tar.gz
==> Caveats
When run from `brew services`, `prometheus` is run from
`prometheus_brew_services` and uses the flags in:
   /opt/homebrew/etc/prometheus.args

To start prometheus now and restart at login:
  brew services start prometheus
Or, if you don't want/need a background service you can just run:
  /opt/homebrew/opt/prometheus/bin/prometheus_brew_services
==> Summary
🍺  /opt/homebrew/Cellar/prometheus/2.44.0: 22 files, 220MB
==> Running `brew cleanup prometheus`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
```



# 安装grafana

>``brew install grafana`` 

```shell
~ > brew install grafana
==> Fetching grafana
==> Downloading https://mirrors.aliyun.com/homebrew/homebrew-bottles/grafana-9.5.3.arm64_v
################################################################################### 100.0%
==> Pouring grafana-9.5.3.arm64_ventura.bottle.tar.gz
==> Caveats
To start grafana now and restart at login:
  brew services start grafana
Or, if you don't want/need a background service you can just run:
  /opt/homebrew/opt/grafana/bin/grafana server --config /opt/homebrew/etc/grafana/grafana.ini --homepath /opt/homebrew/opt/grafana/share/grafana --packaging=brew cfg:default.paths.logs=/opt/homebrew/var/log/grafana cfg:default.paths.data=/opt/homebrew/var/lib/grafana cfg:default.paths.plugins=/opt/homebrew/var/lib/grafana/plugins
==> Summary
🍺  /opt/homebrew/Cellar/grafana/9.5.3: 6,921 files, 269.2MB
==> Running `brew cleanup grafana`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).
```



# 启动服务



```shell
~ > brew services start prometheus
==> Successfully started `prometheus` (label: homebrew.mxcl.prometheus)

~ > brew services start grafana
==> Successfully started `grafana` (label: homebrew.mxcl.grafana)
```

## 查看

prometheus: http://localhost:9090

grafana: http://localhost:3000



# 配置文件

## prometheus

> 文件位置: ``/opt/homebrew/etc/prometheus.yml`` 
>
> `job_name` 名称
>
> `static_configs` 设置地址
>
> 如果要使用`VictoriaMetrics`收集数据则配置
>
> remote_write:
>   - url: http://192.168.122.24:8428/api/v1/write
>
> 使用prometheus默认数据库则不需要配置删除即可,使用VictoriaMetrics需要单独下载启动



```yaml
global:
  scrape_interval: 15s
remote_write:
  - url: http://192.168.122.24:8428/api/v1/write
scrape_configs:
  - job_name: "prometheus"
    static_configs:
    - targets: ["localhost:9090"]

  - job_name: 'gin-app'
    static_configs:
    - targets: ['localhost:9999']
```

# 类型对比

| 特性       | Counter  | Gauge    | Histogram        | Summary    |
| :--------- | :------- | :------- | :--------------- | :--------- |
| 增减性     | 只增不减 | 可增可减 | 只增不减         | 只增不减   |
| 适用场景   | 累计计数 | 当前状态 | 分布分析         | 分布分析   |
| 客户端计算 | 无       | 无       | 桶计数           | 分位数计算 |
| 服务端聚合 | 支持     | 支持     | 支持             | 不支持     |
| 分位数计算 | 不可     | 不可     | 可(近似)         | 可(精确)   |
| 内存消耗   | 低       | 低       | 中(取决于桶数量) | 中         |

# 示例

>先添加`localhost:9999`到`Prometheus`配置文件中后重启,然后启动服务访问`/ping`
>
>启动`grafana`添加数据源Prometheus,在仪表板中创建选择Prometheus,进入后再`Metric`中选择`go-qps`即可查看

```go
package main

import (
	"github.com/gin-gonic/gin"
	"github.com/prometheus/client_golang/prometheus"
	"github.com/prometheus/client_golang/prometheus/collectors"
	"github.com/prometheus/client_golang/prometheus/promhttp"
	"math/rand"
	"net/http"
	"regexp"
	"time"
)

var statusCounter = prometheus.NewCounterVec(
	prometheus.CounterOpts{
		Name: "go-qps",
	},
	[]string{"method", "path"},
)

func initRegistry() *prometheus.Registry {
	// 创建一个 registry
	reg := prometheus.NewRegistry()

	// 添加 Go 编译信息
	reg.MustRegister(collectors.NewBuildInfoCollector())
	// Go runtime metrics
	reg.MustRegister(collectors.NewGoCollector(
		collectors.WithGoCollectorRuntimeMetrics(
			collectors.GoRuntimeMetricsRule{Matcher: regexp.MustCompile("/.*")},
		),
	))

	// 注册自定义的业务指标
	reg.MustRegister(statusCounter)

	return reg
}

func main() {

	r := gin.Default()

	r.GET("/ping", func(c *gin.Context) {
		// 记录
		statusCounter.WithLabelValues(
			c.Request.Method,
			c.Request.URL.Path,
		).Inc()

		c.JSON(200, gin.H{
			"status":  200,
			"message": "pong",
		})
	})

	reg := initRegistry()

	// 对外提供 /metrics 接口，支持 prometheus 采集
	r.GET("/metrics", gin.WrapH(promhttp.HandlerFor(
		reg,
		promhttp.HandlerOpts{Registry: reg},
	)))

	for i := 0; i < 5; i++ {
		go func() {
			for true {
				http.Get("http://localhost:9999/ping")
				time.Sleep(time.Duration(Rand()) * time.Millisecond)
			}
		}()
	}

	r.Run(":9999")
}

func Rand() int {
	// 初始化随机数种子（重要！）
	rand.Seed(time.Now().UnixNano())

	// 生成100-1000的随机数
	return rand.Intn(901) + 10 // 901 = 1000 - 100 + 1
}
```

