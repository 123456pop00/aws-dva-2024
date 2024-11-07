---
icon: left-right
---

# ASG

## Features :feather:

* ASG will launch or terminate instances as needed to **rebalance capacity across AZs**, maintaining high availability and fault tolerance.
* **ELB Health Checks** monitor instance health based on load balancer feedback; if an instance fails, ASG replaces it.
* **Lifecycle Hooks** allow custom actions during instance launch or termination (e.g., running a script, notifying an SNS topic).
* Use **Cooldown** with **Simple/Step Scaling** to stabilise metrics after scaling actions.
* Use **Instance Warm-Up** with **Target Tracking Scaling** to delay metric contribution until instances are ready.

## Scaling Policies

> ASG scaling policies allow your application to automatically adjust the number of instances based on predefined metrics, using **CloudWatch alarms** for monitoring metrics like **CPU utilization**, **Network I/O**, or **RequestCount per Target.**
>
>

**Metrics Examples:**

* **CPU Utilization**: Helps manage scaling based on processor load.
* **Network I/O**: Useful for scaling based on network traffic.
* **RequestCount per Target**: Tracks the number of requests to help balance incoming load.

### Dynamic:

*   **Target Tracking Scaling** :dart::

    * Automatically adjusts capacity to keep a specific metric (like average CPU utilization or average number of connections) at a target value.
    * **Example**: Scale out or in to keep **CPU utilization at 50%** or maintain the average number of connections to your EC2 instance at **1,000**.


* **Simple/Step Scaling**:
  * Uses **CloudWatch Alarms** to add or remove instances **in steps** based on specific metric thresholds.
  * **Example**: If **CPU > 70%** is triggered, scale out by adding 3 instances; if **CPU < 40%**, scale in by removing 2 instances.

### Scheduled:

* Schedules scaling actions to **anticipate** :arrows\_counterclockwise: **known traffic patterns** (e.g., predictable traffic spikes or drops).
* **Example**: Scale out at 8 AM on weekdays to handle increased morning traffic.

### Predictive:

* Uses **machine learning (AI)** to analyze historical data and forecast demand, automatically scaling based on these predictions.

### **ASG Cooldown Period:**

* ASG has a default cooldown period of **300 seconds** (5 minutes) for scale-out actions.
* A **cooldown period** is a pause (timeout) after a scaling action during which the ASG will not perform further scale-in or scale-out actions.&#x20;
* Cooldown periods typically apply to **Simple/Step Scaling** and are not required for **Target Tracking Scaling** since it has its own stabilisation mechanisms.&#x20;
  * Simple Scaling and Step Scaling policies rely on fixed thresholds and discrete actions (like adding or removing instances).
  * Target Tracking Scaling does not use cooldowns because it continuously adjusts the capacity to maintain a target metric value, making cooldowns unnecessary. Better use warm-up timeouts to allow instances fully initialise before contributing to metrics.
* Allows time for instances to stabilise, ensuring that scaling metrics (like CPU or request counts) settle before the next scaling action, similar to a backoff delay.
* Helps avoid **rapid or “thrashing” scaling** (where instances are rapidly launched and terminated in quick succession), which could lead to increased costs and application instability.

<details>

<summary><mark style="color:purple;"><strong>Example Scenario for Cooldown</strong></mark></summary>

An application with an ASG that triggers a scale-out action when **CPU utilization exceeds 75%**:

* Once the CPU threshold is reached, ASG scales out by adding instances, and a 300-second cooldown is set.
* During these 300 seconds, ASG won’t scale out further even if the CPU utilization is high, giving the new instances time to balance the load.
* After 300 seconds, if the CPU is still high, another scale-out event can be triggered.

</details>



### **ASG Instance Refresh -** ensures uninterrupted service :star:

**Instance Refresh** is a feature of Auto Scaling Groups that allows you to systematically replace instances in an ASG with new instances launched using an updated configuration or **launch template**. This is particularly useful when you need to update instances with new application code, AMI versions, or instance types. I.e. upgrade the AMI for your application from `AMI-v1` to `AMI-v2`:

* [x] Updates instances in batches, maintains healthy threshold
* [x] The primary goal of Instance Refresh is to **update instances without manual intervention** while ensuring minimal disruption to the application.
* [x] During the refresh, **health checks** (EC2 or ELB health checks) determine whether new instances are ready to replace the old ones. If :heart\_exclamation:check fails ASG will launch another instance in its place, ensuring the minimum healthy percentage is maintained.

**Configuration Options**:

* **Minimum Healthy Percentage**: Defines the minimum percentage of instances that must remain healthy and in-service during the refresh. For example, if set to 80%, only 20% of instances can be replaced at a time, ensuring the rest continue serving traffic.
* **Instance Warm-Up Time**: time ASG waits after launching a new instance before considering it ready (or "**warm**") to handle requests. During this time, health checks and other application initialisation can complete.
  * Warm-up periods are more flexible than cooldown, and provide a more precise control, especially when paired with dynamic scaling.
* **Pause and Resume**: Instance Refresh can be paused if any issues are detected, allowing you to diagnose problems before resuming. You can also cancel it if a refresh is no longer needed.









