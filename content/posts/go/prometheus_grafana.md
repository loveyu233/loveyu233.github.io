---
title: "go服务监控"
date: 2025-07-09T09:40:58+08:00
author: ["loveyu"]
draft: false
categories:
- go
tags:
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

# Counter（计数器）- 适合统计**累计事件**

### 适用场景：

- **请求总数**：`http_requests_total`
- **错误请求数**：`http_errors_total`
- **发送字节数**：`http_response_bytes_total`
- **重定向次数**：`http_redirects_total`

### 特点：

- 只增不减（除非服务重启）
- 通常配合`rate()`计算变化率
- 适合任何需要"计数"的场景

```shell
# 计算每秒请求率(QPS)
rate(http_requests_total[1m])

# 计算错误率
rate(http_errors_total[5m]) / rate(http_requests_total[5m])
```

```go
package main

import (
	"net/http"
	
	"github.com/prometheus/client_golang/prometheus"
	"github.com/prometheus/client_golang/prometheus/promhttp"
)

func main() {
	// 创建带method和path标签的计数器
	httpRequests := prometheus.NewCounterVec(
		prometheus.CounterOpts{
			Name: "http_requests_total",
			Help: "Number of HTTP requests",
		},
		[]string{"method", "path"}, // 标签维度
	)

	// 注册指标
	prometheus.MustRegister(httpRequests)

	// 模拟请求处理
	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {
		// 根据请求方法和路径增加对应计数器
		httpRequests.WithLabelValues(r.Method, r.URL.Path).Inc()
		w.Write([]byte("Hello World"))
	})

	// 暴露metrics端点
	http.Handle("/metrics", promhttp.Handler())
	http.ListenAndServe(":8080", nil)
}
```



# Gauge（仪表盘） - 适合统计**当前状态值**

### 适用场景：

- **当前并发请求数**：`http_concurrent_requests`
- **内存使用量**：`http_memory_usage_bytes`
- **连接池大小**：`http_connection_pool_size`
- **队列积压量**：`http_request_queue_size`

### 特点：

- 可增可减，反映瞬时值
- 不需要计算变化率
- 适合测量"现在是多少"

```shell
# 当前活跃连接数
http_concurrent_requests

# 检测连接池是否接近满载
http_connection_pool_size / http_connection_pool_capacity > 0.8
```

```go
// 创建带instance标签的当前温度指标
tempGauge := prometheus.NewGaugeVec(
	prometheus.GaugeOpts{
		Name: "temperature_celsius",
		Help: "Current temperature in Celsius",
	},
	[]string{"instance"},
)

// 设置不同实例的温度值
tempGauge.WithLabelValues("sensor1").Set(23.5)
tempGauge.WithLabelValues("sensor2").Set(27.3)

// 温度变化
tempGauge.WithLabelValues("sensor1").Add(0.5) // 增加0.5度
tempGauge.WithLabelValues("sensor2").Sub(1.2) // 减少1.2度
```



# Histogram（直方图） - 适合统计**值的分布情况**

### 适用场景：

- **请求延迟分布**：`http_request_duration_seconds`
- **响应大小分布**：`http_response_size_bytes`
- **认证耗时分布**：`http_auth_duration_seconds`

### 特点：

- 自动计算分位数（P90/P95/P99）
- 需要预先定义桶(buckets)
- 适合分析"大多数情况如何"

```go
// 创建带method标签的请求耗时直方图
requestDuration := prometheus.NewHistogramVec(
	prometheus.HistogramOpts{
		Name:    "http_request_duration_seconds",
		Help:    "Request duration distribution",
		Buckets: []float64{0.1, 0.5, 1, 2.5, 5, 10}, // 自定义桶
	},
	[]string{"method"},
)

// 记录不同方法的请求耗时
requestDuration.WithLabelValues("GET").Observe(0.32)
requestDuration.WithLabelValues("POST").Observe(1.78)
requestDuration.WithLabelValues("GET").Observe(0.21)
```

```shell
# 计算95%分位延迟
histogram_quantile(0.95, 
  rate(http_request_duration_seconds_bucket[5m])
```

# Summary（摘要） - 适合统计**需要精确分位数的指标**

### 适用场景：

- **关键业务接口延迟**：`api_critical_duration_seconds`
- **低频率但重要的操作**：`admin_operation_duration_seconds`
- **不需要跨实例聚合的指标**

### 特点：

- 客户端计算分位数，更精确
- 不能跨实例聚合
- 比Histogram消耗更多资源

```go
// 创建带status标签的响应大小摘要
responseSize := prometheus.NewSummaryVec(
	prometheus.SummaryOpts{
		Name:       "http_response_size_bytes",
		Help:       "Response size distribution",
		Objectives: map[float64]float64{
			0.5: 0.05,   // 50th percentile, 5%误差
			0.9: 0.01,   // 90th percentile, 1%误差
			0.99: 0.001, // 99th percentile, 0.1%误差
		},
	},
	[]string{"status"},
)

// 记录不同状态码的响应大小
responseSize.WithLabelValues("200").Observe(1024)
responseSize.WithLabelValues("404").Observe(512)
responseSize.WithLabelValues("500").Observe(2048)
```

```shell
# 直接获取预计算的P99
http_response_size_bytes{quantile="0.99"}
```

