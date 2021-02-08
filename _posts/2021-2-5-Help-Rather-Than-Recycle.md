---
layout: article
title: Help Rather Than Recycle
tags: Papers
---
Help Rather Than Recycle: Alleviating Cold Startup in Serverless Computing with Inter-function Container Sharing
## 1. Introduction
- Many efforts have been devoted to speeding up the container cold startup:
  - Pre-warm startup creates containers and runtime in advance.
  - Restore-based methods recover containers from previously-stored container images.
- Goal: borrow the ***excessive*** warm containers of some functions(these containers are currently idle but kept warm for warm startup), to help functions that tend to experience many cold startups.
- ***Pagurus***: a container management system that minimizes the container cold startup through adaptive inter-function container sharing.
- Main contributions: 
  - An inter-function sharing mechanism that alleviates cold startup with security guarantee. Zygote containers. => Safe container sharing.
  - A similarity-based container image re-packing policy. => Reduce the number of packages to be installed.
  - A cluster-level function balancing mechanism. The functions are balanced based on their loads and similarities in terms of packages.
## 2. Background and Related Work
- Existing works mainly focus on seeking more lightweight virtualization technologies to pursue lower overhead, or optimizing pre-warm strategies to provide more accurate prediction models and less initialization cost.
- This paper tries to leverage the containers before timeout across different functions to work collaboratively.
## 3. Motivation
### 3.1 Existence of Excessive Warm Containers
- The number of idle containers often increases when the query load drops => Many containers are prepared and invoked to serve the high load, and they become inactive when the load drops.
- The time that idle containers happen and the time that cold startup happens are similar. => Could leverage the idle containers of some functions to help other functions.
### 3.2 Challenges of Inter-Function Sharing
![avatar](https://raw.githubusercontent.com/ziliuziliu/ziliuziliu.github.io/master/_posts/pic/2021_2_6_1.png)

- Challenges
  - 1.Loads of functions change dynamically: the function itself may suffer from QoS violation if its load increase suddenly.
  - 2.The containers for different functions install different software packages. However, functions tend to share a high proportion of packages.
  - 3.Sharing a function's container to another function may hurt the security. Should exclude all the original function code/data.
  - 4.Multiple functions have idle containers, and multiple functions suffer from cold startup. So which to which?
## 4. Design of Pagurus
![avatar](https://raw.githubusercontent.com/ziliuziliu/ziliuziliu.github.io/master/_posts/pic/2021_2_6_2.png)

- An intra-function container manager(manage and classify different containers), an inter-function container scheduler(borrow which to which? analyze the similarities), a sharing-aware function balancer(routes functions across nodes, considering load balance and similarites of required packages).
- Idle containers => Zygote containers that include other function's packages. => Helper container, forked from the zygote container, with code loaded.
- Three plugins: re-packing operator, zygote fork and code copy.
## 5. Intra-Function Container Management
### 5.1 Identifying Idle Private Containers
...some discriminative functions...
### 5.2 Generating Zygote Containers
- Generating a zygote container image once a private container is identified to be idle...

![avatar](https://raw.githubusercontent.com/ziliuziliu/ziliuziliu.github.io/master/_posts/pic/2021_2_8_1.png)

- Generating zygote container.

![avatar](https://raw.githubusercontent.com/ziliuziliu/ziliuziliu.github.io/master/_posts/pic/2021_2_8_2.png)

### 5.3 Recycling Containers in Different Pools
- Shortest timeout period for helper containers, since they're borrowed from other functions, we aim to recycle them as soon as possible.
- Longest timeout period for the zygote containers, since they can serve queries of multiple functions.
## 6. Inter-Function Container Scheduling
### 6.1 Similarity-based Re-packing
- Use cosine distance to calculate the similarity between zygote function and to-be-helped functions. One-hot-per-package.
- Select the functions with the top k(5) highest similarity values as to-be-helped functions on the same node.
- Higher k brings longer re-packing time and larger image size, while lower k makes zygote container inefficient when sharing in the system.
- However, since re-packing phase occurs before cold startup of other functions, these overhead can be hidden and even unperceived.
### 6.2 Forking Helper Containers from Zygote Containers
- Zygote fork: safe copy of the zygote container for to-be-helped functions.
- Code copy: load the function code into the bond shared memory to specialize the zygote container and turn it into a helper container. ***Code copy both for to-be-helped functions or own function code.***

![avatar](https://raw.githubusercontent.com/ziliuziliu/ziliuziliu.github.io/master/_posts/pic/2021_2_8_3.png)

## 7. Sharing-aware Function Balancing
- The policy considers both load balance and package similarities between functions on each node.
- A redirection period: for function fK, if all the functions on the node do not have zygote container for fK, and there is no warm container in its 3 container pools for time Tredir, the redirection is triggered.
- Redirection: find the node whose resource utilization is below some threshold, and have the most zygote containers.
## 8. Evaluation of Pagurus
### 8.1 Experimental Setup
### 8.2 Reducing Tail Latency
### 8.3 Probability of Alleviating Cold Startup
### 8.4 Effectiveness of Inter-function Sharing
### 8.5 Integrating with Orthogonal Techniques
### 8.6 Effectiveness of the Function Balancer
### 8.7 Overheads of Pagurus
## 9. Conclusion
- ***Alleviates the cold container startup by 57.8% on average.***