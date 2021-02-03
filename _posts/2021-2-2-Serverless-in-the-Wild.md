---
layout: article
title: Serverless in the Wild
tags: Papers
---
Serverless in the Wild: Characterizing and Optimizing the Serverless Workload at a Large Cloud Provider
## 1. Introduction
- Three main aspects to how fast functions can execute and the resources they consume:
  - 1.Having the needed code in memory(warm start) vs. brought in from persistent storage(code start).
  - 2.Give the illusion that all functions are always warm, while spending resources as if they were always cold.
  - 3.Widely varying resource needs and invocation frequencies from multiple triggers, which is hard for predicting invocations.
## 2. Background
- ***Triggers***: functions can be invoked in response to several event types, including HTTP, Event, Queue, Timer, Orchestration, Storage, and others.
- Functions are logically grouped in ***applications***, which is the unit of scheduling and resource allocation.
- A key aspect of FaaS is the trade-off between reducing cold starts by keeping instances warm, and the resources they need.
## 3. FaaS Workloads
### 3.1 Data Collection
- Four related datasets: invocation counts, trigger per function, execution time per function, memory usage per function.
### 3.2 Functions, Applications and Triggers
- HTTP trigger is the most popular in most dimensions. While a large portion of functions have time triggers, their invocations are relatively small.
![avatar](https://raw.githubusercontent.com/ziliuziliu/ziliuziliu.github.io/master/_posts/pic/2021_2_2_1.jpg)
### 3.3 Invocation Patterns
- The number of invocations per day varies by over 8 orders of magnitude for functions and applications. 
- The vast majority of applications and functions are invoked, on average, very infrequently, suggesting that the cost of keeping these applications warm can be prohibitively high.
- Internal-arrival time variability suggests that:
  - There is a significant fraction of applications that should have fairly predictable IATs, even if they do not have time triggers.
  - For many applications predicting IATs is not trivial. The time-period between invocations is highly varied and thus unpredictable.
### 3.4 Function Execution Times
- The function execution times are at the same order of magnitude as the cold start times reported for major providers, implicating that optimizing cold starts is extremely important for the overall performance of a FaaS offering.
### 3.5 Memory Usage
- 90% of the applications never consume more than 400MB. Overall, there is a 4x variation in the first 90% of applications, meaning that memory is an important factor in warmup, allocation, and keep-alive decisions for FaaS.
### 3.6 Main Takeaways
## 4. Managing Cold Starts in FaaS
- Hybrid Histogram Policy: reducing the number of cold start invocations with minimum resource waste.
  - Pre-warming window: the time the policy waits, since the last execution, before it loads the application image expecting the next invocation.
  - Keep-alive window: the time during which an application's image is kept alive after it has been loaded to memory.
### 4.1 Design Challenges
- Hard-to-predict invocations, heterogeneous applications, applications with infrequent invocations(take some time to learn invocation patterns), tracking overhead, execution overhead(no expensive prediction techniques).
### 4.2 Hybrid Histogram Policy
- An overivew of policy is shown below. Ultimately, the policy defines the pre-warming and keep-alive windows for each application.
![avatar](https://raw.githubusercontent.com/ziliuziliu/ziliuziliu.github.io/master/_posts/pic/2021_2_3_1.jpg)
- ***Range-limited histogram***
  - A compact histogram data structure that tracks the IT(idle time) distribution. Any ITs longer than 4 hours are considered OOBs(out-of-bounds).
  - For IT distribution, use head(5%) to select the pre-warming window, use tail(99%) to select keep-alive window.
- ***Standard keep-alive window when the pattern is uncertain***
  - The histogram might not be representative of an application's behavior when (1) not enough IT's observed (2) application is transitioning to a different IT regime.
  - Revert to a standard keep-alive approach: pre-warming window = 0, keep-alive window = 4hour.
- ***Time-series analysis when histogram is not large enough***
  - Applications with very infrequent invocations may exhibit many out-of-bounds ITs. For these applications, use time-series analysis(ARIMA modeling) to predict the next IT.
### 4.3 Implementation in Apache OpenWhisk
- OpenWhisk architecture
![avatar](https://raw.githubusercontent.com/ziliuziliu/ziliuziliu.github.io/master/_posts/pic/2021_2_3_1.jpg)
- Implementing policy:
  - 1.Add new logic to the Load Balancer to implement the hybrid policy.
  - 2.Send the latest keep-alive parameter alongside the invocation request.
  - 3.The invoker unloads Docker containers based on the keep-alive parameter.
## 5. Evaluation
### 5.1 Methodology
### 5.2 Simulation Results
### 5.3 Experimental Results
## 6. Production Implementation
- Updating the in-memory histogram, backing up the histogram to database, scheduling pre-warming events, and controlling the worker's keep alive intervals.
## 7. Related Work
- SOCK: optimize the loading of Python functions in OpenLambda by smart caching of sets of libraries, and by using lightweight isolation mechanisms for functions.
- SAND: use application-level sandboxing to prevent the cold start latency for subsequent function invocations within an application.
- Replayable Execution: checkpointing and sharing of memory among containers to speed up the startup times of a JVM-based FaaS system.
- This paper works on reducing the number of cold starts and resource usage by predicting function invocations, which is orthogonal to these improvements.
## 8. Conclusion