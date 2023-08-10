---
title: "6. Instrumenting with client libraries"
weight: 1
sectionnumber: 1
---

While an exporter is an adapter for your service to adapt a service specific value into a metric in the Prometheus format, it is also possible to export metric data programmatically in your application code.

## Client libraries

The Prometheus project provides [client libraries](https://prometheus.io/docs/instrumenting/clientlibs/) which are either official or maintained by third-parties. There are libraries for all major languages like Java, Go, Python, PHP, and .NET/C#.

Even if you don't plan to provide your own metrics, those libraries already export some basic metrics based on the language. For [Go](https://prometheus.io/docs/guides/go-application/), default metrics about memory management (heap, garbage collection) and thread pools can be collected. The same applies to [Java](https://github.com/prometheus/client_java#included-collectors).

## Specifications and conventions

Application metrics or metrics in general can contain confidential information, therefore endpoints should be protected from unauthenticated users. This can be achieved either by exposing the metrics on a different port, which is only reachable by prometheus or by protecting the metrics endpoints with some sort of authentication.

There are some guidelines and best practices how to name your own metrics. Of course, the [specifications of the datamodel](https://prometheus.io/docs/concepts/data_model/#metric-names-and-labels) must be followed and applying the [best practices about naming](https://prometheus.io/docs/practices/naming/) is not a bad idea. All those guidelines and best practices are now officially specified in [openmetrics.io](https://openmetrics.io).

Following these principles is not (yet) a must, but it helps to understand and interpret your metrics.

## Best practices

Though implementing a metric is an easy task from a technical point of view, it is not so easy to define what and how to measure. If you follow your existing [log statements](https://prometheus.io/docs/practices/instrumentation/#logging) and if you define an error counter to count all [errors and exceptions](https://prometheus.io/docs/practices/instrumentation/#failures), then you already have a good base to see the internal state of your application.

### The four golden signals

Another approach to define metrics is based on [the four golden signals](https://sre.google/sre-book/monitoring-distributed-systems/):

* Latency
* Traffic
* Errors
* Saturation

There are other methods like [RED](https://www.weave.works/blog/the-red-method-key-metrics-for-microservices-architecture/) or [USE](http://www.brendangregg.com/usemethod.html) that go into the same direction.

## Collecting Application Metrics

When running applications in production, a fast feedback loop is a key factor. The following reasons show why it's essential to gather and combine all sorts of metrics when running an application in production:

* To make sure that an application runs smoothly
* To be able to see production issues and send alerts
* To debug an application
* To take business and architectural decisions
* Metrics can also help to decide when to scale applications
