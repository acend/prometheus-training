---
title: "7. Alerting with Alertmanager"
weight: 1
sectionnumber: 1
---

## Installation

### Setup

{{% onlyWhenNot baloise %}}
You will install Alertmanager in the following labs. We will have a look at the default configuration in the next chapter.
{{% /onlyWhenNot %}}

{{% onlyWhen baloise %}}
At Baloise the [Alertmanger](http://{{% param replacePlaceholder.alertmanager %}}) is part of the managed monitoring stack and does not need to be installed. We will have a look at the default configuration in the next chapter.
{{% /onlyWhen %}}

## Configuration in Alertmanager

Alertmanager's configuration is done using a YAML config file. There are two main sections for configuring how Alertmanager is dispatching alerts: receivers and routing.

### Receivers

With a [receiver](https://prometheus.io/docs/alerting/latest/configuration/#receiver), one or more notifications can be defined. There are different types of notifications types, e.g. mail, webhook, or one of the message platforms like Slack or PagerDuty.

### Routing

With [routing blocks](https://prometheus.io/docs/alerting/latest/configuration/#route), a tree of routes and child routes can be defined. Each routing block has a matcher which can match one or several labels of an alert. Per block, one receiver can be specified, or if empty, the default receiver is taken.

### amtool

As routing definitions might be very complex and hard to understand, [amtool](https://github.com/prometheus/alertmanager#examples) becomes handy as it helps to test the rules. It can also generate test alerts and has even more useful features. More about this in the labs.

{{% onlyWhen baloise %}}


### Default Configuration

Alertmanager's configuration is managed by the monitoring stack. Take a look at the default configuration in use at Baloise:

```yaml
# baloise config
global:
  resolve_timeout: 5m
  http_config:
    follow_redirects: true
  smtp_from: devops@example.com
  smtp_hello: localhost
  smtp_smarthost: smtp.example.com:25
  smtp_require_tls: false
route:
  receiver: default
  group_by:
  - namespace
  - alertname
  continue: false
  routes:
  - receiver: mail-critical
    match_re:
      severity: critical|warning
    continue: true
  - receiver: deadmanswitch
    match_re:
      alertname: DeadMansSwitch
    continue: false
    group_wait: 0s
    group_interval: 5s
    repeat_interval: 1m
  - receiver: teams-critical-prod
    matchers:
    - env="prod"
    - severity="critical"
    continue: false
  - receiver: teams-warning-prod
    matchers:
    - env="prod"
    - severity="warning"
    continue: false
  - receiver: teams-info-prod
    matchers:
    - env="prod"
    continue: false
  - receiver: teams-critical-nonprod
    matchers:
    - env!="prod"
    - severity="critical"
    continue: false
  - receiver: teams-warning-nonprod
    matchers:
    - env!="prod"
    - severity="warning"
    continue: false
  - receiver: teams-info-nonprod
    matchers:
    - env!="prod"
    - severity="info"
    continue: false
  - receiver: teams-warning-prod
    matchers:
    - env!="prod"
    continue: false
  group_wait: 30s
  group_interval: 1m
  repeat_interval: 12h
inhibit_rules:
- source_match:
    severity: critical
  target_match_re:
    severity: warning|info
  equal:
  - namespace
  - alertname
- source_match:
    severity: warning
  target_match_re:
    severity: info
  equal:
  - namespace
  - alertname
receivers:
- name: default
- name: mail-critical
  email_configs:
  - send_resolved: false
    to: group.devops_system@example.com
    from: devops@example.com
    hello: localhost
    smarthost: smtp.example.com:25
    headers:
      From: devops@example.com
      Subject: '{{ template "email.default.subject" . }}'
      To: group.devops_system@example.com
    html: '{{ template "email.default.html" . }}'
    require_tls: false
- name: teams-critical-prod
  webhook_configs:
  - send_resolved: true
    http_config:
      follow_redirects: true
    url: http://localhost:8089/v2/critical
    max_alerts: 0
- name: teams-warning-prod
  webhook_configs:
  - send_resolved: true
    http_config:
      follow_redirects: true
    url: http://localhost:8089/v2/warning
    max_alerts: 0
- name: teams-info-prod
  webhook_configs:
  - send_resolved: true
    http_config:
      follow_redirects: true
    url: http://localhost:8089/v2/info
    max_alerts: 0
- name: teams-critical-nonprod
  webhook_configs:
  - send_resolved: true
    http_config:
      follow_redirects: true
    url: http://localhost:8090/v2/critical
    max_alerts: 0
- name: teams-warning-nonprod
  webhook_configs:
  - send_resolved: true
    http_config:
      follow_redirects: true
    url: http://localhost:8090/v2/warning
    max_alerts: 0
- name: teams-info-nonprod
  webhook_configs:
  - send_resolved: true
    http_config:
      follow_redirects: true
    url: http://localhost:8090/v2/info
    max_alerts: 0
- name: deadmanswitch
  webhook_configs:
  - send_resolved: false
    http_config:
      follow_redirects: true
    url: http://deadmanswitch:8080/ping/...
    max_alerts: 0
templates: []
```

{{% /onlyWhen %}}

### More (advanced) options

For more insights of the configuration options, study the following resources:

* Example configuration provided by [Alertmanager on GitHub](https://github.com/prometheus/alertmanager/blob/master/doc/examples/simple.yml)
* General overview of [Alertmanager](https://prometheus.io/docs/alerting/latest/alertmanager/)

## Alerting rules in Prometheus

[Prometheus alerting rules](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/) are configured very similarly to recording rules which you will get to know later in this training. The main difference is that the rules expression contains a threshold (e.g., `query_expression >= 5`) and that an alert is sent to the Alertmanager in case the rule evaluation matches the threshold. An alerting rule can be based on a recording rule or be a normal expression query.

{{% alert title="Note" color="primary" %}}
Sometimes the community or the maintainer of your Prometheus exporter already provide generic Prometheus alerting rules that can be adapted to your needs. For this reason, it makes sense to do some research before writing alerting rules from scratch. Before implementing such a rule, you should always understand and verify the rule. Here are some examples:

* MySQL: [mysqld-mixin](https://github.com/prometheus/mysqld_exporter/tree/master/mysqld-mixin)
* Strimzi Kafka Operator: [strimzi/strimzi-kafka-operator](https://github.com/strimzi/strimzi-kafka-operator/blob/master/examples/metrics/prometheus-install/prometheus-rules.yaml)
* General rules for Kubernetes: [kubernetes-mixin-ruleset](https://github.com/prometheus-operator/kube-prometheus/tree/main/manifests)
* General rules for various exporters: [samber/awesome-prometheus-alerts](https://github.com/samber/awesome-prometheus-alerts)

{{% /alert %}}

## Templates for awesome rules

Whenever creating PrometheusRules you can always expect other people having the same problem as you. On this [site](https://samber.github.io/awesome-prometheus-alerts/) you can find a collection of different PrometheusRules for a big amount of cloud native technology.
