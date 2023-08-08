---
title: "5. Prometheus in Kubernetes"
weight: 5
sectionnumber: 1
---

## Prometheus in Kubernetes

{{% onlyWhenNot baloise %}}

We will use [minikube](https://minikube.sigs.k8s.io/docs/start/) to start a minimal Kubernetes environment. If you are a novice in Kubernetes, you may want to use the [{{% param cliToolName %}} cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)

{{% alert title="Minikube" color="primary" %}}
Minikube is already started and configured. When you restart your virtual machine, you might need to start it manually.

```bash
minikube start \
--kubernetes-version=v1.23.1 \
--memory=6g \
--cpus=4 \
--bootstrapper=kubeadm \
--extra-config=kubelet.authentication-token-webhook=true \
--extra-config=kubelet.authorization-mode=Webhook \
--extra-config=scheduler.address=0.0.0.0 \
--extra-config=controller-manager.address=0.0.0.0
```

{{% /alert %}}

Check if you can connect to the API and verify the minikube master node is in `ready` state.

```bash
{{% param cliToolName %}} get nodes
```

```bash
NAME       STATUS   ROLES                  AGE    VERSION
minikube   Ready    control-plane,master   2m2s   v1.23.1
```

Deploy the Prometheus operator stack, consisting of:

* Prometheus Operator [deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)
* Prometheus Operator [ClusterRole and ClusterRoleBinding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#rolebinding-and-clusterrolebinding)
* CustomResourceDefinitions
  * [Prometheus](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#prometheus): Manage the Prometheus instances
  * [Alertmanager](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#alertmanager): Manage the Alertmanager instances
  * [ServiceMonitor](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#servicemonitor): Generate Kubernetes service discovery scrape configuration based on Kubernetes [service](https://kubernetes.io/docs/concepts/services-networking/service/) definitions
  * [PrometheusRule](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#prometheusrule): Manage the Prometheus rules of your Prometheus
  * [AlertmanagerConfig](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#alertmanagerconfig): Add additional receivers and routes to your existing Alertmanager configuration
  * [PodMonitor](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#podmonitor): Generate Kubernetes service discovery scrape configuration based on Kubernetes pod definitions
  * [Probe](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#probe): Custom resource to manage Prometheus blackbox exporter targets
  * [ThanosRuler](https://github.com/prometheus-operator/prometheus-operator/blob/master/Documentation/api.md#thanosruler): Manage [Thanos rulers](https://github.com/thanos-io/thanos/blob/main/docs/components/rule.md)

```bash
git clone https://github.com/prometheus-operator/kube-prometheus.git ~/work/kube-prometheus
cd ~/work/kube-prometheus
git checkout release-0.10
{{% param cliToolName %}} create -f manifests/setup
```

The manifest will deploy a complete monitoring stack consisting of:

* Two Prometheis - [What is the plural of Prometheus? ;)](https://prometheus.io/docs/introduction/faq/#what-is-the-plural-of-prometheus)
* Alertmanager cluster
* Grafana
* kube-state metrics exporter
* node_exporter
* blackbox exporter
* A set of default PrometheusRules
* A set of default dashboards

```bash
{{% param cliToolName %}} create -f manifests/
```

By default, Prometheus is only allowed to monitor the `default`, `monitoring` and `kube-system` namespaces. Therefore we will add the necessary ClusterRoleBinding to grant Prometheus access to cluster-wide resources. Also we will create the needed ingress definitions for you, which will expose the monitoring components.

```bash
{{% param cliToolName %}} -n monitoring apply -f \
https://raw.githubusercontent.com/puzzle/prometheus-training/main/content/en/docs/07/resources.yaml
```

Wait until all pods are running

```bash
watch {{% param cliToolName %}} -n monitoring get pods
```

Check if you can access the Prometheus web interface at <http://{{% param replacePlaceholder.k8sPrometheus %}}>

Check access to Alertmanager at <http://{{% param replacePlaceholder.k8sAlertmanager %}}>

Check access to Grafana at <http://{{% param replacePlaceholder.k8sGrafana %}}>
{{% alert title="Note" color="primary" %}}
Use the default Grafana logging credentials and change the password

* username: admin
* password: admin

{{% /alert %}}

{{% /onlyWhenNot %}}

{{% onlyWhen baloise %}}

Have a look at the [Baloise Monitoring Stack](https://confluence.baloisenet.com/display/BALMATE/Application+Monitoring) and take a look at the different components and how they work together.

You will notice that each Team Monitoring Stack has components on all clusters it is included in. The metrics scraped by the Team Monitoring Stack are not shared by default. However, you can [provide your Prometheus time series to other monitoring stacks](https://confluence.baloisenet.com/atlassian/display/BALMATE/01+-+Deploying+the+Baloise+Monitoring+Stack).

{{% /onlyWhen %}}
