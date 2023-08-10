---
title: "1.1 Tasks: Setup"
weight: 1
onlyWhen: baloise

sectionnumber: 1.1
---

## Executing oc commands

{{% alert title="Note" color="primary" %}}
Execute the following `oc` commands using one of those options:

* OpenShift Webconsole Terminal <http://{{% param replacePlaceholder.openshift_console %}}> right top menu `>_`
* On your local machine using the `oc` tool, make sure to login on your OpenShift Cluster first.

{{% /alert %}}


### Task {{% param sectionnumber %}}.1: Identify your monitoring repository

Before we get started, take the time to familiarize yourself with the config repository of your team - it should already be available as described in [Deploying the Baloise Monitoring Stack](https://confluence.baloisenet.com/atlassian/display/BALMATE/01+-+Deploying+the+Baloise+Monitoring+Stack).

The working directory for this training is the folder in your [team's config repository](http://{{% param replacePlaceholder.git %}}) with the `-monitoring` suffix. If necessary, create the directory `<team>-monitoring`.

{{% alert title="Note" color="warning" %}}
Please name all files created in this training with the filename prefix `training_`. This naming pattern will help in cleaning up all related files after training completion.
{{% /alert %}}

### Task {{% param sectionnumber %}}.2: Deploy example application

{{% alert title="Note" color="primary" %}}
We will deploy an application for demonstration purposes in our monitoring namespace. This should never be done for production use cases. If you are familiar with deploying on OpenShift, you can complete this lab by deploying the application on our test cluster.
{{% /alert %}}

Create the following file `training_python-deployment.yaml` in your monitoring directory.

{{< readfile file="/content/en/docs/01/labs/baloise_python-deployment.yaml" code="true" lang="yaml" >}}

Use the following command to verify that the pod of the deployment `example-web-python` is ready and running (use CTRL+C to exit the command).

```bash
team=<team>
{{% param cliToolName %}} -n $team-monitoring get pod -w -l app=example-web-python
```

We also need to create a Service for the new application. Create a file with the name `training_python-service.yaml` with the following content:

{{< readfile file="/content/en/docs/01/labs/baloise_python-service.yaml" code="true" lang="yaml" >}}

This created a so-called [Kubernetes Service](https://kubernetes.io/docs/concepts/services-networking/service/)

```bash
team=<team>
{{% param cliToolName %}} -n $team-monitoring get svc -l app=example-web-python
```

Which gives you an output similar to this:

```bash
NAME                 TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
example-web-python                  ClusterIP   172.24.195.25    <none>        5000/TCP                     24s
```

Our example application can now be reached on port `5000`.

We can now make the application directly available on our machine using [port-forward](https://kubernetes.io/docs/tasks/access-application-cluster/port-forward-access-application-cluster/)

```bash
team=<team>
{{% param cliToolName %}} -n $team-monitoring port-forward svc/example-web-python 5000
```

Use `curl` and verify the successful deployment of our example application in a separate terminal:

```bash
curl localhost:5000/metrics
```

Should result in something like:

```promql
# HELP python_gc_objects_collected_total Objects collected during gc
# TYPE python_gc_objects_collected_total counter
python_gc_objects_collected_total{generation="0"} 541.0
python_gc_objects_collected_total{generation="1"} 344.0
python_gc_objects_collected_total{generation="2"} 15.0
...
```

Since our newly deployed application now exposes metrics, the next thing we need to do, is to tell our Prometheus server to scrape metrics from the Kubernetes deployment. In a highly dynamic environment like Kubernetes this is done with so called Service Discovery.

### Task {{% param sectionnumber %}}.3: Create a ServiceMonitor

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

