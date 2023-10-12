---
title: "4.1 Tasks: Blackbox exporter"
weight: 2
sectionnumber: 4.1
onlyWhen: baloise
---

### Task {{% param sectionnumber %}}.1: Add a blackbox target

We will use the blackbox exporter to create a new probe which accepts a `2xx` return code as a valid http return code. This will return the `probe_success` metric from the blackbox exporter with the value `1`, if the http status code is `2xx`.

**Task description**:

* Create a probe in the monitoring directory which uses the HTTP prober and expects a `2xx` return code as a valid status code
* Define `https://bitbucket.balgroupit.com/status` as a single static target, which the blackbox should probe
* Use the following documentation as reference [06 - HTTP and TCP endpoint monitoring](https://confluence.baloisenet.com/atlassian/display/BALMATE/06+-+HTTP+and+TCP+endpoint+monitoring)


{{% details title="Hints" mode-switcher="normalexpertmode" %}}

To configure the blackbox exporter you have to add the following file `training_blackbox_target.yaml` to your monitoring directory, commit and push the changes:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: Probe
metadata:
  name: bitbucket-2xx
spec:
  module: http_2xx
  prober:
    url: blackbox:9115
  targets:
    staticConfig:
      static:
      - https://bitbucket.balgroupit.com/status
      labels:
        env: nonprod
```

You can simulate this by directly running a curl inside the prometheus pod on this URL. The `probe_success` metric should have the value `1`.

{{% alert title="Note" color="primary" %}}
As you will be executing some `oc` commands in the following labs, make sure you are logged in to your OpenShift Cluster.

You can copy the login Command from the OpenShift UI:

* Browse to <http://{{% param replacePlaceholder.openshift_console %}}>
* Click on your name in the top right
* `Copy login command`
* Replace `6443` with `443`

{{% /alert %}}

```bash
oc -n <team>-monitoring exec prometheus-prometheus-0 -c prometheus-proxy -- \
curl blackbox:9115/probe?target=https://bitbucket.balgroupit.com/status&module=http_2xx
...
# HELP probe_success Displays whether or not the probe was a success
# TYPE probe_success gauge
probe_success 1
...
```

{{% alert title="Warning" color="primary" %}}
As you can see in the [06 - HTTP and TCP endpoint monitoring](https://confluence.baloisenet.com/atlassian/display/BALMATE/06+-+HTTP+and+TCP+endpoint+monitoring) documentation, there are some modules pre-defined. You can directly use them as shown in the lab above.

If you have a custom use case, you can provide your own module as described in `Custom modules`.
{{% /alert %}}

{{% /details %}}

### Task {{% param sectionnumber %}}.2: Query blackbox metrics

Let's now create a query which selects all metrics belonging to the blackbox exporter target `https://bitbucket.balgroupit.com/status` and display them in the [Thanos Querier UI](http://{{% param replacePlaceholder.thanosquerier %}}).

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

We can select all metrics for the target with the following query:

```promql
{instance="https://bitbucket.balgroupit.com/status"}
```

or directly navigate to your [Thanos Querier](http://{{% param replacePlaceholder.thanosquerier %}}/graph?g0.expr=%7Binstance%3D"https%3A%2F%2Fbitbucket.balgroupit.com%2Fstatus"%7D&g0.tab=1)


{{% alert title="Warning" color="primary" %}}
In the list of metrics you will find one metric with the name `up`. In the case of a multi-target exporter such as the blackbox exporter this metric will always be up as long as Prometheus is able to successfully scrape the exporter even if the actual target (website, TCP service, etc.) is down. To monitor the state of the targets always use the `probe_success` metric.
{{% /alert %}}

{{% /details %}}


### Task {{% param sectionnumber %}}.3 (optional): Add a protocol label to your blackbox target

Add the new label `protocol` to every blackbox exporter target by updating the relabel config. The new label should contain the protocol (HTTP or HTTPS) extracted from the target URL.

{{% details title="Hints" mode-switcher="normalexpertmode" %}}

To configure the blackbox exporter you have to updates the following file `training_blackbox_target.yaml` in your monitoring directory:

```yaml
apiVersion: monitoring.coreos.com/v1
kind: Probe
metadata:
  name: bitbucket-2xx
spec:
  module: http_2xx
  prober:
    url: blackbox:9115
  targets:
    staticConfig:
      static:
      - https://bitbucket.balgroupit.com/status
      labels:
        env: nonprod
  metricRelabelings:
  - sourceLabels: [instance] #1
    targetLabel: protocol #2
    regex: '^(.+):.+' #3
    replacement: $1 #4

```

* **1**: Use the value from the label `instance`. This label contains all targets defined at `.spec.targets.staticConfig.static`
* **2**: We will call the new label `protocol`
* **3**: Capture the first part of your url until `:`. In our case `https` from `https://bitbucket.balgroupit.com/status`
* **4**: Replace `target_label` value with the regex match from `source_labels` value

{{% /details %}}
