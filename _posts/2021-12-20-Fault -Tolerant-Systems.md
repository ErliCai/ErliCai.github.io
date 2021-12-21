---
layout: post
title: Fault Tolerant Systems
---

#### Name: Erli Cai
#### Papers: 
- Castro, Miguel, and Barbara Liskov. "[Practical byzantine fault tolerance.](https:/www.cs.cmu.edu/~15712/papers/castro99.pdf)" OSDI. Vol. 99. No. 1999. 1999.
- Scales, Daniel J., Mike Nelson, and Ganesh Venkitachalam. "[The design of a practical system for fault-tolerant virtual machines.](https://pdos.csail.mit.edu/6.824/papers/vm-ft.pdf)" ACM SIGOPS Operating Systems Review 44.4 (2010): 30-39.
- Schneider, Fred B. "[Implementing fault-tolerant services using the state machine approach: A tutorial.](https://www.cs.cmu.edu/~15712/papers/schneider90.pdf)" ACM Computing Surveys (CSUR) 22.4 (1990): 299-319.

### Overview

With the growth of size and complexity of computer software, there are also an increasing number of software errors. Using a single machine to hold service is always good enough since the resulting service can only be as fault-tolerant as the processor executing that machine. A common way to tackle this problem is by introducing replicas of the primary machine that runs on separate processors and therefore fail independently.

Generally, there are two kinds of failures, The Byzantine failures and the Fail-stop failures. A Fail-stop failure is the machine changes to a state that permits other components to detect that a failure has occurred and then stops (e.g. the machine shuts down).
And a Byzantine failure is when software exhibit arbitrary and malicious behaviour. Byzantine failures are usually more disruptive since there is usually no evidence when such a  failure occurs in practice.

The three papers we chose are all dedicated to implementing a fault-tolerant system. However, these papers have made different assumptions on types of failures, bandwidth, storage and output requirement, which leads to different design choices. We will have a closer look at these three papers and analyse their choices.

For each paper, we will focus on discussing the following element:
- approach to the fault-tolerant system
- what are the assumptions
- How is design choices made against these assumptions

### First paper: 
**Implementing fault-tolerant services using the state machine approach: A tutorial.**

Compared with the other two papers, this paper focuses more on the theoretical side of the discussion and looks at a larger scale than the other two papers. It discusses the solution for faulty severs, faulty output device and also faulty client, whereas the other two paper only focuses on a faulty server.

In this paper, the author proposes using state machines to solve the problem and assumes we are working with both Byzantine failures and Fail-stop failures.

For a faulty server, the general idea is to replicate the state machine and run a replica on each of the processors in the distributed system.
This idea is based on the assumption that if each replica starts with the same initial state and executes the same request in the same order, then each replica should produce the same result. Thus, if each machine is failing independently, then by combining the output of the state machine replicas, we can obtain the output for the fault-tolerant state machine.


