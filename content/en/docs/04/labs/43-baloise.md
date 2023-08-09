---
title: "4.3 Tasks: Exporter as a sidecar"
weight: 3
sectionnumber: 4.3
onlyWhen: baloise
---

### Task {{% param sectionnumber %}}.1: Deploy a database and use a sidecar container to expose metric TODO: references etc etc
**Task description**:

As we've learned in [Lab 4 - Prometheus exporters](../../../04/) when applications do not expose metrics in the Prometheus format, there are a lot of exporters available to convert metrics into the correct format. In Kubernetes this is often done by deploying so called sidecar containers along with the actual application.

Use the following command to deploy a MariaDB database your monitoring or application namespace on CAAST.

Create the following deployment (`training_baloise_mariadb-deployment.yaml`)

{{< readfile file="/content/en/docs/04/labs/baloise_mariadb-init-deployment.yaml" code="true" lang="yaml" >}}

Create the following secret (`training_baloise_mariadb-secret.yaml`)

{{< readfile file="/content/en/docs/04/labs/baloise_mariadb-init-secret.yaml" code="true" lang="yaml" >}}

Create the following service (`training_baloise_mariadb-service.yaml`)

{{< readfile file="/content/en/docs/04/labs/baloise_mariadb-init-service.yaml" code="true" lang="yaml" >}}


This will create a [Secret](https://kubernetes.io/docs/concepts/configuration/secret/) (username password to access the database), a [Service](https://kubernetes.io/docs/concepts/services-networking/service/) and the [Deployment](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/).

* Deploy the [mariadb exporter](https://github.com/prometheus/mysqld_exporter) from <https://registry.hub.docker.com/r/prom/mysqld-exporter/> as a sidecar container
  * Alter the existing MariaDB deployment definition to contain the side car
* Create a ServiceMonitor to instruct Prometheus to scrape the sidecar container

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

First we need to alter the MariaDB deployment `training_baloise_mariadb-deployment.yaml` by adding the MariaDB exporter as a second container.

{{< readfile file="/content/en/docs/04/labs/baloise_mariadb-deployment.yaml" code="true" lang="yaml" >}}

Then extend the service `training_baloise_mariadb-service.yaml` by adding a second port for the MariaDB exporter.

{{< readfile file="/content/en/docs/04/labs/baloise_mariadb-service.yaml" code="true" lang="yaml" >}}

Then we also need to create a new ServiceMonitor `training_baloise_mariadb-servicemonitor.yaml`.

{{< readfile file="/content/en/docs/04/labs/servicemonitor-sidecar.yaml" code="true" lang="yaml" >}}

Verify that the target gets scraped in the [Prometheus user interface](http://{{% param replacePlaceholder.prometheus %}}/targets). Target name: `serviceMonitor/<team>-monitoring/mariadb/0` (It may take up to a minute for Prometheus to load the new configuration and scrape the metrics).

{{% /details %}}

### Task {{% param sectionnumber %}}.2: generic-chart MariaDB deployment (optional)

**Task description**:

* Deploy the [mariadb exporter](https://github.com/prometheus/mysqld_exporter) from [quay.io/prometheus/mysqld-exporter](https://quay.io/repository/prometheus/mysqld-exporter) as a sidecar container.
* Define all parameters using the [generic-chart](https://bitbucket.balgroupit.com/projects/CONTAINER/repos/generic-chart/browse).

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

Create an application on CAAST and deploy the following configuration.

**Chart.yaml**:

{{< readfile file="/content/en/docs/04/labs/baloise-generic-chart-Chart.yaml" code="true" lang="yaml" >}}

**values.yaml**:

{{< readfile file="/content/en/docs/04/labs/baloise-generic-chart-values.yaml" code="true" lang="yaml" >}}

**templates/secret.yaml**:

{{< readfile file="/content/en/docs/04/labs/baloise-generic-chart-secret.yaml" code="true" lang="yaml" >}}

Verify that the target gets scraped in the [Prometheus user interface](http://{{% param replacePlaceholder.prometheus %}}/targets). Target name: `application-metrics/mariadb/0` (it may take up to a minute for Prometheus to load the new configuration and scrape the metrics).

Make sure to remove the files `Chart.yaml`, `values.yaml` and `templates/secret.yaml` once finished.

{{% /details %}}

