---
title: "4. Prometheus exporters"
weight: 1
sectionnumber: 1
---

An increasing number of applications directly instrument a Prometheus metrics endpoint. This enables applications to be scraped by Prometheus out of the box. For all other applications, an additional component (the Prometheus exporter) is needed to close the gap between Prometheus and the application which should be monitored.

{{% alert title="Note" color="primary" %}}
There are lots of exporters available for many applications, such as MySQL/MariaDB, Nginx, Ceph, etc. Some of these exporters are maintained by the [Prometheus GitHub organization](https://github.com/prometheus?q=exporter) while others are maintained by the community or third-party vendors. Check out the [list of exporters](https://prometheus.io/docs/instrumenting/exporters/) on the Prometheus website for an up-to-date list of exporters.
{{% /alert %}}


One example of a Prometheus exporter is the `node_exporter` we used in the second chapter of this training. This exporter collects information from different files and folders (e.g., `/proc/net/arp`, `/proc/sys/fs/file-nr`, etc.) and uses this information to create the appropriate Prometheus metrics.
In the tasks of this chapter we will configure two additional exporters.

## Special exporters

### Blackbox exporter

This is a classic example of a multi-target exporter which uses relabeling to pass the targets to the exporter. This exporter is capable of probing the following endpoints:

* HTTP
* HTTPS
* DNS
* TCP
* ICMP

By using the TCP prober you can create custom checks for almost any service including services using STARTTLS. Check out the [example.yml](https://github.com/prometheus/blackbox_exporter/blob/master/example.yml) file in the project's GitHub repository.

### Prometheus Pushgateway

The Pushgateway allows jobs (e.g., Kubernetes Jobs or CronJobs) to push metrics to an exporter where Prometheus will collect them. This can be required since jobs only exist for a short amount of time and as a result, Prometheus would fail to scrape these jobs most of the time. In addition, it would require all these jobs to implement a webserver in order for Prometheus to collect the metrics.

{{% alert title="Note" color="primary" %}}
The Pushgateway should only be used for for this specific use case. It simply acts as cache for short-lived jobs and by default does not even have any persistence. It is not intended to convert Prometheus into a push-based monitoring system
{{% /alert %}}
