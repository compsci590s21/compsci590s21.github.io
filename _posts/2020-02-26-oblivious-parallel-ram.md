---
layout: post
title: Oblivious Parallel RAM
subtitle: By George Wang and Haozhan Sun
tags: [oblivious computation]
---

## Motivation
As previous blogs discussed, Oblivious RAM (ORAM) takes advantage of the property that each specific sequence of memory locations accessed during the execution of a certain program is independent of the program, so that an adversary cannot infer any additional information about the original operations other than the number of accesses by observing the access pattern. In real-life scenarios, there may exist multiple readers accessing a SQL database simultaneously to extract information, which we want to hide from an adversary. Therefore, the addition of parallelism is strongly preferable to guarantee security and efficiency. Oblivious Parallel RAM (OPRAM) achieves this notion such that it supports multiple clients simultaneously accessing a storage server in parallel, and ensures that the communication among clients shall also remain oblivious.


## Overview
In this blog post, we are looking into a special scheme of Oblivious RAM (ORAM): Oblivious Parallel RAM (OPRAM). We will first discuss Oblivious RAM (ORAM), which tries to "fool" the adversary using garbled read/write operations (to access a remote storage server or a random-access memory). Then we talk about the natural setting of Oblivious Parallel RAM (OPRAM) where $m$ parallelly access the storage server at the same time. Finally we discuss the contribution of the paper's work: 1. Having constructed the first OPRAM scheme that matches the storage and server-client communication complexities. 2. Presenting a generic transformation turning any ORAM scheme into an OPRAM scheme. 


## Introduction

The paper tries to solve the problem about the attempt to *hide access patterns* when we read from and write to a memory or storage server that is not trustworthy. This is a basic problem both in the context of software protection and secure outsourcing to a third-party storage provider.

The most simple way to "hide access patterns" is to use *Oblivious RAM* (ORAM). This technique can compile logical access sequences into "garbled ones", so that a (potentially malicious) observer cannot infer anything other than *the overall number of logical accesses*.     These garbled access sequences are considered *oblivious*. The state-of-the-art implementation of the ORAM scheme currently has a $O(log^2N)$ computation overhead, where $N$ stands for the size of storage. 

***
### Parallel Oblivious Accesses
There is an issue with existing ORAM schemes: they only support a *single client*. More importantly, they cannot handle *parallel accesses from multiple clients*. The authors then introduce the notion of *Oblivious Parallel RAM*, which compiles *synchronous parallel* logical access sequences by $m$ clients into *parallel* and garbled sequences. This, after introducing parallelism, still does not reveal any information other than the total number of logical accesses. The authors then introduced the OPRAM scheme, but pointing out that the complexity of the scheme is $\omega(log^3N)$, which is a factor of $\Omega(logN)$ higher than the state-of-the-art ORAM scheme. 

From here, the authors try to answer the question:

"*Can we design an OPRAM scheme with the same per-client efficiency as the state-of-the-art ORAM schemes?*"

The paper gives the affirmative answer by proving the following two informal theorems:

> ### Theorem 1:
> There is an OPRAM scheme with $O(log^2N)$ (amortized) server-client communication cost, and constant storage cost.

The authors did not stop here. They further think about this question: 

"*Are there any more abstract relationship between ORAM and OPRAM?*"

The answer is again YES. So they came up with a more generic theorem:

> ### Theorem 2:
> There is a generic transformation that turns any ORAM scheme into an OPRAM scheme, with an additional $O(logN)$ (amortized) server-client communication cost, and constant storage cost.

This approach is significantly different from that when the OPRAM scheme is first introduced. One key idea is the use of partitioning, i.e. each client is actually responsible for a designated portion of the server storage. This new idea makes the system no longer need much coordination, which is fantastic.

## Solutions

To address the issues above, we may want to ask what novel scheme we should have. Here is how:

