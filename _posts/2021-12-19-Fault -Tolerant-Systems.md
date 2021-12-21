---
layout: post
title: 2021-12-19-FaultTolerance
---

#### Name: Erli Cai
#### Papers: 
- Castro, Miguel, and Barbara Liskov. "[Practical byzantine fault tolerance.](https:/www.cs.cmu.edu/~15712/papers/castro99.pdf)" OSDI. Vol. 99. No. 1999. 1999.
- Scales, Daniel J., Mike Nelson, and Ganesh Venkitachalam. "[The design of a practical system for fault-tolerant virtual machines.](https://pdos.csail.mit.edu/6.824/papers/vm-ft.pdf)" ACM SIGOPS Operating Systems Review 44.4 (2010): 30-39.
- Schneider, Fred B. "[Implementing fault-tolerant services using the state machine approach: A tutorial.](https://www.cs.cmu.edu/~15712/papers/schneider90.pdf)" ACM Computing Surveys (CSUR) 22.4 (1990): 299-319.


### Overview

With the growth of size and complexity of computer software, there are also an increasing number of software errors. Using a single machine to hold service is always good enough since the resulting service can only be as fault-tolerant as the processor executing that machine. A common way to tackle this problem is by introducing replicas of the primary machine that runs on separate processors and therefore fail independently.

These three papers are all dedicated to implementing a fault-tolerant system. However, these papers have made different assumptions on types of failures, bandwidth, storage and output requirement, which leads to different design choices. We will have a closer look at these three papers and analyse their choices.




First paper: **Implementing fault-tolerant services using the state machine approach: A tutorial.**
