





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



## metric

```sh
prometheus_target_interval_length_seconds
```



## 星光配置

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
- job_name: 'federate'
  metrics_path: '/federate'
  params:
    'match[]':
      - '{job="kuber-pod"}'
  static_configs:
  - targets: ['10.127.48.35:32120', '172.16.26.20:11625']
- job_name: 'kuber-pod'
  metrics_path: /metrics
  scheme: http
  kubernetes_sd_configs:  # k8s服务发现配置
  - role: pod  # 发现k8s的所有pod，并把它们的容器作为target,容器的每个端口会作为一个target
  relabel_configs:
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



## 知识点

可以配置rule来做聚合运算，这些rule会形成新的metric



### Query

prometheus提供了一种PromQL(Prometheus Query Language)来让用户实时的选择和聚合时序数据。