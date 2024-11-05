---
icon: chart-network
---

# ELB & ASG

> Everything fails all the time, so plan for failure and nothing fails.

**Elasticity** - refers to the cloud's ability to automatically scale resources up or down in response to varying workloads, dynamic scaling allows  to pay only for the resources used, on-demand. &#x20;

**HA** - resistance to common failures through design and operational mechanism, HA ensures that applications are consistently operational and accessible, with RTO & RPO target downtime, achieved through redundancy, failover strategies, and operational best practices that mitigate the impact of common failures.

**Scalability** - capability of a system to handle increased loads by either upgrading existing resources (vertical scaling) or adding more resources (horizontal scaling) .

* **Vertical Scaling:** Upgrading existing resources (e.g., CPU, RAM) on a single instance.
  * **Use Cases**:&#x20;
    * Legacy applications needing high performance but not designed for distributed environments.
    * Applications like databases where single-instance performance is critical (e.g., PostgreSQL, MySQL).
  * **Advantages**: Elastic vertical scaling,  for applications that benefit from resized instance based on workload demands, like experiencing sudden spikes but can remaining idle during off-peak. Changing instance types is straightforward without needing to re-architectuirng for horizontal scaling (particularly legacy).
  * **Limitations** :
    * Growth is constrained by the maximum capacity of the hardware.
    * Potential downtime during upgrades.
* **Horizontal Scaling**:  more instances to distribute the load
  * **Use Cases**:&#x20;
    * Ideal for distributed systems that require flexibility and fault tolerance.
    * Web applications that can handle many users (e.g., e-commerce sites, content delivery).
  * **Advantages**: Not limited by hardware, allowing for virtually unlimited growth by adding more instances.
  * **Limitations**:
    * More complex architecture and management.
    * Potential consistency issues across instances.

**Reliability** - ability of a workload to perform its required function correctly and consistently, includes  aspects such as uptime, fault tolerance, and adherence to service level agreements (SLAs).

**Resilience** -  workload ability to recover from infrastructure or service disruptions,  including disaster recovery mechanisms and proactive measures to prevent outages.



## ELB - elastic LB

* Managed by AWS
  * AWS takes care of upgrade, maintenance, integration
  * Integration with multiple services ( Certificate manager, CloudWatch, WAF, Route53)
* Spread load across multiple downstream EC2s
* Regular health checks to the EC2 instances
* Handle failure downstream → Health checks of backend EC2s if one instance is failing traffic is **redirected**
  * Check on the port, route, endpoint /health&#x20;
* Expose **single point of access**
* DNS host name for your application
* Provide SSL termination (HTTPS) for your websites
* Multi AZ

#### ELB Types :

1. **ALB** (HTTP/HTTPS only) - Layer 7
2. **Network LB** (ultra-high performance, allows for TCP, UDP, and TLS traffic)- Layer 4
   1. suited for load balancing Transmission Control Protocol (TCP), User Datagram Protocol (UDP), and Transport Layer Security (TLS) traffic and has the capability of handling **millions of requests per second** while maintaining ultra-low latencies
3. **Gateway** LB - Layer 3
4. Classic LB ( Layer 4 +7) / depreciates in 2023

Typical HA architecture is multi-AZ ( minimum 2 ) with cross-region replication for data redundancy,  disaster recovery, safeguarding against region-specific failures.

HA leverages ELB, which is a server that distributes traffic to servers downstream. Typically, it is configured to direct traffic to the back end that has the least outstanding requests. Backend scales, once the new instance is ready ( configure ASG) and tells the ELB that it can take traffic. The frontend doesn't know and doesn't care how many back end instances are running. This is **true decoupled architecture.**

* #### Instances inform LB about their capacity
* Health Check mechanism :green\_heart: , periodically sends metrics
* As ASG backend adds new instance, new instance tells ELB when it can take request





* **What to use to handle hundreds of thousands of connections with low latency?A Network Load Balancer can handle millions** of requests per second with low-latency. It operates at Layer 4, and is best-suited for load-balancing TCP, UDP, and TLS traffic with ultra high-performance.

