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


For faulty output devices and faulty clients, one possible strategy will always be acquiring more machines. And for a faulty client, we could also do defensive programming to prevent malicious clients.

### Second paper: 
[**The design of a practical system for fault-tolerant virtual machines.**](https://pdos.csail.mit.edu/6.824/papers/vm-ft.pdf)

In this paper, the discussion are based on a fault-tolerant virtual machine implemented by the author. It chooses a primary/backup approach. where a backup server is always available to take over if the primary server fails. 

![_config.yml]({{ site.baseurl }}/images/Basic-FT-Config.png)


This paper also makes the assumption that the servers are deterministic state machines that can be kept in sync by starting from the same initial state and receiving the same input requests in the same order. On top of that, his paper also assumes that we only consider the fail-stop failure. There are several ways to achieve that, one would be shipping changes to all states of the primary to the backup, but this approach would take significant bandwidth. Therefore, the author chooses to pass only the log entries to the backup machine. 

There are several challenges faced by this design, the primary challenge is that not all event that happens in a virtual machine is deterministic (e.g. virtual interrupts or reading the clock cycle). Extra design is needed to correctly capture all these non-determinisms. The second challenge is that when the primary machine is dead, the backup machine needs to consume all its log before it "come alive", which could take a very long time. The author chooses to let the primary machine wait for the backup when it's too far ahead.

What lacks in this paper: Although the performance of this system is quite good, the extra design for backing up the system adds less than 10% overhead, we have to be aware that there are lots of restrictions on this system 1. it only works on the uni-processor machine. 2. there is no way to deal with Byzantine failure.

### Third paper: 
[**Practical byzantine fault tolerance**](https:/www.cs.cmu.edu/~15712/papers/castro99.pdf)

This paper focuses more on the hard stone in the area, the Byzantine failure. I can see a prototype for what is the well-known Paxos algorithm (an algorithm that solves the consensus problem) in this paper.

To solve the byzantine failure, the system needs at least 3n+1 machines assuming n of them are faulty. This design does not rely on synchrony for safety and this makes it less vulnerable to malicious attacks. This design also offers a guarantee for liveness, which means clients eventually receive replies to their requests.

When the primary, receives a client request, it starts a three-phase protocol to atomically multicast the request to the replicas. The three phases are pre-prepare, prepare, and commit. The pre-prepare and prepare phases are used to totally order requests even when the primary is faulty. 

![_config.yml]({{ site.baseurl }}/images/byzantine.png)

What lacks in this paper: This paper generally provide a good solution to the byzantine problem. The only problem is that it takes too many machines to achieve this guarantee and it lacks some availability.










