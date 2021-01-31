---
layout: article
title: Berkeley View on Serverless Computing
tags: Papers
---
Cloud Programming Simplified: A Berkeley View on Serverless Computing
## 1. Introduction to Serverless Computing
- Cloud computing relieved users of physical infrastructure management, but left them with a proliferation of virtual resources to manage, and a dozen of environments to be set...
![avatar](https://raw.githubusercontent.com/ziliuziliu/ziliuziliu.github.io/master/_posts/pic/berkeley_environment.jpg)
- Serverless computing = FaaS(cloud functions) + BaaS. For a service to be considered serverless, it must scale automatically with no need for explicit provisioning, and be billed based on usage.
## 2. Emergence of Serverless Computing
- Serverless cloud functions vs. serverful cloud VMs
![avatar](https://raw.githubusercontent.com/ziliuziliu/ziliuziliu.github.io/master/_posts/pic/berkeley_serverless_vs_serverful.jpg)
- In the cloud context, serverful computing is like programming in low-level assembly language, whereas serverless computing is like programming in a higher-level language such as Python.
- Critical distinctions between serverless and serverful computing:
  - 1.Decoupled computation and storage. (FaaS + BaaS)
  - 2.Executing code without managing resource allocation.
  - 3.Paying in proportion to resources used instead of for resources allocated.
### 2.1 Contextualizing Serverless Computing
- Serverless computing with cloud functions differs from its predecessors in serveral essential ways: better autoscaling, strong isolation, platform flexibility, and service ecosystem support.
  - It charged the customer for the time their code was actually executing, not for the resources reserved to execute their program.
  - Strong performance and security isolation is needed to make multi-tenant hardware sharing possible. Reduce the overhead!
- K8s is a technology that simpifies management of serverful computing, while serverless computing introduces a paradigm shift that allows fully offloading operational responsebilities to the provider, and makes possible fine-grained multi-tenant multiplexing.
### 2.2 Attractiveness of Serverless Computing
- Deploy without any understanding of the cloud infrastructure, save development time, stay focused on problems unique to their application, and save money.
- A brand new programming abstraction for researching.
## 3. Limitations of Today's Serverless Computing Platforms
- Requirements for new application
![avatar](https://raw.githubusercontent.com/ziliuziliu/ziliuziliu.github.io/master/_posts/pic/berkeley_requirements.jpg)
### 3.1 Inadequate storage for fine-grained operations
- The stateless nature of serverless platforms makes it difficult to support applications that have fine-grained state sharing needs.
- Different applications will likely motivate different guaranteens of persistence and availability, and perhaps also latency or other performance measures.
### 3.2 Lack of fine-grained coordination
- To expand support to stateful applications, serverless frameworks need to provide a way for tasks to coordinate.
  - When is input available? Need notification services...
### 3.3 Poor performance for standard communication patterns
- Broadcast, aggregation and shuffle are 3 standard communication patterns.
- VM instances offer ample opportunities to share, aggregate or combine data locally across tasks before sending it or after receiving it. => FaaS solution suffers from a high complexity of data-transferring.
### 3.4 Predictable performance
- Cold-start latency can be high in
  - Starting a cloud function
  - Initializing the software environment(load the libraries,...)
  - Application-specific initialization in user code
- The hardware resources is highly variable(different generations), thus unpredictable.
## 4. What Serverless Computing Should Become
### 4.1 Abstraction challenges
- For ***Resource requirements***: Raise the level of abstraction, having the cloud provider infer resource requirements instead of having the developer specify them.
- For ***Data dependencies***: cloud provider should expose an API that allows an application to specify its computational graph, enabling better placement decisions that minimize communication and improve performance.
### 4.2 System challenges
- High-performance, affordable, transparently provisioned storage
  - Ephemeral storage(cache) is needed to maintain application state during the application lifetime. 
  - Solution: a distributed in-memory service that can automatically scale the storage capacity and the IOPS with the application's demands, also with access protection and performance isolation provided.
  - Leveraging statistical multiplexing.
  - Durable storage is needed for certain applications that require longer retention and greater durability.
  - Solution: an SSD-based distributed store paired with a distributed in-memory cache.
- Coordination/signaling service
  - Including sharing state between funcitions via a producer-consumer design pattern, or signal a function when a condition becomes available.
  - Would benefit from microsecond-level latency, reliable delivery, and broadcast or group communication.
- Minimize startup time
  - Resource scheduling and initialization can incur significant delays and overheads.
  - Unikernel, perform startup tasks ahead of time...
### 4.3 Networking challenges
- Cloud functions can impose significant overhead(complexity) on popular communication primitives.
- Let application provide a computation graph, enabling the cloud provider to co-locate the cloud functions to minimize communication overhead.
### 4.4 Security challenges
- Physical co-residency is the center of hardware-level side-channel attacks inside the cloud. Need a randomized, adversary-aware scheduling algorithm and physical isolation techniques.
- Cloud function need fine-grained configuration, and there may be security dependencies between cloud functions. A capability-based access control mechanism using cryptographically protected security contexts could be a natural fit for such a distributed security model.
- Search for strong isolation mechanisms with low startup overheads. Light-weight virtualization technologies are starting to be adopted by cloud providers.
- Cloud functions can leak access patterns and timing information through network communication, so we need to adopt oblivious algorithms.
### 4.5 Computer architecture challenges
- Hardwares that dominated the cloud are barely improving in performance. Two paths forward: (1) language-specific custom processors for high-level scripting languages (2) Domain Specific Architectures tailored to a specific problem domain.
- Serverless computing has to support the upcoming heteroeneity:
  - 1.Serverless could embrace multiple instance types, with a different price per accounting unit depending on the hardware used.
  - 2.The cloud provider could select language-based accelerators and DSAs automatically.
## 5. Fallacies and Pitfalls
- The beauty of serverless computing is that all the system administration capability is included in the price, including redundancy for availability, monitoring, auto-scaling and logging. So, it's possible that serverless computing can become much less expensive.
- To achieve portability among different cloud providers, serverless users have to propose and embrace some kind of standard API.
## 6. Summary and Predictions
- Finally, the author concludes the paper with the following predictions:
  - 1.We expect new BaaS storage that match the performance of local block storage and come in ephemeral and durable variants.
  - 2.We expect serverless computing become simpler to program securely, benefiting from the high level of program abstraction and the fine-grained isolation of cloud functions.
  - 3.We predict that billing model will evolve, so that almost any application will cost much less with serverless computing.
  - The future of serverful computing will be to facilitate BaaS, applications that prove to be difficult to write on top of serverless computing will likely be offered as a part of a richer set of services from all cloud providers.
  - Serverless computing will become the default computing paradigm of the Cloud Era, largely replacing serverful computing and thereby bringing closure to the Client-Server Era.

嗯...这饼挺大的