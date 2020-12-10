# Prometheus Guide for Beginners
While prometheus is a great tool for monitoring metrics, it has somewhat of a difficult learning curve because of all the pieces moving around. Nobody wants to read boatloads of text and have to figure out where to start. I’m hoping this can be that guide for beginners.

#### Why Prometheus?
We want this service for two simple purposes:
* Alert on high impact issues that affect our service
* Create metrics to help us investigate and troubleshoot issues. These could also give us insight into our system.  

## Introduction: Understanding The Basics

### Architecture 
In its simplest form, prometheus can function with the following services
* Prometheus - Collects all the metrics from many different exporters and handles the alerting logic. When an alert triggers, prometheus sends it to the alertmanager. 
* Alertmanager - Handles alert routing and sends them to external services (e.g: slack)
* Exporter - An exporter is just their fancy term for "http service that provides metrics in a plaintext, prometheus-compatible format". (You'll likely have many of these)
* Notification  - This is an external service that receives the alerts and notifies us. 

### Configuration 
Both prometheus and alertmanager have their own configs. The alert logic is actually managed by prometheus. Alertmanager is responsible for routing those alerts to the right services as well as some extra functionality (silencing, grouping, filtering, etc.). 


### Adding a new metric 
1. Check whether an [out-of-the-box exporter](https://prometheus.io/docs/instrumenting/exporters/) has already been created for your application and provides such a metric. Otherwise, create a [custom exporter](https://github.com/prometheus/client_python) and add the metric. (The link is for python but prometheus has client support for many other languages)
2. Add the new exporter endpoint to “scrape_configs” in the prometheus configuration file. 

*What to know beforehand*
* [Metric types](https://prometheus.io/docs/concepts/metric_types/) - You’ll mainly care about counters/gauges. Only integers are accepted as values. However, you could consider using labels if you have a limited amount of strings
* Stick to a sensible naming convention. If you need a starting point, <service>_<metric>_<unit> is an example. (see [Metric and label naming](https://prometheus.io/docs/practices/naming/) for more info.)

### Adding a new alert
To create an additional alert to an existing config, you just have to create an alert block like the following:	
```
# An alert has two main pieces:
# * expr - A condition that is true/false
# * for - How long a condition has to be consistently true for an alert to trigger. Keep in mind that this could depend on ‘evaluation_interval’ in the config
- name: example_alerting_rules
  rules:
  - alert: Test Counter Alert
    expr: sample_counter_int > 3
    for: 1m 
    labels:
      severity: page 
    annotations:
      summary: Insert some summary here
```

*What to know beforehand*
* You need to know how PromQL works. (Unfortunately, I haven't come up with a guide for this one yet. My suggestion is to go into the prometheus UI and see the graph results of a metric and learn it by testing with different PromQL expressions.)

## Getting Started
1. Read the introduction section first.
2. If you already have a running prometheus service available, take a look at the alerts and the metrics available and understand how each one works. (If it’s too complex, is there a way of simplifying that? Alerts should be simple and easy to read.)
3. [prometheus-sandbox](https://github.com/deeteecee/prometheus-sandbox) is a quick and easy way to setup a basic custom exporter with prometheus. Even if you aren’t planning on creating a custom exporter, it would give you a good feel of how it works.  

## Guidelines
* Metrics and alerts should be short and simple and not depend on complex edge cases/conditions. 
* Even if you have existing metrics and want to add alerts, prioritize thinking about what alerts make sense to you. That could help produce more useful and solid metrics. 
* When you create an alert, you should know how to respond to it as well. (e.g: via playbooks) Otherwise, you’re just making noise. 

## FAQ
* Am I ready to read the prometheus docs?

Yes you are. https://prometheus.io/docs

* Should I use pushgateway or not?

IMO, it adds an extra layer that’s bound to fail. Instead of using it, consider creating a custom exporter that retrieves state data from your application first before going this route.

## Links/Sources
* https://www.robustperception.io/blog - A good source to search for some metric/alerting examples. 
* Book: Prometheus - Up & Running - A slightly more detailed version of docs with a few more examples.
* [My Philosophy on Alerting](https://docs.google.com/document/d/199PqyG3UsyXlwieHaqbGiWVa8eMWi8zzAn0YfcApr8Q/edit) - a short guide written by a former Google SRE
* [Google's SRE Book](https://sre.google/sre-book/table-of-contents/) - I wouldn’t read the whole thing but it has some useful info for ways to think about monitoring/alerting. 


