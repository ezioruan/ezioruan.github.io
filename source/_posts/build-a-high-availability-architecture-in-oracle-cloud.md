---
title: Build a high availability architecture in oracle cloud
date: 2022-08-10 15:07:06
tags: System Architecture
---


This article will introduce how to use Oracle Cloud to build a highly available, cross-regional system. We know that high availability describes a system that is available most of the time and can provide us with services. High availability means the service is still available even during a hardware failure or system upgrade. Eliminate single points of failure through proper use of deploying instances across multiple availability domains
We'll create load balancers in three different regions, each with two servers at least behind them that use the same application and expose the same ports.


Let's take a look at our system design architecture design first.
![architecture](/images/oracle/architecture.png)

In this architecture, We create three load-balancers distributed in the us-west, us-east, and Europe regions.
So we have three regions with identical resources. And the traffic will route to the specific load balancer with rules. And fallback to other load balancers.
We'll show how to create this architecture step by step

<!--more-->


## create load balancers in different regions
- Go to  Network->Load balancer and create a general Load balancer
- Select the corresponding instance network and subnet
- Select Backend Type, Weight Round Robin
- Add backend set, select at least two servers with ports to achieve high availability in the area
- Add Listener; we choose HTTP and listen on port 80. If HTTPS is needed, we can add Listener later
- Use the default health check or adjust it according to the needs of the application

After the creation is complete, the Load balancer will have a public network IP, and we will use this IP in the following steps.


## create a DNS record
Click Create Zone. In the Create Zone dialog, choose Manual.
Enter the name of the zone to develop. Select the organizational account to create the site—Select Primary as the area controlling zone content directly in OCI. Click Create. The system makes and publishes the zone and contains the necessary SOA and NS records.

Then you need to point the name server of the corresponding domain name to the NS record in your domain name management system so that the records created by OCI can take effect.


## Create Traffic Management Steering
After we have the domain name, we can create traffic rules for the given domain.

oracle cloud supports the following traffic rules
- Failover
Failover policies allow you to determine the order in which you want to provide acknowledgments in the policy (for example, primary and secondary). Oracle Cloud Infrastructure health checks are used to determine the health of the answers in the policy. If the health check for the primary answer fails, DNS traffic will automatically be directed to the secondary answer.
- Load balancer
Load balancer policies allow traffic to be distributed across multiple endpoints. Endpoints can be assigned equal weights to distribute traffic evenly across endpoints, or custom weights can be assigned for ratio load balancing. Oracle Cloud Infrastructure Health Checks are used to determine the health of endpoints. If an endpoint is determined to be unhealthy, DNS traffic is automatically distributed to other endpoints.
- Geolocation guidance
Geo-directed policies distribute DNS traffic to different endpoints based on the location of the end user. Customers can define geographic regions consisting of origin continents, countries, or states (North America) and define an individual endpoint or set of endpoints for each region.
- ASN steering
ASN steering policies enable you to steer DNS traffic based on autonomous system numbers (ASNs). DNS queries originating from a specific ASN or set of ASNs can be directed to specified endpoints.
- IP prefix bootstrap
The IP prefix steering policy enables customers to direct DNS traffic based on the IP prefix of the original query.

Here we will use the Geolocation guidance method to configure.
 Select Networking, > Traffic Management Steering Policies -> Create Traffic Management Steering Policy.
Select  Geolocation Steering
 Enter the name of the rules; other fields keep the default.
Add an answer pool. We will create an answer pool for the load-balancer in each area, and the name can be the region name; selects A record and fill in the IP of the corresponding load-balancer
 Define the rules,We can define the direction of the traffic according to the region. Here we create the corresponding answer order according to the priority of our rules.
 Add the Global > Catch-all rule. This rule is when our previous rules do not match the matching rules to be carried out in the future and also creates the corresponding response order according to the priority of our rules
On the bound domain, click Attached Domains. We can add a subdomain, such as hello.example.com

In this way, we have completed all the steps to create a cross-region highly available application in OCI.
