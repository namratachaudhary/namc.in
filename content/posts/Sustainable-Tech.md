---
title: "Thoughts on embedding sustainability in cloud computing"
tags: [sustainability, cloud, computing]
date: 2022-06-27 12:39:46 +0530
keywords: [sustainability, cloud]
---

Climate change is an issue which affects us all, as industries and individuals. Many businesses and institutions are taking active measures to help achieve carbon neutrality, or “net zero”. However, we have a lot of work to do in software and systems engineering. SRE in particular has a part to play. The industry is increasingly working with huge data volumes, which make the sustainability of our technology choices all the more important for the overall picture. It’s crucial that we have a conversation about this. 

> We always ask how we can make our applications faster, but never think how we can make them greener 🌳

## Carbon footprinting and what it means for SREs 

Lets talk about Net zero. In simple terms, [net zero](https://technation.io/programmes/net-zero/) refers to the balance between the amount of greenhouse gas produced and the amount removed from the atmosphere. This is easy to compute when you talk about, say organic vs inorganic materials. For technology, net zero is more than just carbon neutrality. It extends to transportation, equipment management and energy consumption, end-of-life treatment of equipment, upcycling and recycling.

Many industries have seen great innovation to provide solutions in order to achieve carbon neutrality, and some software firms like [Sweep](https://www.sweep.net/), are helping other companies reduce their carbon emissions. However, in terms of monitoring the impact of infrastructure on environment, there is still a long way to go.

## Driving change with accountability 

### 2. 1 Rethinking data governance

We store large volumes of data, run large amount of applications, in various data stores running in multi-region multi-cloud environments. We try to make internet faster, more available, more reliable. 

I believe, as a part of infrastructure design, aside from determining "requests served per second", we should start evaluating if the technology decisions made are sustainable. 
* Do we need all the extra data we are storing? 
* Are we really using the large chunks of metadata being stored in an obscure database? 
* Is it really necessary to keep a point-in-time events in a datastore permanently and indefinitely? 
* Could this data be aggregated and purged on regular? 

In my opinion, starting with "why do we need this data" and then collecting it is a better way to go about it - not the other way round. 

And while we are talking about data, some peers and friends also pointed out how the standard "indefinite / unplanned" data retention can often lead to privacy, regulatory, and legal risk. With [GDPR standards](https://gdpr-info.eu/), we could help make data storage less carbon intensive.

### 2. 2 Infrastructure's 3Rs

* **Reduce** - 
  * Reducing the number of idle and stranded instances, copies of databases, unused test environments, reserving a whole node for a function when only a portion is necessary.
  * Incorporating sustainability as a part of capacity planning and utilisation of resources in infrastructure design 

* **Reuse** -
  * making infrastructure reusable and extensible by making use of orchestration platforms like kubernetes, which helps in scheduling more than one kind of application on its nodes by sharing resources.  K8s also enables us to treat more of the underlying resource pool as "fungible". 
  * It's still possible to over-scale a cluster vs. its actual utilisation though - so Reduce continues to play a part - and staying ahead of that is a question of good capacity planning (and common cloud tools like autoscaling).

* **Recycle** - 
  * In IT departments as well as data centres, ensure that outdated equipment is properly recycled
  * Major equipment provides have recycling facilities on industrial level as well. 

* **The economics of scale** 
  * [Advanced technologies are designed to reduce electricity requirements, cooling and power conditioning](https://new.abb.com/news/detail/66580/how-data-centers-can-minimize-their-energy-use), this is an important consideration if your infrastructure currently runs on an old data center like AWS's us-east-1
  * Centralisation into the big cloud providers means that instead of building many relatively inefficient datacenters, we are building fewer, larger, and more efficiently designed datacenters. 
  * Big cloud providers can put work into cost reduction efforts that N-smaller companies could never afford - and they're incentivised to do so.

Some organisations feel managing their legacy infrastructure on-prem is better than managing it on cloud. Quoting [Microsoft-WSP study](https://www.wsp.com/en-GB/insights/microsoft-cloud-computing-environmental-benefit-study) - 

> The life cycle approach provided a full picture of the environmental impact of a product or service and the required energy consumption, including:
>  - raw material extraction and equipment assembly,
>  - transportation of the equipment to the datacenter,
>  - use of the equipment and its energy consumption, and
>  - the end-of-life treatment and disposal of the equipment.

With above in mind, perhaps migrating on-prem installation is better managed on cloud while offsetting the carbon footprint due to procurement, management and disposal for data-center and related technology. Also, carbon offset cloud providers (like GCP in some scenarios) are a good first step. Similarly, when one is opting to use a service provider, we should factor in the carbon offset of the said service, and its contribution to netzero during procurement, maintenance and end of life.

### 2.3 Monitoring your cloud resources 

* We monitor uptimes, throughputs, network traffic, could we perhaps monitor the carbon impact of infrastructure as part of our monitoring and observability stack? 
* Does we know how well utilised our systems are? What quantity of stranded resources do we have, or "over-storage" due to naive or nonexistent data collection and retention policy? 
* This is an important case for the investment both as cost and carbon saving. Some monitoring solutions available currently - 
  * AWS - https://aws.amazon.com/blogs/aws/new-customer-carbon-footprint-tool
  * GCP - https://cloud.google.com/carbon-footprint
  * Cloud Carbon Footprint - http://www.cloudcarbonfootprint.org/

## Practical advice for SREs

Rethinking how we manage data by asking “why do we need this data” and then collecting it.

As a "horizontal" function that addresses the whole software lifecycle, SRE is well positioned in many organisations to bring the necessary people together to examine and act on these questions: product management, legal, product engineering, infrastructure.

This is a tough problem to solve, but just as we achieved the mindset of reliability by embedding it in our software architecture design process, we could also start being mindful about the cost of carbon, or start small by taking a look at carbon cost of one service, one datastore. 

However, in conclusion, we do need to take a deeper look at the carbon cost of the infrastructure we are using and further design, and do better at making it efficient AND greener.

