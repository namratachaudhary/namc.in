---
title: "Observability Cloud Infrastructure - Using the right tools"
tags: [Devops]
date: 2020-05-12 06:25:12 +0200
keywords: ["cloud", "kubernetes", "monitoring", "logging"]
tldr: (An introduction to observability in Cloud Infrastructure and how to get started with basic set of tools. Explore Prometheus, Fluentd, Loki ..)
---

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
