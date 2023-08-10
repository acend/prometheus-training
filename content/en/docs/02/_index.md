---
title: "2. Metrics"
weight: 1
sectionnumber: 1
---


In this lab you are going to learn about the Prometheus exposition format and how metrics and their values are represented withing the Prometheus ecosystem.

## Prometheus exposition format

Prometheus consumes metrics in Prometheus text-based exposition format and plans to adopt the [OpenMetrics](https://openmetrics.io/) standard: <https://prometheus.io/docs/introduction/roadmap/#adopt-openmetrics>.

Optionally check [Prometheus Exposition Format](https://prometheus.io/docs/instrumenting/exposition_formats/) for a more detailed explanation of the format.

All metrics withing Prometheus are scraped, stored and queried in the following format:
```promql
# HELP <metric name> <info>
# TYPE <metric name> <metric type>
<metric name>{<label name>=<label value>, ...} <sample value>
```

The Prometheus server exposes and collects its own metrics too. You can easily explore the metrics with your browser under (<http://{{% param replacePlaceholder.prometheus %}}/metrics>).

Metrics similar to the following will be shown:

{{% onlyWhenNot baloise %}}
```promql
...
# HELP prometheus_tsdb_head_samples_appended_total Total number of appended samples.
# TYPE prometheus_tsdb_head_samples_appended_total counter
prometheus_tsdb_head_samples_appended_total 463
# HELP prometheus_tsdb_head_series Total number of series in the head block.
# TYPE prometheus_tsdb_head_series gauge
prometheus_tsdb_head_series 463
...
```
{{% /onlyWhenNot %}}

{{% onlyWhen baloise %}}
```promql
...
# HELP prometheus_tsdb_head_min_time_seconds Minimum time bound of the head block.
# TYPE prometheus_tsdb_head_min_time_seconds gauge
prometheus_tsdb_head_min_time_seconds 1.669622401e+09
# HELP prometheus_tsdb_head_samples_appended_total Total number of appended samples.
# TYPE prometheus_tsdb_head_samples_appended_total counter
prometheus_tsdb_head_samples_appended_total 2.5110946e+07
...
```
{{% /onlyWhen %}}


### Metric Types


There are 4 different metric types in Prometheus

* Counter, (Basic use cases, always goes up)
* Gauge, (Basic use cases, can go up and down)
* Histogram, (Advanced use cases)
* Summary, (Advanced use cases)

For now we focus on Counter and Gauge.

Find additional information in the official [Prometheus Metric Types](https://prometheus.io/docs/concepts/metric_types/) docs.

## Special labels

As you have already seen in several examples, a Prometheus metric is defined by one or more labels with the corresponding values. Two of those labels are special, because the Prometheus server will automatically generate them for every metric:


* instance

     The instance label describes the endpoint where Prometheus scraped the metric. This can be any application or exporter. In addition to the ip address or hostname, this label usually also contains the port number. Example: `10.0.0.25:9100`.

* job

     This label contains the name of the scrape job as configured in the Prometheus configuration file. All instances configured in the same scrape job will share the same job label. In a Kubernetes environment this relates to the `Service`-Name.


{{% alert title="Note" color="primary" %}}
Prometheus will append these labels dynamically before sample ingestion. Therefore you will not see these labels if you query the metrics endpoint directly (e.g. by using `curl`).

{{% /alert %}}

Let's take a look at the following `ServiceMonitor` (example, no need to apply this to the cluster):

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    app.kubernetes.io/name: example-web-python
  name: example-web-python-monitor
spec:
  endpoints:
  - interval: 30s
    port: http
    scheme: http
    path: /metrics
  selector:
    matchLabels:
      name: example-web-python-monitor
```

In the example above we instructed Prometheus to scrape all Pods that are matched by the `Service` named `example-web-python-monitor`. After ingestion into Prometheus, every metric scraped by this job will have the label: `job="example-web-python-monitor"`. In addition, metrics scraped by this job from the Pod with IP `10.0.0.25` will have the label `instance="10.0.0.25:80"`

## Node Exporter

The tasks of this chapter will all be based on metrics that are provided by the `node_exporter`. An exporter is generally used to expose metrics from an application or system that would otherwise not expose metrics natively in the Prometheus exposition format. You will learn more about other exporters in the lab TODO.

In case of the `node_exporter`, the system we're interested in are Linux machines. It gathers the necessary information from different files and folders (e.g. `/proc/net/arp`, `/proc/sys/fs/file-nr`, etc.) and therefore is able to expose information about common metrics like CPU, Memory, Disk, Network, etc., which makes it very useful for expanding Prometheus' monitoring capabilities into the infrastructure world.