> 1. First, construct an ORAM scheme, called *Subtree-ORAM*, that enables a *single* client to batch-process $m$ logical accesses at a time *in parallel*.
> 2. Second, exploit the batch-processing structure of Subtree-ORAM to adapt it to the multiple-client setting. Then, distribute its computation across multiple clients to derive the novel Subtree-OPRAM scheme.

You may have noticed here that this Subtree-ORAM is quite similar to the previously introduced Path-ORAM. Yes. Actually it is a generalization of Path-ORAM. So why do we use Path-ORAM as its cornerstone?

You may refer to the previous blog post by Andrew and Anurag about Path-ORAM, but here is a very brief review of it.

### Review of Path-ORAM 

In order to implement a storage space for N data blocks, basic Path-ORAM organizes the storage space as a complete binary tree with depth $O(logN)$. In Path-ORAM, in order to *hide access patterns*, each data block is assigned to a random path $l$, and is stored in some bucket on path $l$. After each access, the assignment is "refreshed" to a new random path $l'$. The client will use a *position map* to track the current path of each block, and also keep an additional memory for overflowing blocks, called the *stash*. Every time a logical access takes place, the following two things will happen:

> 1. **Fetching a path.** Retrieve the path $l$ currently associated with block $a$ in the position map, and find block $a$ on the path or in the local stash. Then, assign the block $a$ to a newly assigned random path $l'$ and update the position map accordingly.
> 2. **Flushing along a path.** Iterate over every block $a'$ in the fetched path $l$ and in the stash, then re-insert every block $a'$ into the lowest possible bucket on $l$ that is also
on the path assigned to $a'$ according to the aforementioned position map. Then if no suitable place is found, the block will be placed into the stash. Contents of the path then are re-encrypted when being written back to the server.

It is important to understand what happens during the two steps above. But there are also issues occurring during the above process: The generated position map can be quite large. To solve this, Path-ORAM recursively stores the position map at the server. Therefore, the overall scheme has a recursion depth of $O(logN)$, where each logical access is translated to $O(logN)$ actual accesses. 

So the overall communication overhead for Path-ORAM becomes $O(log^2N)$. The overall storage complexity at the server would be $O(N)$ despite the recursion.

### Review of Subtree-ORAM

Having read until here you may want to ask: we have seen how to design a scheme in a recursive manner, can we also have a non-recursive scheme? Yes, Subtree-ORAM is the answer here.

A brief review of what Subtree-ORAM does here:
Retrieve a subtree of $m$ paths, i.e. for every $m$ logical accesses to blocks $a_1, a_2, \dots, a_m$, we can:

> 1. **Fetch subtree.** Retrieve the subtree that is composed of the paths assigned to the $m$ blocks, and then find the blocks of interest in the subtree or in the stash (also possibly update their values).
> 2. **Path-by-path flushing.** Execute the flushing procedure from Path-ORAM on the $m$ paths in the subtree, with each access $a_i$ assigned to a new random path.

The Subtree-ORAM seems fantastic at first glance, but again, there are two significant issues with it. First, if the $m$ accesses are not *all distinct*, the accesses will not be oblivious, because the same path would be retrieved multiple times. Second, repeating the flushing procedure $m$ times is actually sequential. To overcome the flushing issue, a new manner of flushing can be introduced:

> 2. **Subtree flushing.** Iterate over every block in the subtree and in the stash. Then replace each block into the lowest node in the entire subtree that is still on its assigned path, and not yet full. In this case, the order to process the blocks can be arbitrary, and the process can also be parallelized.

### From the Basics to Subtree-OPRAM

The final goal of this idea is to create an interactive protocol that can make $m$ clients access blocks ($a_1, a_2, \dots, a_m$) in parallel. Actually, we can think of Subtre-OPRAM to be a collectively emulated series of single Subtree-ORAM.

An example here can be that we have $m = 2^l$, then we remove the top $l$ levels, turning the tree into a forest of $m$ trees $T_1, \dots, T_m$. Each client $C_i$ manages the read/write operations of $T_i$, and all blocks assigned to $T_i$ that do not fit in one of the buckets on the server would remain in a local stash managed locally by its corresponding client $C_i$. 

