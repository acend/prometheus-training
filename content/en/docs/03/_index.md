---
title: "3. Visualization"
weight: 1
sectionnumber: 1
---

Our goal with this lab is to give you a brief overview how to visualize your Prometheus time series produced in the previous labs.
For a more detailed tutorial, please refer to the [Grafana tutorials](https://grafana.com/tutorials/).

{{% onlyWhenNot baloise %}}
Grafana is already installed and running on your machine. Login to your Grafana instance on <http://{{% param replacePlaceholder.grafana %}}/>. Use your personal credentials to access the Grafana login page. Then use the Grafana default credentials (username: `admin`, password: `admin`) to log in to Grafana.

{{% alert title="Note" color="warning" %}}
Wou will have to **authenticate twice** to access Grafana. First use your personal credentials (same used in the earlier labs to log in to Prometheus or Alertmanager) then on the Grafana login page use the Grafana default credentials (`admin`/`admin`)

{{% /alert %}}

{{% /onlyWhenNot %}}
{{% onlyWhen baloise %}}
Grafana is already provided in your Stack. Login to your Grafana instance on <http://{{% param replacePlaceholder.grafana %}}/>. Use your personal credentials to log in to Grafana.
{{% /onlyWhen %}}

## Useful links and guides

* [Prometheus data source](https://grafana.com/docs/grafana/latest/datasources/prometheus/)
* [Grafana dashboards](https://grafana.com/docs/grafana/latest/best-practices/best-practices-for-creating-dashboards/)
* [Grafana provisioning](https://grafana.com/docs/grafana/latest/administration/provisioning/)
