---
title: "Observability Family in cloud Infrastructure"
tags: [Devops]
date: 2020-05-11 16:25:12 +0200
keywords: ["cloud", "kubernetes", "monitoring", "logging"]
tldr: (An introduction to observability in Cloud Infrastructure and how to get started with basic set of tools. Explore Prometheus, Fluentd, Loki ..)
---

## Terminologies in Observability Ecosystem

With a huge shift towards Kubernetes and therefore, a distributed systems service architecture, we see a constant need to have solid observability practices. This ranges from application-level monitoring to OS & Network monitoring, as well as capturing telemetry and kubernetes events.

On the CNCF landscape, numerous tools are available but it is often hard to find the right tools for the right job. This process can also be overwhelming.

Let’s begin with understanding some fundamental concepts in the universe of observability -

### Monitoring

-   Questions like - "What's happening currently?" and “is my service behaving abnormally right now?”
-  Metrics gathered from application and services are observed.    
-   Usually warrants usage of dashboards and a query language interface to generate reports.
-   Observable containers are imperative to implement cluster-wide monitoring.
-   Sample usage would be - “what even caused a sudden spike in CPU usage?”
-   Suggested tools : 
	-   Prometheus
	-   Sysdig
	-   Cloudwatch metrics in AWS
    
### Alerting

-   This step is critical. Since we cannot keep a constant watch over each metric that we capture, we setup an alerting mechanism which compares present metric data with a user-defined or AI-assisted thresholds
-   Alerting usually calls for an action by system maintainers.
-   This is implemented and configured on an infrastructure level.
-   Sample usage would be - the monitoring system observed that 20 requests in last 1 minute took more than 10ms as opposed to 5ms on average, hence, a human should be alerted to check the system health.
-   Suggested tools -
	-   PagerDuty
	-   Alerting tools integrated in public cloud providers. (SNS)
	-   Uptime
	
### Logging

-   Implemented on application level, it refers to responsible behaviour of the developer and application to make a note of all necessary and relevant information in a workflow.
-   Implemented on system level, it an indication of all the events triggered by the system when a workflow is executed.
-   There are multiple log levels - debug, error, fatal, info. They should be enforced in the business logic (application code) to improve visibility within the application.
-   Logs are very useful to debug applications and services without stopping them and determining if the workflow is broken somewhere.
-   Logs are often routed to ELK stack, sentry.io etc ; combined across services with a unique context ID and an query language interface is provided to get logs pertaining to a particular function call or context ID.
-   Sample usage would be - logging all steps involved when an http request is initiated, i.e. the request payload as debug log, an exception or external service error as Error logs
-   Suggested tools -
	-   ELK stack
	-   Loki log framework
	-   Cloud provider specific tools
		-   Azure Monitor
		-   AWS Centralized Logging
		-   Google Cloud Stackdriver Logging
    
### Tracing

-   Tracing is in-line with logging but it is constrained by single workflow as opposed to application-wide logging.
-   Cases where you want to trace all function call made when a workflow is triggered, starting from audit logs to variable initialization, class instance being created to final result of success or error.
-   Traces are important for developers who want to capture multiple transformation of input payloads within their services, thereby providing visibility over how the business logic behaves.
-   Sample usage would be - "A single users' (or a small sample set) journey across the software (it can be part/whole of multiple software in a larger system too). this is to identify what the action flow is and how is it impacted in normal vs error-prone cases"
-   Suggested tools -
	-   Jaeger Tracing
	-   OpenTracing framework for cloud apps  
	-   Zipkin
	-   Sentry.io for error tracing

### Telemetry

-   Telemetry provides an overview of how your services interact with each other.
-   If you have 20 microservices in a cluster that communicate with each other using HTTP or GRPC or queues, telemetry software would lay out the network topology which depicts which service is dependent on which other service for communication.
-   In a distributed system, telemetry helps in debugging network workflows and bottlenecks, and also helps in optimizing network iops.
-   Suggested tools -
	-   For capturing :
		-   Riemann 
		-   Fluentd 
		-   Collectd
	-   For visualization :
		-   Grafana
		-   Kibana
	   -   Third-party vendors :
		    -   Datadog
		   -   Dynatrace
		   -   New Relic
    
