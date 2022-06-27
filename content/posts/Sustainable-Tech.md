---
title: "Thoughts on embedding sustainability in cloud computing"
tags: [sustainability, cloud, computing]
date: 2022-06-27 12:39:46 +0530
keywords: [sustainability, cloud]
---

Lets talk about Ner zero. In simple terms, [net zero](https://technation.io/programmes/net-zero/) refers to the balance between the amount of greenhouse gas produced and the amount removed from the atmosphere. This is easy to compute when you talk about, say organic vs inorganic materials. For technology, net zero is more than just carbon neutrality. It extends to transportation, equipment management and energy consumption, end-of-life treatment of equipment, upcycling and recycling.

Many industries have seen great innovation to provide solutions in order to achieve carbon neutrality, and some software firms like [Sweep](https://www.sweep.net/), are helping other companies reduce their carbon emissions. However, in terms of monitoring the impact of infrastructure on environment, there is still a long way to go.

We store large volumes of data, run large amount of applications, in various data stores running in multi-region multi-cloud environments. We try to make internet faster, more available, more reliable. 

> We always ask how we can make our applications faster, but never think how we can make them greener 🌳

I believe, as a part of infrastructure design, aside from determining "requests served per second", we should start evaluating if the technology decisions made are sustainable. 
* Do we need all the extra data we are storing? 
* Are we really using the large chunks of metadata being stored in an obscure database? 
* Is it really necessary to keep a point-in-time events in a datastore permanently and indefinitely? 
* Could this data be aggregated and purged on regular? 

In my opinion, starting with "why do we need this data" and then collecting it is a better way to go about it - not the other way round.

We monitor uptimes, throughputs, network traffic, could we perhaps monitor the carbon impact of infrastructure as part of our monitoring and observability stack? 

We should focus on making the infrastructure reusable and extensible, so that it can be scaled with greater efficiency. Orchestration platforms like Kubernetes could play a vital role in this, as it enables the users to schedule more than one kind of application on its nodes and does resource allocation efficiently if configurued right.

[Advanced technologies are designed to reduce electricity requirements, cooling and power conditioning
](https://new.abb.com/news/detail/66580/how-data-centers-can-minimize-their-energy-use), this is an important consideration if your infrastructure currently runs on an old data center like AWS's us-east-1

Some organisations feel managing their legacy infrastructure on-prem is better than managing it on cloud. Quoting [Microsoft-WSP study](https://www.wsp.com/en-GB/insights/microsoft-cloud-computing-environmental-benefit-study) - 

> The life cycle approach provided a full picture of the environmental impact of a product or service and the required energy consumption, including:
>  - raw material extraction and equipment assembly,
>  - transportation of the equipment to the datacenter,
>  - use of the equipment and its energy consumption, and
>  - the end-of-life treatment and disposal of the equipment.

With above in mind, perhaps migrating on-prem installation is better managed on cloud while offsetting the carbon footprint due to procurement, management and disposal for data-center and related technology. Also, carbon offset cloud providers (like GCP in some scenarios) are a good first step. Similarly, when one is opting to use a service provider, we should factor in the carbon offset of the said service, and its contribution to netzero during procurement, maintenance and end of life.

This is a tough problem to solve, but just as we achieved the mindset of reliability by embedding it in our software architecture design process, we could also start being mindful about the cost of carbon. We need to take a deeper look at the carbon cost of the infrastructure we are using and further design, and do better at making it efficient AND greener. 

