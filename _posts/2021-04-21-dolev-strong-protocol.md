---
layout: post
title: Dolev-Strong protocol and FLM lower bound
subtitle: By Qiaoyi Fang and Yuqing Zhang
tags: [consensus]
---

# Introduction

The Byzantine Agreement problem can be defined as follows: \\(n\\) processors are connected with a network where every processor can communicate with any other connected processor. However, a number of faulty processors that exhibit arbitrary behavior may exist in the network. There are at most \\(t\\) faulty processors in the network. Initially, each processor has an input value \\(v_i \in V\\). After the executing a protocol, every non-faulty processor then is able to decising on an output value \\(v'_i\\). For a _Byzantine agreement_ protocol, there are three properties that it must satisfy: 
- **Termination** All honest processors must eventually decide on a value and terminate.
- **Agreement** All honest processors decide on the same value; i.e, for any two non-faulty processors \\(i\\),\\(j\\), their outputs \\(v'_i\\) = \\(v'_j\\).
- **Validity** Assume the initial inputs to all honest processors are the same, then all non-faulty processors reach agreement on that value; i.e. if \\(v_i=v\\) for any honest processor \\(i\\), then \\(v'_i=v\\).

The _Byzantine agreement_ problem remains to be an important research question with wide applications. For example, it is a foundamental primitive used within multi-party computation algorithms or distributed systems where a group might be exposed in the presence of a certain fraction of malicious parties. In the following two sections, we will discuss a more specific scenario _Broadcast_ and a solution _Dolev-strong protocol_ to the broadcast problem when an authentification scheme is established in the setup.

# Impossibility of Byzantine Agreement
The FLM lower bound states that Byzantine agreeement is imporssible if the number of faulty processors $f$ is greater than the 1/3 of the total processors in the network $n$. In order to prove the lower bound, we construct a network with three parties $A$, $B$, and $C$. The goal of this proof is to show that when $B$ and $C$ blame each other for lying and are unable to prove the correctness, A cannot decide on who to trust and therefore unable to decide on a value between $B$ and $C$. 

**Theorem:** It is impossible to solve synchronous agreement in a plain authenticated channels model if $f\geq n/3$.

Assume that Byzantine agreement can be achieved between $A$, $B$, and $C$. We can construct three worlds where in each world, an adversary simulates a world of four players and communicate with non-faulty processors.

**World 1:** 
![](https://i.imgur.com/Hv13Nq8.png)
In world one, $C$ is the adversary and generated $A'$, $B'$, and $C'$ for communication. The correct processors($A$, $B$) initiated with the same value 1; however, $C'$, an instance of $C$, claims to $A$ that a value of 0 is received from $B'$ and then sent to $A$. Despite the effort to perturb the network, $A$ and $B$ are still able to commit to 1 according to validity property since all correct processors are initiated with the same value.

**World 2:**
![](https://i.imgur.com/wxvMAnJ.png)
In world two, $A$ is the adversary and simulated three instances $A'$, $B'$ and $C'$. Similar to world 1, $B$ and $C$ are able to commit to 0 because validity holds.

**World 3:**
![](https://i.imgur.com/cPa49N4.png)
World three is similar to world 1 and world 2 except that the non-faulty processors are not initiated with the same value. However, $A$ and $C$ cannot decide on the same value in this case. If we observe the network from $A$'s perspective, we can see that world 3 is identical to world 1 in which $C$ and $C'$ are switched and $A$ is not able to distinguish. Similarly, $C$'s perception of world 2 and world 3 are the same. Since $A$ commited to 1 in world 1 and $C$ commited to 0 in world 2, they holds the same commited value. Then, the agreement property does not hold in world 3. 

# Broadcast
Similar to the definition we provided above, the _Broadcast_ problem can be defined as follows and describes the basic functionality of broadcasting: \\(n\\) processors are connected with a point-to-point network, in which a distinguished party acts as the \\(sender\\) with a value \\(v_s\\) at the beginning. All nodes will receive \\(v_s\\) and, after the execution of the protocol, will all reach agreement on the \\(v_s\\), or \\(v'_i = v_s\\). Again, we assume up to \\(t\\) faulty processors in the system. In addition to the same termination and agreement properties, a _broadcast_ protocol should also satisfies the following:
- __Agreement__: All honest processors reach agreement on the same value, i.e. \\(v'_i = v'_j\\) for all honest \\(i,j\\).
- __Validity__: If the _sender_ is honest, then all honest processors can agree on \\(v_s\\), i.e. for all honest players \\(i\\), \\(v'_i = v_s\\)if the _sender_ is honest.

<!-- Could add the lower bound part here? -->
The lower bound is less strict for broadcast problem. We claim that Byzantine agreement is trivial if processors are able to broadcast and the number of faulty processors $f$ is less than half of the total number of processors in the netowrk $n$. The proof is very intuitive: if every processor is able to broadcast a value to all other processors in the network, correct processors only need to decide on the majority value. Therefore, there is an equivalence between broadcast and Byzantine agreement. Based on the equivalance and the impossibility of Byzantine agreement, we can conclude that：

**Theorem:** Broadcast(or Byzantine agreement) is possible iff $t<n/3$.

The equivalence between broadcast and agreement proves that broadcast is possible if $t<n/2$ and hence possible if $t<n/3$. Impossibility of Byzantine agreement is used to show that Broadcast is possible only if $t<n/3$. In another word, broadcast(and hence Byzantine agreement) is not achievable when the number of faulty processors $f\geq n/3$. Recall that in the three worlds containing parties $A$, $B$ and $C$, Byzantine agreement cannot be reached in world 3 since world 1 and world 3 are identical in $A$'s perspective. Through broadcast, correct processors are observing the same information as in Byzantine agreement. Hence, agreement cannot be reached in world 3 as well. 
# Authenticated Broadcast
In this setting, we assume the existence of some authentification technique in the initial setup that prevents faulty processors from undetectably modifying the message contents. One can assume a public-key interfrastructure has been established that every processor has a pair of public and secret keys and knows the public key of all other processors. Under this setting, any computationally-bounded adversary cannot forge signatures on arbitary messages provided with the public keys of honest processors. Having defined the authenticated broadcast scenario, we now introduce the _Dolev-Strong protocol_. 

## Notations
Without loss of generality, we assume that processor 1 is the _sender_. We then introduce some notations used in the protocol:
- _Authenticated message_ \\(m\\): a message received by processor _i_ in round k is called (v, k)-authentific for _i_ if it has the form \\((v_s, p_1, s_1, ..., p_k, s_k)\\), where \\(p_1\\) = 1 as the _sender_ and all \\(p_k\\) are different and don't include processor _i_. In addition, for ever \\(1 < i \leq k\\), the signature \\(s_i\\) is valid when it signs \\((v_s, p_1, s_1, ..., p_{i-1}, s_{i-1})\\). Effectively, a _(v, k)-authentific_ message is a signature chain of size _k_ that contains the path that the message has travelled.
- _Decision function_ \\(F\\): is a function that takes the received messages of the processor \\(i\\) and outputs the decided values \\(v'_i\\) with a symbol 0 representing "_sender fault_".

## Protocol
The protocol proceeds as follows:
__Round 1__:
```{r, tidy=FALSE, eval=FALSE, highlight=FALSE}
// Sender (processor 1) with input value v_s
Broadcast (v_s, 1, s_1) to all replicas, s_1 = sign(v_s, 1)
```
__Round 2 - t__:
```{r, tidy=FALSE, eval=FALSE, highlight=FALSE}
// Processor i
Round k: 
if processor i has sent less than 2 messages: 
    if i receives a (v, k-1)-authentic message in the k-1 round and v in V:
        m is (v, p_1, s_1, ..., p_{k-1}, s_{k-1})
        s_k = sign(m, i)
        create new message (v, p_1, s_1, ..., p_{k-1}, s_{k-1}, i, s_k)
        broadcast it all processors not included in the message
```
__Round t+1 Decision Rule__:
```{r, tidy=FALSE, eval=FALSE, highlight=FALSE}
// Processor i
if processor i has received authenticated messages of different values:
    it outputs "sender fault": v'_i = 0
it has never received any authenticated message:
    it outputs "sender fault": v'_i = 0
if processor i has only received authenticated message of one value v
    it output the value: v'_i = v
```
When the processor outputs "sender fault", it indicates that the _sender_ must be dishonest: either it failed to send an authentic message or send multiple of different values in the first round. 

## Validity
 For correctness, if the _sender_ is honest with initial value \\(v_s\\), then every processor would receive a \\((v_s, 1)\\)-authentic message and would never receive a \\((v, k)\\) for which \\(v \neq v_s\\). As in order for a forged \\((v, k)\\) to be authenticated by a honest processor, the adversary must forge the siganture of the _sender_. Due the unforgeability of signatures, it is impossible. Therefore, the protocol satisfies the correctness property.

## Agreement
For agreement, observe that if an honest processor _i_ receives a \\((v-k)\\)-authentic message at round \\(k\\), there are two possible cases:
- If \\(k \leq t\\), by the design of the protocol, every other honest processors will have received a \\((v, k')\\)-authentic message for \\(k' \leq k+1\\) in the next round. Among them, k processors must have already received a \\((v, k')\\)-authentic message for \\( k' < k\\), as otherwise the message would not have contained their signatures in the \\((v,k)\\)-authentic message received by processor _i_. If a process _j_ has not received it, _j_ will receive a \\((v, k+1)\\)-authentic message from _i_ in the next round. 
- If \\(k=t+1\\) and processor _i_ received a \\(v' \leq k\\)-authentic message in the final round and \\(v' \neq v \\). The first _t_ processors on the list of signatures must be faulty and all other honest processors must have also received the message in the same round, as we assume up to _t_ faulties and one honest processor must have signed it and broadcasted to any other honest processors. 

By the design of protocol, if any honest processor has received a \\((v, k)\\)-authentic message, every other honest processors will receive a \\((v, k')\\)-authentic message, and thus, the decision function F will output the same value for them. Therefore, the agreement holds. 
## Total Messages:
\\(O(e)=O(n^2)\\), as by the design of algorithm, each coorect processor sends at most two messages over each other.


# References

https://decentralizedthoughts.github.io/2019-12-22-dolev-strong/
https://decentralizedthoughts.github.io/2019-08-02-byzantine-agreement-is-impossible-for-\$n-slash-leq-3-f\$-is-the-adversary-can-easily-simulate/
https://groups.csail.mit.edu/tds/papers/Lynch/FischerLynchMerritt-dc.pdf
https://epubs.siam.org/doi/pdf/10.1137/0212045
http://www.wisdom.weizmann.ac.il/~oded/foc-vol2.html
http://www.cs.umd.edu/~jkatz/gradcrypto2/NOTES/lecture26.pdf