## Creating a very basic and bare-minimum observable infrastructure

Considering that the system engineer is somewhat well-versed with cloud-friendly observability tools, let’s look at some tools that can help lay-down a sound foundation for system observability

### [Prometheus](https://prometheus.io/) 
**to scrape data from K8S resources**

A good practice would be to use observable containers (application containers that push metrics to an endpoint). A lot can be automated with CR auto discovery from the Prometheus Operator. If you are using helm charts, this is already enabled by default. Prometheus provides an interface for querying metrics (PromQL)

Deploy using Prometheus operator :
-   simplifies configuration by using CRDs to define the jobs and rules.
-   far easier to maintain configurations in a bunch of small files, rather than a monolithic config with multiple scrapes.
-   adds common components (node-exporter, kube-state-metrics, grafana, etc).
    
### [VictoriaMetrics](https://victoriametrics.com/) / [Thanos](https://thanos.io/getting-started.md/) 
**for storing all historical metrics data**

Prometheus stores data on disk, therefore you can assume that your metric data will be short lived.  Victoria Metrics and Thanos is built on top of Prometheus and stores all your historical metric data for unlimited time Compatible with PromQL therefore, it can be used to generate visualizations on Grafana using the same queries as that for Prometheus. It exposes a "master Prometheus" to scrape data from all Prometheus instances.
    
Some tangential but necessary advantages include - high availability, querying across instances, long-term block storage, query-level downsampling
    
### Grafana  
**to visualize metrics data and logs**

Grafana lays out the metrics in visual appeasing manner using templates provided (proprietary, OSS). Grafana integrates beautifully with Prometheus and it is compatible with PromQL. You can build custom charts using PromQL and combine multiple metrics across applications.
    
Grafana is a state of the art visualization tool in the industry, and open source too. It integrates with plenty of other tools, thereby making it a one-stop shop for all visualization needs. 

### [Loki](https://github.com/grafana/loki) 
**for building a logging framework**

This tool is absolutely necessary if you want to process log files. However, it does not index log files [like ElasticSearch] but indexes metadata. 

If you don't need Fulltext indexing, and grepping your logs is sufficient use case, go for `loki`. It will be fast, reliable, already well integrated in kubernetes and managed easier. It is also compatible with Grafana, hence no need for additional visualization tools.

### [Fluentd](https://www.fluentd.org/) 
**for collecting all your logs**

 A log collector to gather all kinds of logs and translate them to JSON. It enables other systems to seamlessly work with the data. 

There are multiple options to setup Fluentd with your system - 

* setup a logfile watcher as input, which sends the log messages over to a logfile writer on another machine, all within 5..10 minutes on 2 VMs.
* directly write messages (JSON or MessagePack) to a TCP socket (MessagePack-formatted messages to TCP socket is basically the lowest level you get).

It's basically just input plugins connected to output plugins, and you build a message distribution network from that. 

Fluentd is automatically installed by Google on their GKE instances.
    
### [KubeCost](https://kubecost.com/) 
**to determine how much your kubernetes resources are costing you**

KubeCost provides cost allocation models for kubernetes workloads across various cloud providers. In a shared cluster, it helps in linking the costs to the resources and link that to users/teams. This also helps in being able to estimate, analyse, and create accurate reporting on costs

This is truly essential for firms that operate on a budget and need more visibility of resource allocation and optimization within the organisation

### Final words 

It's a rich ecosystem with flexible and powerful parts. As a warning, there are strong opinions in how to configure certain things. You may require supplemental work depending on your org's requirements and what you want to accomplish.

So head over to the CNCF landscape and create the stack you like. 

Special thanks to [Srujan's blog post](https://acsrujan.net/monitoring-and-siblings/) for motivation
