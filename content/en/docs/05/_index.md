---
title: "5. Prometheus in Kubernetes"
weight: 1
sectionnumber: 1
---

{{% onlyWhen baloise %}}

## Baloise Monitoring Stack

Have a look at the [Baloise Monitoring Stack](https://confluence.baloisenet.com/display/BALMATE/Application+Monitoring) and take a look at the different components and how they work together.

You will notice that each Team Monitoring Stack has components on all clusters it is included in. The metrics scraped by the Team Monitoring Stack are not shared by default. However, you can [provide your Prometheus time series to other monitoring stacks](https://confluence.baloisenet.com/atlassian/display/BALMATE/01+-+Deploying+the+Baloise+Monitoring+Stack).

{{% /onlyWhen %}}

## kube-prometheus

{{% onlyWhenNot baloise %}}

The [kube-prometheus](https://github.com/prometheus-operator/kube-prometheus) stack already provides an extensive Prometheus setup and contains a set of default alerts and dashboards from [Prometheus Monitoring Mixin for Kubernetes](https://github.com/kubernetes-monitoring/kubernetes-mixin). The following targets will be available.

{{% /onlyWhenNot %}}

{{% onlyWhen baloise %}}

The [Baloise Monitoring Stack](https://confluence.baloisenet.com/display/BALMATE/Application+Monitoring) stack already provides an extensive Prometheus setup and contains a set of default alerts and dashboards. The following targets will be available.

{{% /onlyWhen %}}

**kube-state-metrics:** Exposes metadata information about Kubernetes resources. Used, for example, to check if resources have the expected state (deployment rollouts, pods CrashLooping) or if jobs fail.

```promql
# Example metrics
kube_deployment_created
kube_deployment_spec_replicas
kube_daemonset_status_number_misscheduled
...
```

**kubelet/cAdvisor:** [Advisor](https://github.com/google/cadvisor) exposes usage and performance metrics about running container. Commonly used to observe memory usage or [CPU throttling](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/).

```promql
# Example metrics
container_cpu_cfs_throttled_periods_total
container_memory_working_set_bytes
container_fs_inodes_free
...
```

**kubelet:** Exposes general kubelet related metrics. Used to observe if the kubelet and the container engine is healthy.

```promql
# Example metrics
kubelet_runtime_operations_duration_seconds_bucket
kubelet_runtime_operations_total
...
```

{{% onlyWhenNot baloise %}}
**apiserver:** Metrics from the Kubernetes API server. Commonly used to catch errors on resources or problems with latency.

```promql
# Example metrics
apiserver_request_duration_seconds_bucket
apiserver_request_total{code="200",...}
...
```
{{% /onlyWhenNot %}}

**kubelet/probes:** Expose metrics about [Kubernetes liveness, readiness and startup probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/) Normally you would not alert on Kubernetes probe metrics, but on container restarts exposed by `kube-state-metrics`.

```promql
# Example metrics
prober_probe_total{probe_type="Liveness", result="successful",...}
prober_probe_total{probe_type="Startup", result="successful",...}
...
```

**blackbox-exporter:** Exposes default metrics from blackbox-exporter. Can be customized using the [Probe](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#probe) custom resource.

```promql
# Example metrics
probe_http_status_code
probe_http_duration_seconds
...
```

{{% onlyWhenNot baloise %}}
**node-exporter:** Exposes the hardware and OS metrics from the nodes running Kubernetes.

```promql
# Example metrics
node_filesystem_avail_bytes
node_disk_io_now
...
```
{{% /onlyWhenNot %}}

**alertmanager-main/grafana/prometheus-k8s/prometheus-operator/prometheus-adapter:** Exposes all monitoring stack component metrics.

```promql
# Example metrics
alertmanager_alerts_received_total
alertmanager_config_last_reload_successful
...
grafana_build_info
grafana_datasource_request_total
...
prometheus_config_last_reload_successful
prometheus_rule_evaluation_duration_seconds
...
prometheus_operator_reconcile_operations_total
prometheus_operator_managed_resources
...
```

**pushgateway:** Exposes metrics pushed to your pushgateway.

```promql
# Example metrics
pushgateway_build_info
pushgateway_http_requests_total
...
```

{{% onlyWhen baloise %}}

**Vault:** Exposes metrics from your vaults instance.

```promql
# Example metrics
vault_autopilot_healthy
vault_core_unsealed
...
```

**Jenkins:** Exposes metrics from your Jenkins instance.

```promql
# Example metrics
jenkins_builds_duration_milliseconds_summary_count
jenkins_executors_available
...
```

**Argo CD:** Exposes metrics from your Argo CD instance.


```promql
# Example metrics
argocd_app_info{sync_status="Synced", health_status="Healthy", ...}
argocd_app_sync_total
...
```
{{% /onlyWhen %}}
