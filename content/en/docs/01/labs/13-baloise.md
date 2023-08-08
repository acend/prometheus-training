---
title: "1.3 Tasks: Application Monitoring"
weight: 1
sectionnumber: 1.3
onlyWhen: baloise
---

### Task {{% param sectionnumber %}}.1: Create a ServiceMonitor

**Task description**:

Create a ServiceMonitor for the example application.

* Create a ServiceMonitor, which will configure Prometheus to scrape metrics from the example-web-python application every 30 seconds.

For this to work, you need to ensure:

* The example-web-python Service is labeled correctly and matches the labels you've defined in your ServiceMonitor.
* The port name in your ServiceMonitor configuration matches the port name in the Service definition.
  * hint: check with `{{% param cliToolName %}} -n <team>-monitoring get service example-web-python -o yaml`
* Verify the target in the Prometheus user interface.

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

Create the following ServiceMonitor (`training_python-servicemonitor.yaml`):

{{< readfile file="/content/en/docs/01/labs/baloise_python-servicemonitor.yaml" code="true" lang="yaml" >}}

Verify that the target gets scraped in the [Prometheus user interface](http://{{% param replacePlaceholder.prometheus %}}) (either on CAASI or CAAST, depending where you deployed the application). Target name: `serviceMonitor/<team>-monitoring/example-web-python-monitor/0` (it may take up to a minute for Prometheus to load the new
configuration and scrape the metrics).

{{% /details %}}


### Task {{% param sectionnumber %}}.2: Deploy a database and use a sidecar container to expose metrics

**Task description**:

As we've learned in [Lab 4 - Prometheus exporters](../../../04/) when applications do not expose metrics in the Prometheus format, there are a lot of exporters available to convert metrics into the correct format. In Kubernetes this is often done by deploying so called sidecar containers along with the actual application.

Use the following command to deploy a MariaDB database your monitoring or application namespace on CAAST.

Create the following deployment (`training_baloise_mariadb-deployment.yaml`)

{{< readfile file="/content/en/docs/01/labs/baloise_mariadb-init-deployment.yaml" code="true" lang="yaml" >}}

Create the following secret (`training_baloise_mariadb-secret.yaml`)

{{< readfile file="/content/en/docs/01/labs/baloise_mariadb-init-secret.yaml" code="true" lang="yaml" >}}

Create the following service (`training_baloise_mariadb-service.yaml`)

{{< readfile file="/content/en/docs/01/labs/baloise_mariadb-init-service.yaml" code="true" lang="yaml" >}}


This will create a [Secret](https://kubernetes.io/docs/concepts/configuration/secret/) (username password to access the database), a [Service](https://kubernetes.io/docs/concepts/services-networking/service/) and the [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

* Deploy the [mariadb exporter](https://github.com/prometheus/mysqld_exporter) from <https://registry.hub.docker.com/r/prom/mysqld-exporter/> as a sidecar container
  * Alter the existing MariaDB deployment definition to contain the side car
* Create a ServiceMonitor to instruct Prometheus to scrape the sidecar container

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

First we need to alter the MariaDB deployment `training_baloise_mariadb-deployment.yaml` by adding the MariaDB exporter as a second container.

{{< readfile file="/content/en/docs/01/labs/baloise_mariadb-deployment.yaml" code="true" lang="yaml" >}}

Then extend the service `training_baloise_mariadb-service.yaml` by adding a second port for the MariaDB exporter.

{{< readfile file="/content/en/docs/01/labs/baloise_mariadb-service.yaml" code="true" lang="yaml" >}}

Then we also need to create a new ServiceMonitor `training_baloise_mariadb-servicemonitor.yaml`.

{{< readfile file="/content/en/docs/01/labs/servicemonitor-sidecar.yaml" code="true" lang="yaml" >}}

Verify that the target gets scraped in the [Prometheus user interface](http://{{% param replacePlaceholder.prometheus %}}/targets). Target name: `serviceMonitor/<team>-monitoring/mariadb/0` (It may take up to a minute for Prometheus to load the new configuration and scrape the metrics).

{{% /details %}}

### Task {{% param sectionnumber %}}.3: Troubleshooting Kubernetes Service Discovery

We will now deploy an application with an error in the monitoring configration.

Deploy [Loki](https://grafana.com/oss/loki/) in the monitoring namespace.

Create a deployment `training_loki-deployment.yaml`.

{{< readfile file="/content/en/docs/01/labs/baloise_loki-deployment.yaml" code="true" lang="yaml" >}}

Create a Service `training_service-loki.yaml`.

{{< readfile file="/content/en/docs/01/labs/baloise_loki-service.yaml" code="true" lang="yaml" >}}

Create the Loki ServiceMonitor `training_servicemonitor-loki.yaml`.

{{< readfile file="/content/en/docs/01/labs/servicemonitor-loki.yaml" code="true" lang="yaml" >}}

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

### Task {{% param sectionnumber %}}.4: Cleanup your monitoring workspace

Make sure to remove all files with the `training_` prefix in your monitoring directory.


### Task {{% param sectionnumber %}}.5: generic-chart MariaDB deployment (optional)


**Task description**:

* Deploy the [mariadb exporter](https://github.com/prometheus/mysqld_exporter) from [quay.io/prometheus/mysqld-exporter](https://quay.io/repository/prometheus/mysqld-exporter) as a sidecar container.
* Define all parameters using the [generic-chart](https://bitbucket.balgroupit.com/projects/CONTAINER/repos/generic-chart/browse).

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

Create an application on CAAST and deploy the following configuration.

**Chart.yaml**:

{{< readfile file="/content/en/docs/01/labs/baloise-generic-chart-Chart.yaml" code="true" lang="yaml" >}}

**values.yaml**:

{{< readfile file="/content/en/docs/01/labs/baloise-generic-chart-values.yaml" code="true" lang="yaml" >}}

**templates/secret.yaml**:

{{< readfile file="/content/en/docs/01/labs/baloise-generic-chart-secret.yaml" code="true" lang="yaml" >}}

Verify that the target gets scraped in the [Prometheus user interface](http://{{% param replacePlaceholder.prometheus %}}/targets). Target name: `application-metrics/mariadb/0` (it may take up to a minute for Prometheus to load the new configuration and scrape the metrics).

Make sure to remove the files `Chart.yaml`, `values.yaml` and `templates/secret.yaml` once finished.

{{% /details %}}
