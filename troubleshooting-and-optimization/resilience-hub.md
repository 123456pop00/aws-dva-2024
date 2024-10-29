# Resilience Hub

> A central place to define, validate and track the resiliency of your applicaion.  Provides architectural guidance for improving application resiliency, implementing tests, alarms, SOPs( standard ops procedures) that you can deploy and run in CI/CD pipeline.

_Application Resilience - ability to maintain the availability and recover from failure, disruption within_ :clock2: _RTO (Recovery Time Objective, ie time an application can be down without causing significant damage)  and PPO (Recovery Point Objective)._ RPO helps determine how much data a company can tolerate losing during an unforeseen event.

_The things that can go wrong are called faults, and systems that anticipate faults and can cope with them are called fault-tolerant or resilient._

_Need for formal methods to reason (abstractly) about distributed system resilience and gain confidence of system design, formal methods help discover bugs early, their aim is to ensure that system satisfies formal specifications. Especially when you're trying to build a complicated system that is hard to reason about._

_Whats is the assumotion that component makes about other component?_&#x20;

_AWS P formal framework to go over design iterrations._

* Customers need to build a replication mechanism and **validate** that the procedure will work.
* &#x20;Validation is expensive as it involves month of planning.
* For very low RTOs, AWS allows for  **Aurora** ( for cross-region resilience).
* For longer. RTOs (1+hr) use **backup in different AZ or Region.**

## Resilience Hub Features

* Continuously validate & track application resilience with dashboard view.
* Provides resiliency score, which reflects how closely the application follows recommendations for meeting the application's resiliency policy, alarms, SOPs, and tests. Application resiliency score over time graph is presented in the dashboard so the user can monitor his resiliency score over the past 30 days. Max is 100.
* Audit trail for compliance.
* Define RTO & RPO policies and hub will provide procedures to meet them.
* Runs weakness assessments.
* Makes recommendations on SOPs and alarms.
* Clarify & resolve incidents before they occur
* Prepare for outages, by conducting testing and verification.
* CI/CD integration.

<figure><img src="../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

## Testing Resilience

Hub is a fully managed service that provides FIS experiments

Fault injection is a practice in chaos engineering of stressing an application in testing or production environments by creating disruptive events, such as sudden increase in CPU or memory consumption, observing how the system responds, and implementing improvements → helps uncover bugs, 🐞 bottlenecks 🧣 and issues by :

* Simulate a wide range of failures.&#x20;
* Validate that monitors and alarms identify the correct outage.
* Validate that the SOP recovered the application within its resilience targets.
* Integrate into CI/CD pipelines for continuous assessment and testing.



