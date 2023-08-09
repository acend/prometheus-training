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


## Recording Rules

Prometheus [recording rules](https://prometheus.io/docs/prometheus/latest/configuration/recording_rules/) allow you to precompute queries at a defined interval (`global.evaluation_interval` or `interval` in `rule_group`) and save them to a new set of time series.

## Special labels

As you have already seen in several examples, a Prometheus metric is defined by one or more labels with the corresponding values. Two of those labels are special, because the Prometheus server will automatically generate them for every metric:


* instance

     The instance label describes the endpoint where Prometheus scraped the metric. This can be any application or exporter. In addition to the ip address or hostname, this label usually also contains the port number. Example: `10.0.0.25:9100`

* job

     This label contains the name of the scrape job as configured in the Prometheus configuration file. All instances configured in the same scrape job will share the same job label.


{{% alert title="Note" color="primary" %}}
Prometheus will append these labels dynamically before sample ingestion. Therefore you will not see these labels if you query the metrics endpoint directly (e.g. by using `curl`).

{{% /alert %}}

Let's take a look at the following scrape config (example, no need to change the Prometheus configuration on your lab VM):

```yaml
...
scrape_configs:
  ...
  - job_name: "node_exporter"
    static_configs:
      - targets:
        - "10.0.0.25:9100"
        - "10.0.0.26:9100"
        - "10.0.0.27:9100"
  ...
```

In the example above we configured a single scrape job with the name `node_exporter` and three targets. After ingestion into Prometheus, every metric scraped by this job will have the label: `job="node_exporter"`. In addition, metrics scraped by this job from the target `10.0.0.25` will have the label `instance="10.0.0.25:9100"`

## TODO Node Exporter theory

`node_exporter` is a Prometheus exporter for hardware and OS metrics. Or in other words, it supplies us with the more common metrics we know from classic monitoring systems.
It is therefore very useful for expanding Prometheus' monitoring capabilities into the infrastructure world.


## Relabeling (advanced)

[Relabeling](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#relabel_config) in Prometheus can be used to perform numerous tasks using regular expressions, such as

* adding, modifying or removing labels to/from metrics or alerts,
* filtering metrics based on labels, or
* enabling horizontal scaling of Prometheus by using `hashmod` relabeling.

It is a very powerful part of the Prometheus configuration, but it can also get quite complex and confusing. Thus, we will only take a look at some basic/simple examples.

There are four types of relabelings:

* `relabel_configs` (target relabeling)

  Target relabeling is defined in the job definition of a `scrape_config`. This is used to configure scraping of a multi-target exporter (e.g., `blackbox_exporter` or `snmp_exporter`) where one single exporter instance is used to scrape multiple targets. Check out the [Prometheus docs](https://prometheus.io/docs/guides/multi-target-exporter/#querying-multi-target-exporters-with-prometheus) for a detailed explanation and example configurations of `relabel_configs`.

* `metric_relabel_configs` (metrics relabeling)

  Metrics relabeling is applied to scraped samples right before ingestion. It allows adding, modifying, or dropping labels or even dropping entire samples if they match certain criteria.

* `alert_relabel_configs` (alert relabeling)

  Alert relabeling is similar to `metric_relabel_configs`, but applies to outgoing alerts.

* `write_relabel_configs` (remote write relabeling)

  Remote write relabeling is similar to `metric_relabel_configs`, but applies to `remote_write` configurations.

