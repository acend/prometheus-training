---
title: "1. Setting up Prometheus"
weight: 1
sectionnumber: 1
---

{{% onlyWhen baloise %}}
In this first section we are going to eplore the already set up Prometheus stack. Each trainee will have their own stack to work with.
{{% /onlyWhen %}}

{{% onlyWhenNot baloise %}}
In this first section we are going to set up our first parts of the Prometheus stack. Each trainee will have their own stack installed and configured.


## Working mode (GitOps)

During the labs you will deploy and update several resources on your Kubernetes environment. [ArgoCD](https://{{% param argoCdUrl %}}) will be your primary interface to interact with the cluster and will simplify the GitOps process for you.

{{% alert title="Note" color="info" %}}
Argo CD is a part of the Argo Project and affiliated under the Cloud Native Computing Foundation (CNCF) . The project is just under three years old, completely open source, and primarily implemented in Go.

As the name suggests, Argo CD takes care of the continuous delivery aspect of CI/CD. The core of Argo CD consists of a Kubernetes controller, which continuously compares the live-state with the desired-state. The live-state is tapped from the Kubernetes API, and the desired-state is persisted in the form of manifests in YAML or JSON in a Git repository. Argo CD helps to point out deviations of the states, to display the deviations or to autonomously restore the desired state.
{{% /alert %}}

The configuration and deployments needed for you are already in a git repository. Navigate to your [Gitea](https://{{% param giteaUrl %}}) and look for a project called 'prometheus-training-lab-setup'. The repository consists of two [Helm](https://helm.sh/) Charts you will further use in this lab. In this first section we will no setup your Prometheus instance step by step.

We're going to use two main Namespaces for the lab

* `<user>` - where the user workload (Demo application, Webshell) is deployed
* `<user>-monitoring` - where we deploy our monitoring stack to

{{% /onlyWhenNot %}}


## How do metrics end up in Prometheus?

Since Prometheus is a pull-based monitoring system, the Prometheus server maintains a set of **targets** to scrape. This set can be configured using the `scrape_configs` option in the Prometheus configuration file. The `scrape_configs` consist of a list of jobs defining the targets as well as additional parameters (path, port, authentication, etc.) which are required to scrape these targets. As we will be using the Prometheus Operator on Kubernetes, we will never actually touch this configuration file by ourselves. Instead, we rely on the abstractions provided by the Operator, which we will look at closer in the next section.

There are two basic types of targets that we can add to our Prometheus server:

### Static targets

In this case, we define one or more targets statically. In order to make changes to the list, you need to change the configuration file. As the name implies, this way of defining targets is inflexible and not suited to monitor workloads inside of Kubernetes as these are highly dynamic.

{{% onlyWhen baloise %}}
We will use this type of configuration in the task [2.1](/docs/02/labs/21-baloise/).
{{% /onlyWhen %}}

### Dynamic configuration

Besides the static target configuration, Prometheus provides many ways to dynamically add/remove targets. There are builtin service discovery mechanisms for cloud providers such as AWS, GCP, Hetzner, and many more. In addition, there are more versatile discovery mechanisms available which allow you to implement Prometheus in your environment (e.g. DNS service discovery or file service discovery). Most importantly, the Prometheus Operator makes it very easy to let Prometheus discover targets dynamically using the Kubernetes API.

## Prometheus Operator

The Prometheus Operator is the preferred way of running Prometheus inside of a Kubernetes Cluster. In the following labs you will get to know its [CustomResources](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/) in more detail, which are the following:

* [Prometheus](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#prometheus): Manage the Prometheus instances
* [Alertmanager](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#alertmanager): Manage the Alertmanager instances
* [ServiceMonitor](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#servicemonitor): Generate Kubernetes service discovery scrape configuration based on Kubernetes [service](https://kubernetes.io/docs/concepts/services-networking/service/) definitions
* [PrometheusRule](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#prometheusrule): Manage the Prometheus rules of your Prometheus
* [AlertmanagerConfig](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#alertmanagerconfig): Add additional receivers and routes to your existing Alertmanager configuration
* [PodMonitor](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#podmonitor): Generate Kubernetes service discovery scrape configuration based on Kubernetes pod definitions
* [Probe](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#probe): Manage Prometheus blackbox exporter targets
* [ThanosRuler](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#thanosruler): Manage [Thanos rulers](https://github.com/thanos-io/thanos/blob/main/docs/components/rule.md)

### Service Discovery

When configuring Prometheus to scrape metrics from containers deployed in a Kubernetes Cluster it doesn't really make sense to configure every single target (Pod) manually. That would be far too static and wouldn't really work in a highly dynamic environment. A container platform is too dynamic. Pods can be scaled, the names are random and so on.

In fact, we tightly integrate Prometheus with Kubernetes and let Prometheus discover the targets, which need to be scraped, automatically via the Kubernetes API.

The tight integration between Prometheus and Kubernetes can be configured with the [Kubernetes Service Discovery Config](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config).

The way we instruct Prometheus to scrape metrics from an application running as a Pod is by creating a `ServiceMonitor`.

ServiceMonitors are Kubernetes custom resources, which look like this:

```yaml
# just an example
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
      prometheus-monitoring: 'true'
```

#### How does it work

The Prometheus Operator watches namespaces for `ServiceMonitor` custom resources. It then updates the Service Discovery configuration of the Prometheus server(s) accordingly.

The selector part in the `ServiceMonitor` defines which Kubernetes Services will be scraped. Here we are selecting the correct service by defining a selector on the label `prometheus-monitoring: 'true'`.

```yaml
# servicemonitor.yaml
...
  selector:
    matchLabels:
      prometheus-monitoring: 'true'
...
```

The corresponding `Service` needs to have this label set:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: example-web-python
  labels:
    prometheus-monitoring: 'true'
...
```

The Prometheus Operator then determines all `Endpoints`(which are basically the IPs of the Pods) that belong to this `Service` using the Kubernetes API. The `Endpoints` are then dynamically added as targets to the Prometheus server(s).

The `spec` section in the ServiceMonitor resource allows further configuration on how to scrape the targets.
In our case Prometheus will scrape:

* Every 30 seconds
* Look for a port with the name `http` (this must match the name in the `Service` resource)
* Scrape metrics from the path `/metrics` using `http`

### Best practices

Use the common k8s labels <https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels/>

If possible, reduce the number of different `ServiceMonitors` for an application and thereby reduce the overall complexity.

* Use the same `matchLabels` on different `Services` for your application (e.g. Frontend Service, Backend Service, Database Service)
* Also make sure the ports of different `Services` have the same name
* Expose your metrics under the same path

{{% onlyWhen baloise %}}

## Add your application as monitoring target at Baloise

Have a look at the [Add Monitoring Targets outside of OpenShift](https://confluence.baloisenet.com/atlassian/display/BALMATE/02+-+Add+your+application+as+monitoring+target#id-02Addyourapplicationasmonitoringtarget-AddMonitoringTargetsoutsideofOpenShift) documentation. There are two ways to add machines outside of OpenShift to your monitoring stack.

* Using `File Service Discovery` you have the following options (lab [2.1](/docs/02/labs/21-baloise/))
  * Add targets using TLS and using the default credentials provided
  * Add targets without TLS and authentication
* You can use the approach with `ServiceMonitors`, which provides more flexibility for cases like
  * custom targets with non standard basic authentication
  * custom targets with non TLS and non standard basic authentication
  * provide ca to verify custom certificate on the exporter side
  * define a non default `scrape_interval`

{{% /onlyWhen %}}