In later sections, we will show how exactly this works, and how the $O(log^2N)$ overhead of recursive Subtree-OPRAM and worst-case overhead $O((log^2N) \cdot loglogN)$ are derived.

### The Generic Transformation

The authors further try to abstract a more generic transformation that can convert *any* ORAM scheme into an OPRAM protocol. 

The initial inspiration comes from partition-based ORAM. The general idea is to split the server storage into $m$ partitions, with each storing $N/m$ blocks. Then let the $m$ clients run each a copy of the basic ORAM algorithm. Then, as the procedure mentioned above, when $m$ clients try to request the $m$ blocks in parallel, the clients will simply find the respective partitions containing these blocks, and let the corresponding clients retrieve the desired blocks, then delete them from their partitions. In this manner, the *access pattern* is *oblivious* since all patterns are random, and the basic ORAM scheme ensures that retrieving blocks from each partition is done *obliviously*.

As for overhead analysis, the non-recursive version has the same amortized computation and communication overhead as the underlying ORAM scheme. But for the final OPRAM scheme, the authors applied recursive techniques to outsource the partition map to the server.


## Subtree-OPRAM
### Settings
We now move to generalizing the Subtree-ORAM scheme to $m$ clients accessing $m$ blocks in parallel. As mentioned, this has a lot of practical applications in real-life scenarios. We want both the access pattern and the inter-client communications to be oblivious. We further assume that the $m$ blocks the clients are requesting are distinct. Formally, there exist clients $c_1, c_2, \cdots c_m$, which are simultaneously requesting access to blocks $b_1, b_2, \cdots b_m$, all of which are distinct. For simplicity, we assume $m$ is a power of 2.

### The Subtree-OPRAM Scheme
As the Subtree-OPRAM Scheme is largely based on the Subtree-ORAM Scheme, we first draw some comparisons among them. We can consider Subtree-OPRAM essentially multiple clients acting together on a collection of Tree ORAM collaboratively that mimics what a single client would do. This is achieved by rigorous inter-client communication, which we shall introduce in the succeeding section. Writing the requested block back without revealing additional information is also a necessary property of the scheme, which also follows closely to the Subtree-ORAM scheme.

The Subtree-OPRAM Scheme is prepared as follows: We remove $\log_{2}{m}$ layers from the ORAM tree previously introduced, which renders a forest of $m$ complete binary trees $T_1, T_2, \cdots T_m$. We let client $C_i$ to manage all requests to tree $T_i$. We identify whether any path l belongs to $T_i$ by evaluating whether the leaf of l belongs to $T_i$. The clients keep track of the path l by referring to a position map of size $O(NlogN)$. In addition, each client similarly maintains a local stash to store any overflowing block. It also holds that the space of the local stash is bounded by $\omega(log \lambda) + O(log m)$.

The execution of Subtree-OPRAM is as follows, specific subprotocols will be introduced in the succeeding section.

1. Assign $m$ clients such that they are representative for accesing block $b_i$ respectively. The primary goal for this step is to select block representatives for each tree in the forest. The *oblivious elect* protocol in this step has fixed communication pattern.
2. The execution of the scheme starts by $m$ clients finding the path of their corresponding block $b_i$ to access via the position map. The inter-client communication in this step depends on random path
3. If the block happens to be in $T_i$, then $C_i$ can simply access that block. Otherwise, $C_i$ initialzes a fake read, and sends an request to $C_j$ who manages $T_j$ where block $b_i$ is stored. The requests (paths read) in this step are executed randomly.
4. $C_j$ will implement Subtree-ORAM to execute all requests and send the retrieved information back. The *oblivious cast* protocol in this step has fixed communication pattern.
5. $C_i$ then remaps $b_i$ to a new path L', and sends the request of writing the new path to $C_j$. The *oblivious route* protocol in this step has fixed communication pattern.
6. Finally, each client runs the Subtree Flushing Procedure on the retrieved subtree and stash, before writing the subtree back. The remap protocol in this step is random.

