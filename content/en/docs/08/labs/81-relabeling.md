---
title: "8.1 Tasks: Relabeling"
weight: 81
sectionnumber: 8.1
---

[Relabeling](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#relabel_config) in Prometheus can be used to perform numerous tasks using regular expressions, such as

* adding, modifying or removing labels to/from metrics or alerts,
* filtering metrics based on labels, or
* enabling horizontal scaling of Prometheus by using `hashmod` relabeling.

It is a very powerful part of the Prometheus configuration, but it can also get quite complex and confusing. Thus, we will only take a look at some basic/simple examples.

There are four types of relabelings:

* `relabel_configs` (target relabeling)

  Target relabeling is defined in the job definition of a `scrape_config`. When using the Prometheus Operator, custom `relabel_configs` can be added to the `ServiceMonitor`. This concept is also used to configure scraping of a multi-target exporter (e.g., `blackbox_exporter` or `snmp_exporter`) where one single exporter instance is used to scrape multiple targets. Check out the [Prometheus docs](https://prometheus.io/docs/guides/multi-target-exporter/#querying-multi-target-exporters-with-prometheus) for a detailed explanation and example configurations of `relabel_configs`.

* `metric_relabel_configs` (metrics relabeling)

  Metrics relabeling is applied to scraped samples right before ingestion. It allows adding, modifying, or dropping labels or even dropping entire samples if they match certain criteria.

* `alert_relabel_configs` (alert relabeling)

  Alert relabeling is similar to `metric_relabel_configs`, but applies to outgoing alerts.

* `write_relabel_configs` (remote write relabeling)

  Remote write relabeling is similar to `metric_relabel_configs`, but applies to `remote_write` configurations.
