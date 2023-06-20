---
title: "Prometheus_grafana"
date: 2023-06-21T010:40:58+08:00
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

