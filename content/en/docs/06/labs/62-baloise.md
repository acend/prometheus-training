---
title: "6.2 Tasks: Troubleshoot Kubernetes Service Discovery"
weight: 2
sectionnumber: 6.2
onlyWhenNot: baloise
---

### Task {{% param sectionnumber %}}.1: Troubleshooting Kubernetes Service Discovery TODO: rewrite
We will now deploy an application with an error in the monitoring configration.

Deploy [Loki](https://grafana.com/oss/loki/) in the monitoring namespace.
    
Create a deployment `training_loki-deployment.yaml`.
    
{{< readfile file="/content/en/docs/06/labs/baloise_loki-deployment.yaml" code="true" lang="yaml" >}}

Create a Service `training_service-loki.yaml`.

{{< readfile file="/content/en/docs/06/labs/baloise_loki-service.yaml" code="true" lang="yaml" >}}

Create the Loki ServiceMonitor `training_servicemonitor-loki.yaml`.

{{< readfile file="/content/en/docs/06/labs/servicemonitor-loki.yaml" code="true" lang="yaml" >}}

* When you visit the [Prometheus user interface](http://{{% param replacePlaceholder.prometheus %}}/targets) you will notice that the Prometheus Server does not scrape metrics from Loki. Try to find out why.

{{% alert title="Troubleshooting: Prometheus is not scraping metrics" color="primary" %}}
The cause that Prometheus is not able to scrape metrics is usually one of the following:

* The configuration defined in the ServiceMonitor does not appear in the Prometheus scrape configuration.
  * Check if the label of your ServiceMonitor matches the label defined in the `serviceMonitorSelector` field of the Prometheus custom resource 
  * Check the Prometheus operator logs for errors (permission issues or invalid ServiceMonitors)
* The Endpoint appears in the Prometheus scrape config but not under targets.
  * The namespaceSelector in the ServiceMonitor does not match the namespace of your app
  * The label selector does not match the Service of your app
  * The port name does not match the Service of your app
* The Endpoint appears as a Prometheus target, but no data gets scraped.
  * The application does not provide metrics under the correct path and port
  * Networking issues
  * Authentication required, but not configured

{{% /alert %}}

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

The quickest way to do this is to follow the instructions in the info box above. So let's first find out which of the following statements apply to us:

* The configuration defined in the ServiceMonitor does not appear in the Prometheus scrape configuration.
  * Let's check if Prometheus reads the configuration defined in the ServiceMonitor resource. To do so, navigate to [Prometheus configuration](http://{{% param replacePlaceholder.prometheus %}}/config) and search if `loki` appears in the scrape_configuration. You should find a job with the name `serviceMonitor/loki/loki/0`, therefore this should not be the issue in this case.
* The Endpoint appears in the [Prometheus configuration](http://{{% param replacePlaceholder.prometheus %}}/config) but not under targets.
  * Let's check if the application is running:
    ```bash
    {{% param cliToolName %}} -n <team>-monitoring get pod -l app=loki
    ```
    The output should be similar to the following:
    ```bash
    NAME                    READY   STATUS    RESTARTS   AGE
    example-loki-7bb486b647-dj5r4          1/1     Running   0             112s
    ```
  * Lets check if the application is exposing metrics:
    ```bash
    PODNAME=$({{% param cliToolName %}} -n <team>-monitoring get pod -l app=loki -o name)
    {{% param cliToolName %}} -n <team>-monitoring exec $PODNAME -it -- wget -O - localhost:3100/metrics
    ...
    ```
  * The application exposes metrics and Prometheus generated the configuration according to the defined ServiceMonitor. Let's verify, if the ServiceMonitor matches the Service.
    ```bash
    {{% param cliToolName %}} -n <team>-monitoring get svc loki -o yaml
    ```

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      ...
      labels:
        app: loki
        argocd.argoproj.io/instance: ...
      name: loki
    spec:
      ...
      ports:
      - name: http
        ...
    ```
    We see that the Service has the port named `http` and the label `app: loki` set. Let's check the ServiceMonitor:
    ```bash
    {{% param cliToolName %}} -n <team>-monitoring get servicemonitor loki -o yaml
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
    We see that the ServiceMonitor expect the port named `http` and a label `prometheus-monitoring: "true"` set. So the culprit is the missing label. Let's set the label on the Service by updating the the service `training_service-loki.yaml`.

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: loki
      labels:
        app: loki
        prometheus-monitoring: "true"
    spec:
    ...
    ```

    Verify that the target gets scraped in the [Prometheus user interface](http://{{% param replacePlaceholder.prometheus %}}/targets).

{{% /details %}}

