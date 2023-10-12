---
title: "3.1 Tasks: Grafana intro"
weight: 2
sectionnumber: 3.1
onlyWhen: baloise
---

### Task {{% param sectionnumber %}}.1 Have a look at the default team dashboard

The Monitoring Stack provides a dashboard that shows you common aggregated metrics of your applications. You should see the dashboard as soon as you log in.

Have a look at the [provided dashboard](http://{{% param replacePlaceholder.grafana %}}/d/HoxOa1aTMk1/).

* Select either the cluster `caast01` or `caasp01` to get metrics from applications running on these clusters

### Task {{% param sectionnumber %}}.2 Have a look at the default team namespace dashboard

The Monitoring Stack provides a [dashboard](http://{{% param replacePlaceholder.grafana %}}/d/HoxOa1aTMk2/) that shows you metrics of your applications.

* Navigate to **Dashboards** (Icon with the four squares on the left navigation menu)
* The dashboard can be found under the name `Openshift Namespace Dashboard`

* Select metrics about your
  * `prometheus` container
  * running on the `caasi01` cluster