The obliviousness is guaranteed by the randomness, fake read, and the remapping subprotocol. Namely, the paths of read/write from or into the server are all independent and random for each iteration; The inter-client communications are managed using oblivious subprotocols (i.e. OblivAgg, etc. introduced in the succeeding section) or depends on random paths; Finally, the remapped paths to blocks accessed in each iteration are hidden using oblivious route.

### Space Optimization and Persisting Issues
As a shared global map may be undesirable, we introduce space optimizations using recursion to eliminate the global position map. This can be achieved by storing the position map to the server in the trees recursively, which is reduced to $O(1)$ space and can be stored in any client's local storage. In addition, to prevent errors induced by write-collision, we need to ensure that for each address, only the write request with a minimal index will be executed. We can then execute the rest of the preceding Subtree-OPRAM Scheme. The protocol can thus reach the same computation and communication overhead as the ORAM scheme. 

Some persisting issues include treating concurrent access of the same blocks by multiple clients. In general parallel system settings, colliding requests can be addressed by introducing a lock to critical sections, but to ensure obliviousness, additional work needs to be done.

<!---
George
-->
## Oblivious Inter-Client Communication Interfaces
To ensure the security of inter-client communications, we introduce more protocols, which can be applied to the preceding section OPRAM. We provide some overview of the protocols needed:

**Oblivious Aggregation (OblivAgg)**: OblivAgg allows clients to aggregate their requests, parametrized by an aggregation function (agg). 

For such property to hold, we need agg to be associative, that is, the outcome is independent of the order of agg is applied to the messages.

**Oblivious Routing (OblivRoute)**: For the $m$ party involved, OblivRoute allows each party to send a message to another party.

**Oblivious Election (OblivElect)**: For each unique address in the $m$ requests, OblivElect allows the $m$ parties to elect an entity as the representative of the party. 

This is a stronger notion than Oblivious Aggregation. The outcome of this function is the smallest identity requesting to write to the address. If such an entity does not exist, then the smallest identity that wants to read from the address will be selected.

**Oblivious Multicasting (OblivMCast)**: Allows a subset of clients (senders) to multicast values to others (receivers). This is essentially the reversal of OblivElect.

We can now formulate the preceding section more rigorously. 
1. The $m$ clients first run OblivElect to select block representatives. 
2. Then each client determines the path of block it wants to request, and executes Real Read or Fake Read respectively. 
3. Each client $C_i$ then executes read, and send an encrypted message to some client $C_j$ if it manages the desired block of $C_i$. 
4. $C_j$ will then decrypt the request message and execute correspondingly, and utilizes OblivMCast to send the messages back. 
5. $C_i$ remaps the block using the OblivRoute subprotocol. 
6. Finally, each client flushes the retrieved subtree and stash, before writing the subtree back. 
To eliminate the global position map, one would run OblivAgg at the first stage. The rest of the protocol should be identical to a non-recursive Subtree-OPRAM.

## Conclusion
To summarize, the paper introduces an OPRAM scheme, Subtree-ORAM, inspired by Path ORAM, with the same per client efficiency as SOTA ORAM schemes. The authors first give a brief overview of Path-ORAM, which serves a single client with one access to the server. Then, they present the concept of Subtree-ORAM, which supports a single client to batch-processes $m$ logical accesses in parallel. They generalize this notion further by presenting Subtree-OPRAM, which supports concurrent access of $m$ blocks by $m$ clients. The paper also consists of a final section of generalizing the idea of Subtree-OPRAM to a generic transformation from any ORAM to a concurrent OPRAM scheme. As there are currently few OPRAM schemes available, we shall expect, with the help of such generalization, there would be more efficient OPRAM schemes being proposed in the future.

## References

[Oblivious Parallel RAM: Improved Efficiency and Generic Constructions](https://eprint.iacr.org/2015/1053.pdf)
