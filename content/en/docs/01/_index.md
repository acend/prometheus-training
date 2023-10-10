---
title: "1. Setting up Prometheus"
weight: 1
sectionnumber: 1
---

{{% onlyWhenNot baloise %}}

## Working mode (GitOps)

During the labs you will deploy and update several resources on your Kubernetes environment. [ArgoCD](https://{{% param argoCdUrl %}}) will be your primary interface to interact with the cluster and will simplify the GitOps process for you.

{{% alert title="Note" color="primary" %}}
Argo CD is a part of the Argo Project and affiliated under the Cloud Native Computing Foundation (CNCF) . The project is just under three years old, completely open source, and primarily implemented in Go.

As the name suggests, Argo CD takes care of the continuous delivery aspect of CI/CD. The core of Argo CD consists of a Kubernetes controller, which continuously compares the live-state with the desired-state. The live-state is tapped from the Kubernetes API, and the desired-state is persisted in the form of manifests in YAML or JSON in a Git repository. Argo CD helps to point out deviations of the states, to display the deviations or to autonomously restore the desired state.
{{% /alert %}}

The configuration and deployments needed for you are already in a git repository. Navigate to your [Gitea](https://{{% param giteaUrl %}}) and look for a project called 'prometheus-training-lab-setup'. The repository consists of two [Helm](https://helm.sh/) Charts you will further use in this lab. In this first section we will no setup your Prometheus instance step by step.

## Installation

### Create ArgoCD Application

For ArgoCD to synchronize applications we need to create a Kubernetes Custom Resource called Application. The Application creates a logical connection between a Kubernetes namespace and a git repository to synchronize. Log into the ArgoCD instance and verify that there is no project called 'userX-prom-stack' (replace the userX with your user id).

To start we create the namespace your prometheus will be deployed in:

`kubectl create namespace userX-monitoring`

Create a file called `user-prom-stack-app.yaml` in your editor with the following content:

```
{{< highlight YAML >}}
---
kind: Application
metadata:
  name: user4-prom-stack
  namespace: argocd
spec:
  destination:
    namespace: userX-monitoring
    server: https://kubernetes.default.svc
  project: default
  source:
    repoURL: 'https://gitea.training.cluster.acend.ch/userX/prometheus-training-lab-setup'
    path: charts/user-monitoring/
    targetRevision: main
    helm:
      values: |
        user: userX
        # alertmanager
        alertmanager:
          enabled: false
        # grafana
        grafana:
          enabled: false
        # prometheus
        prometheus:
          enabled: true
        # pushgateway
        pushgateway:
          enabled: false
        # thanos-ruler
        ruler:
          enabled: false
        # thanos-query
        query:
          enabled: false
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
{{< / highlight >}}
```

Apply the manifest, creating your ArgoCD application:

```
kubectl -n argocd create -f user-prom-stack-app.yaml
```

Head over to the [ArgoCD UI](https://{{% param argoCdUrl %}}) and verify that the synchronization process of your application is synced and healthy. As soon as your application is healthy and synced (green status on top) you are good to go and have your prometheus instance ready.

When all has finished syncing, you can inspect your prometheus installation in your namespace:

```
kubectl -n userX-monitoring get prometheus prometheus -oyaml
```

### Setup and configure Prometheus

In the prometheus custom resource's spec block you can find various configuration options:

```yaml
spec:
  enableAdminAPI: true
  evaluationInterval: 30s
  externalLabels:
    monitoring: user4
  podMonitorNamespaceSelector:
    matchLabels:
      user: userX
  podMonitorSelector: {}
  portName: web
  probeNamespaceSelector:
    matchLabels:
      user: userX
  probeSelector: {}
  resources:
    requests:
      memory: 400Mi
  scrapeInterval: 10s
  serviceAccountName: prometheus-userX
  serviceMonitorNamespaceSelector:
    matchLabels:
      user: userX
  serviceMonitorSelector: {}
  ```
TODO: Describe the values that we set in the config: Things like scrape interval: (Prometheus is a pull-based monitoring system which means it will reach out to the configured targets and collect the metrics from them (instead of a push-based approach where the targets will push their metrics to the monitoring server). The option `scrape_interval` defines the interval at which Prometheus will collect the metrics for each target)

{{% alert title="Note" color="primary" %}}
We will learn more about other configuration options (`evaluation_interval`, TODO: other settings visible in the CR) later in this training.
{{% /alert %}}

TODO: Alternatively just switch the branch of your repo to XY.

### Check Prometheus

Is your prometheus running? Use your browser to navigate to https://{{% param prometheusUrl %}} . You should now see the Prometheus web UI.

{{% /onlyWhenNot %}}

## Targets

Since Prometheus is a pull-based monitoring system, the Prometheus server maintains a set of targets to scrape. This set can be configured using the `scrape_configs` option in the Prometheus configuration file. The `scrape_configs` consist of a list of jobs defining the targets as well as additional parameters (path, port, authentication, etc.) which are required to scrape these targets. As we will be using the Prometheus Operator on Kubernetes, we will never actually touch this configuration file by ourselves. Instead, we rely on the abstractions provided by the Operator, which we will look at closer in the next section.

There are two basic types of targets that we can add to our Prometheus server:

### Static targets

In this case, we define one or more targets statically. In order to make changes to the list, you need to change the configuration file. As the name implies, this way of defining targets is inflexible and not suited to monitor workloads inside of Kubernetes as these are highly dynamic.

{{% onlyWhenNot baloise %}}
TODO: We currently have no example of a static target in the labs for non-baloise. Do we keep it that way?
{{% /onlyWhenNot %}}
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

As discussed above, when configuring Prometheus to scrape metrics from containers deployed in a Kubernetes Cluster it doesn't really make sense to configure every single target manually. That would be far too static and wouldn't really work in a highly dynamic environment.

In fact, we tightly integrate Prometheus with Kubernetes and let Prometheus discover the targets, which need to be scraped, automatically via the Kubernetes API.

The tight integration between Prometheus and Kubernetes can be configured with the [Kubernetes Service Discovery Config](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config).

The way we instruct Prometheus to scrape metrics from an application running as a Pod is by creating a `ServiceMonitor`.

ServiceMonitors are Kubernetes custom resources, which look like this:

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
