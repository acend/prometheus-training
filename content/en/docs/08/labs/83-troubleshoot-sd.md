---
title: "8.3 Tasks: Troubleshoot Kubernetes Service Discovery"
weight: 83
sectionnumber: 8.3
onlyWhenNot: baloise
---

### Task {{% param sectionnumber %}}.1: Troubleshooting Kubernetes Service Discovery

We will now deploy an application with an error in the monitoring configration.

* Deploy [Loki](https://grafana.com/oss/loki/) in your namespace by adding the following files to your git repo

```yaml
## Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: loki
  name: loki
spec:
  replicas: 1
  selector:
    matchLabels:
      app: loki
  template:
    metadata:
      labels:
        app: loki
    spec:
      containers:
      - image: mirror.gcr.io/grafana/loki:latest
        name: loki
```

```yaml
## Service
apiVersion: v1
kind: Service
metadata:
  labels:
    app: loki
  name: loki
spec:
  ports:
    - name: http
      port: 3100
      protocol: TCP
      targetPort: 3100
  selector:
    app: loki
  type: NodePort
```

```yaml
## ServiceMonitor
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  labels:
    app.kubernetes.io/name: loki
  name: loki
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

* When you visit the [Prometheus user interface](http://{{% param replacePlaceholder.k8sPrometheus %}}/targets) you will notice, that the Prometheus Server does not scrape metrics from Loki. Try to find out why.

{{% alert title="Troubleshooting: Prometheus is not scrapping metrics" color="primary" %}}
The cause that Prometheus is not able to scrape metrics is usually one of the following.

* The configuration defined in the ServiceMonitor does not appear in the Prometheus scrape configuration
  * Check if the label of your ServiceMonitor matches the label defined in the `serviceMonitorSelector` field of the Prometheus custom resource
  * Check the Prometheus operator logs for errors (Permission issues or invalid ServiceMonitors)
* The Endpoint appears in the Prometheus scrape config but not under targets.
  * The namespaceSelector in the ServiceMonitor does not match the namespace of your app
  * The label selector does not match the Service of your app
  * The port name does not match the Service of your app
* The Endpoint appears as a Prometheus target, but no data gets scraped
  * The application does not provide metrics under the correct path and port
  * Networking issues
  * Authentication required, but not configured

{{% /alert %}}

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

The quickest way to do this is to follow the instructions in the info box above. So let's first find out which of the following statements apply to us

* The configuration defined in the ServiceMonitor does not appear in the Prometheus scrape configuration
  * Let's check if Prometheus reads the configuration defined in the ServiceMonitor resource. To do so navigate to [Prometheus configuration](http://{{% param replacePlaceholder.k8sPrometheus %}}/config) and search if `loki` appears in the scrape_configuration. You should find a job with the name `serviceMonitor/loki/loki/0`, therefore this should not be the issue in this case.
* The Endpoint appears in the [Prometheus configuration](http://{{% param replacePlaceholder.k8sPrometheus %}}/config) but not under targets.
  * The namespaceSelector in the ServiceMonitor does not match the namespace of your app
  * The label selector does not match the Service of your app
  * The port name does not match the Service of your app
* The Endpoint appears as a Prometheus target, but no data gets scraped
  * The application does not provide metrics under the correct path and port
  * Networking issues
  * Authentication required, but not configured

{{% /alert %}}

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

The quickest way to do this is to follow the instructions in the info box above. So let's first find out which of the following statements apply to us

* The configuration defined in the ServiceMonitor does not appear in the Prometheus scrape configuration
  * Let's check if Prometheus reads the configuration defined in the ServiceMonitor resource. To do so navigate to [Prometheus configuration](http://{{% param replacePlaceholder.k8sPrometheus %}}/config) and search if `loki` appears in the scrape_configuration. You should find a job with the name `serviceMonitor/loki/loki/0`, therefore this should not be the issue in this case.
* The Endpoint appears in the [Prometheus configuration](http://{{% param replacePlaceholder.k8sPrometheus %}}/config) but not under targets.
  * Lets check if the application is running
    ```bash
    {{% param cliToolName %}} get pod
    ```
    You should see a loki Pod in the `Running` state:
    ```bash
    NAME                    READY   STATUS    RESTARTS   AGE
    loki-5846d87f4c-tthsr   1/1     Running   0          34m
    ```
  * Lets check if the application is exposing metrics
    ```bash
    PODNAME=$({{% param cliToolName %}} get pod -l app=loki -o name)
    {{% param cliToolName %}} exec $PODNAME -it -- wget -O - localhost:3100/metrics
    ...
    ```
  * The application exposes metrics and Prometheus generated the configuration according to the defined servicemonitor. Let's verify, if the ServiceMonitor matches the Service.
    ```bash
    {{% param cliToolName %}} get svc loki -o yaml
    ```

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      ...
      labels:
        app: loki
      name: loki
      namespace: loki
    spec:
      ...
      ports:
      - name: http
        ...
    ```
    We see that the Service has the port named `http` and the label `app: loki` set. Let's check the ServiceMonitor
    ```bash
    {{% param cliToolName %}} get servicemonitor loki -o yaml
    ```

    ```yaml
    apiVersion: monitoring.coreos.com/v1
    kind: ServiceMonitor
    ...
    spec:
      endpoints:
      - interval: 30s
        ...
        port: http
        ...
      selector:
        matchLabels:
          prometheus-monitoring: "true"
    ```
    We see that the ServiceMonitor expect the port named `http` and a label `prometheus-monitoring: "true"` set. So the culprit is the missing label. Let's adjust the service manifest, commit and push.
    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      labels:
        app: loki
        prometheus-monitoring=true
      name: loki
    spec:
      ports:
        - name: http
          port: 3100
          protocol: TCP
          targetPort: 3100
      selector:
        app: loki
      type: NodePort
    ```

    Verify that the target now gets scraped in the [Prometheus user interface](http://{{% param replacePlaceholder.k8sPrometheus %}}/targets).

{{% /details %}}

