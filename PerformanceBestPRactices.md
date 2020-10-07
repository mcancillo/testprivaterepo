# Performance guidelines
 	


## Introduction

Current recommendation is to have one subscription per application portfolio or per application
https://docs.omnia.equinor.com/products/product-ordering/#recommendation
 
When multiple subscriptions are used for one application portfolio than we have the following initial performance risks
1.	Fragmented resources over multiple datacenters when using one region
2.	Fragmented resources over multiple datacenters when using multiple regions

This document is aimed at the performance guidelines
**NOTE**: Resilience and BCDR are covered in other documents.

## How is Azure built up

It is important to have a high level overview on how Azure is Built up. Management Groups and Subscription exists on a global level (and there are more components that exist on this level).

Regions, at this moment Azure has 61 regions (at this moment 2-Oct-20). The majority of regions have 1 or more datacenters.
In order to support Availability Zones in a region multiple datacenters are required.
The datacenters that construct a region are bound to a maximum latency (as shown below).
[Region and datacenter](/img/tshirt.png)  
When resources are deployed in a region this does not guarantee that all located in one datacenter. There are scenarios, often when High Availability is high(er) in priority or when there are strict security rules.
 
## Default Subscription topology

Equinor has a Hub-Spoke topology, as shown below, which means that all cross VNET traffic traverses through the regional  HUB VNET in which the firewall lives and enforces the security measurements.
[HubAndSpoke](/img/hubspoke.png)  


## General performance Guideline

As a basic guideline it is best practice to place as much as possible into a single subscription; Resource Group and VNET. This has by default the best performance behavior. 

When this isn't possible, due to:
 * Multiple application teams
 * Multiple LOB's
 * Subscription Limits
 * Security and Compliance (e.g. need for segmentation)
only then use multiple subscriptions.

The most important advise is to design from the application point of view and use the building blocks that fit the application (portfolio).



## Scenario checks

First is the characteristics and building blocks used in the solution. Are the components used all IaaS based? Or is there a combination of IaaS and PaaS components.

When everything is IaaS based than we have to look at: 


## System/VM performance requirements 
 * [Storage tier(s)](https://docs.microsoft.com/en-us/azure/storage/)
 * VM CPU performance
 * Autoscaling needs. [Scale Up](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-vertical-scale-reprovision) or [Scale Out](https://docs.microsoft.com/en-us/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview)
 * Memory
 * Network Throughput

In other words, is the right [VM SKU selected](https://docs.microsoft.com/en-us/azure/virtual-machines/sizes)



## Inter system/VM requirements

Is Latency a concern 
 * Does the traffic need to traverse through the HUB NVA (PaloAlto Firewall) from a security requirement?
 * Do the systems live in the same VNET? This is strongly recommended to reduce the inter system communication latency
 * Is the inter system communication chatty or require real low latency? If this is the case than using Proximity Placement Group is the best option. When there are HA requirements then please have a look at: https://docs.microsoft.com/en-us/azure/virtual-machines/windows/proximity-placement-groups-portal
 * [Enable Network Accelerated Networking](https://docs.microsoft.com/en-us/azure/virtual-network/create-vm-accelerated-networking-cli#limitations-and-constraints) if needed and if the VM SKU supports this



## High volume traffic between VNET's 

 *  Option 1. If the security policy permits then having direct VNET peering between the two applications which are spread across the 2 VNET's. This should
    *	Reduce the path and as such latency
    *	could introduce a small cost saving (since only [VNET peering](https://azure.microsoft.com/en-us/pricing/details/virtual-network/) is used  ).
    *	Introduce network routing complexity to isolate the network flows.
 * Option 2 could be to implement Private Links Service for the source application and place a Private Endpoint in the target VNET.  This would have the following benefits:
    *	No VNET peering is required, simplifying the topology. From the following perspectives
    *	Routing complexity and potential security concerns as only the application is presented and nothing else within the application subscription.
    *	Cross Region availability
    *	Utilizing the Azure backbone
    *	There are costs associated to [Private Endpoints](https://azure.microsoft.com/en-us/pricing/details/private-link/) but these do not differ much compared to VNET peering.

Using Private Endpoints, option2, would be preferred. This is based on the benefits this brings compared to option one.   




## Application Design scenario's

**NOTE**: This document is focussed on Performance (Resilience and BCDR are covered in other documents).

## Example 1 - Chatty application Portfolio with 1 application owner


Before we can start the design process we need to understand the (performance) characteristics and requirements of the solution as a whole and per sub application.
Once this is clear than we can start mapping the information shared above on the solution/application.

* Since we have one Application owener we should be able to use 1 subscription.
* The application portfolio is chatty and for example sake we make it chatty within one of the sub application. To address this need we can use the proximity placement groups. This provides the fastest inter system communication.
* limiting the amount of VNET's and use subnet segmentation between applications with the goal to keep the inter system/application communication local to the VNET so that it doesn't have to traverse over the Hub.
* The IaaS based DBMS service used in the application portfolio needs to have a memory optimzed VM SKU
* Application servers need to get the VM SKU that matches their characteristics

[PEusage](/img/PEusage.png)

## Example 2 - application Portfolio spread across multiple application owners


Similiar to the first example we need to start with understanding the (performance) characteristics and requirements of the solution as a whole and per sub application before we start with the design.
Once this is clear than we can start mapping the information shared above on the solution/application.

* Since we have multiple application oweners we need multiple subscriptions.
* For inter application communication, for this example there is no compliance and security need to traverse through the Hub network. This gives us 2 options. Direct VNET peering and using a Private Endpoint. From an architectual and operational point of view using Private Endpoints would be the preferred solution (connection would be as shown below). 
* The IaaS based DBMS service used in the application portfolio needs to have a memory optimzed VM SKU
* Application servers need to get the VM SKU that matches their characteristics

[Region and datacenter](/img/tshirt.png)
















