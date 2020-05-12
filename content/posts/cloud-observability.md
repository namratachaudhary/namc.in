---
title: "Understanding Observability (and the family) in Cloud Infrastructure"
tags: [Devops]
date: 2020-05-11 16:25:12 +0200
keywords: ["cloud", "kubernetes", "monitoring", "logging"]
tldr: (An introduction to observability in Cloud Infrastructure and how to get started with basic set of tools. Explore Prometheus, Fluentd, Loki ..)
---

This blog post is divided in two parts - 
1. [Understanding the terminlogogies in Observability Ecosystem and corresponding tools](#https://namc.in/posts/cloud-observability/#terminologies-in-observability-ecosystem)
2. [How you can use some bare-minimum tooling to improve observability in your kubernetes cluster](https://namc.in/posts/cloud-observability-ii/)

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
		   
[In next part, we will learn how to use some of these tools to obtain a basic level of observability in your kuubernetes cluster.](https://namc.in/posts/cloud-observability-ii/)

Special thanks to [Srujan's blog post](https://acsrujan.net/monitoring-and-siblings/) for motivation
