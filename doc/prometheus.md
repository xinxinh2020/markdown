



[toc]

## 安装

```shell
wget https://github.com/prometheus/prometheus/releases/download/v2.17.0-rc.0/prometheus-2.17.0-rc.0.linux-amd64.tar.gz
tar zxvf prometheus-2.17.0-rc.0.linux-amd64.tar.gz
cd prometheus-2.17.0-rc.0.linux-amd64/
mv prometheus.yml prometheus.yml.bak
cat > prometheus.yml << EOF
global:
  scrape_interval:     15s # By default, scrape targets every 15 seconds.

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
    monitor: 'codelab-monitor'

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label job=<job_name> to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s

    static_configs:
      - targets: ['localhost:9090']
EOF
./prometheus --config.file=prometheus.yml
curl localhost:9090/metrics
curl http://localhost:9090/graph
```



## metric列表

```sh
prometheus_target_interval_length_seconds

# Counter
# Counter 表示收集的数据是按照某个趋势（增加／减少）一直变化的，我们往往用它记录服务请求总量、错误总数等。
# 例如 Prometheus server 中 http_requests_total, 表示 Prometheus 处理的 http 请求总数，我们可以使用 delta, 很容易得到任意区间数据的增量
# 星光的starlight_metrics_rc

# Summary
# Summary 和 Histogram 类似，由 <basename>{quantile="<φ>"}，<basename>_sum，<basename>_count 组成，主要用于表示一段时间内数据采样结果（通常是请求持续时间或响应大小），它直接存储了 quantile 数据，而不是根据统计区间计算出来的。
# 星光：starlight_metrics_rls、starlight_metrics_rls_sum、starlight_metrics_rls_count 
starlight_metrics_rls  # 应该是请求的响应时间



# Gauge
go_goroutines
```





## 知识点

可以配置rule来做聚合运算，这些rule会形成新的metric。

count用于统计记录数量，sum用于求记录的值的和

每个服务上都要安装exporter来为prometheus提供数据



### Query

prometheus提供了一种PromQL(Prometheus Query Language)来让用户实时的选择和聚合时序数据。

query语句：

```shell
scrape_duration_seconds{job="federate", instance="10.127.48.35:32120"} # 表示拉取数据的时间间隔
up{job="federate", instance="10.127.48.35:32120"} # 查询实例是否正常工作，1表示正常工作
```





```shell
label_values(starlight_metrics_rc, svc) # 查询某个metrics的label的值
label_values(starlight_metrics_rc{svc=~"^($svc)$"}, opt)
```







## 星光的Prometheus

### 配置

```shell
# 命令行配置
/bin/prometheus --config.file=/etc/prometheus/prometheus.yml --storage.tsdb.path=/prometheus --storage.tsdb.retention=7d --web.enable-admin-api --web.enable-lifecycle

# 配置文件：
/etc/prometheus # cat /etc/prometheus/prometheus.yml
global:
  scrape_interval: 15s
  scrape_timeout: 15s
scrape_configs:
- job_name: 'prometheus'
  static_configs:
  - targets: ['localhost:9090']
- job_name: 'federate' # 从其他Prometheus抓取数据
  metrics_path: '/federate'
  params:
    'match[]':
      - '{job="kuber-pod"}'
  static_configs:
  - targets: ['10.127.48.35:32120', '172.16.26.20:11625']
- job_name: 'kuber-pod'
  metrics_path: /metrics  # 拉取节点的 metric 路径
  scheme: http
  kubernetes_sd_configs:  # k8s服务发现配置
  - role: pod  # 发现k8s的所有pod，并把它们的容器作为target,容器的每个端口会作为一个target
  relabel_configs: # 拉取数据重置标签配置
  - source_labels: [__meta_kubernetes_namespace] # k8s对象的namespace
    action: keep
    regex: "(sl-dev|sl-v4-pro)"
  - source_labels: [__meta_kubernetes_pod_container_port_number] # 容器的端口
    action: keep
    regex: "9000"
- job_name: 'kuber-pod-new'
  metrics_path: /metrics
  scheme: http
  kubernetes_sd_configs:
  - role: pod
  relabel_configs:
  - source_labels: [__meta_kubernetes_namespace]
    action: keep
    regex: "sl-.*"
  - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape] # 带有prometheus.io/scrape annotation的pod
    action: keep
    regex: "true"
  - source_labels: [__meta_kubernetes_pod_container_port_name] # 端口名
    action: keep
    regex: "monitor"
```



### Metrics

#### starlight_metrics_rc

```shell
# label
instance
job
exported_instance
exported_job
namespace
opt
pod
svc
```



#### starlight_metrics_rls

```shell
# label
instance
job
code
exported_instance
exported_job
namespace
opt
pod
quantile
svc
```



