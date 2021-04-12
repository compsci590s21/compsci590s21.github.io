---
layout: post
title: Contact Tracing Approaches
subtitle: By Omar AlSughayer and Jian Yao
tags: [contact tracing]
---

# Introduction

In the previous blog post we encountered several proposed and recent methods of contact tracing all which aim to aid in limiting the spread of the COVID-19 virus. The attentive reader might have noticed similarities between the Apple-Google approach and the Hamagen one. In contrast, TraceTogether+ had a wildly different approach. The similarities and differences observed are attributed to the approaches belonging to two different schools of thoughts existing in the realm of Contact Tracing. Those schools of thoughts predate the novel Corona virus, but the debate between them have been brought back to the forefront of the field due to the current pandemic circumstances. In this blog post, we step back from specific implementations of Contact Tracing and focus instead on the general types they might fall into. We compare and contrast the each type's strengths and weaknesses and offer some possible unique remedies. 

# Motivation
The current pandemic caused by the novel Coronavirus brought the world to a sudden and long halt. Many countries and cities implemented some form of quarantine, lockdown, curfew, or general restrictions on movement and commerce. All these methods were preventative, attempting to restrict the transmitional abilities of the virus and make easier the task of predicting who a certain covid-positive individual might have infected so they too might be quarantined and treated. Those much-needed restrictions, however, did come with a heavy impact affecting the financial and mental health of the populace negatively. Thus, when life activities partially returned to normal, the task of predicting to whom a virus might have spread was of monumental importance. That task is known in literature as Contact Tracing. 

Traditionally, contact tracing was performed manually. Doctors would interview their infected patients to get a list of all those who they had contact with recently; the doctors then can contact those individuals and request that they quarantine themselves or be tested for the infection. Automating this process promises to make it quicker and more accurate as it would roll out potential problems with humans’ fallible memory or dishonesty. 


# Schools of Thought
COVID, fortunately, is the first pandemic of this magnitude to strike since the beginning of the Digital Revolution. Unfortunately, this means that the usage of modern technology to automate Contact Tracing has no standard form as of yet. One thing near all approaches discussed has in common is using Bluetooth technology on smartphones. I.e. a reasonable assumption is made that the problem of tracing which people made contact could be reduced to the problem of which people’s phones made contacts.

When proposed protocols and methods are surveyed, two schools of thoughts can be observed: Centralized and Decentralized Contact Tracing.Some of the proposed centralized systems were [ROBERT](https://github.com/ROBERT-proximity-tracing/), [NTK](https://github.com/pepp-pt/), and [TraceTogether](https://www.tracetogether.gov.sg/). While the decentralized solutions included [DP3T](https://github.com/DP-3T/documents), [Canetti-Trachtenberg-Varia](https://arxiv.org/pdf/2003.13670.pdf), [PACTEast](https://pact.mit.edu/wp-content/uploads/2020/04/The-PACT-protocol-specification-ver-0.1.pdf), [PACT-West](https://covidsafe.cs.washington.edu/), [TCN Coalition](https://www.lfph.io/tcn-coalition/), and the [Apple-Google solution](https://covid19.apple.com/contacttracing).


# The Divide
Naturally, different factions emerged each in support of one of the two approaches. Due to the novelty of both the situation and the discussion, no consensus has been reached yet over which approach is superior. A joint statement signed by hundreds of researchers did list four privacy pillar requirements which all contact tracing systems must fulfill. However, a fair reading into the wording of these pillars does not conclude, despite scarce objection, that the opinion of those researchers was swayed in one direction or the other. Centralized or decentralized. Therefore, the need emerges for a detailed comparison between the pros and the cons of both approaches. We offer such a comparison in the following sections. 


# Formalization of Annotation
The following notation can be used for the discussion of both approaches and to formalize the difference between centralized and decentralized systems. 

An application $app$ has instances $app_U$ and $app_V$ running on the bluetooth-enabled smartphone devices of users $U$ and $V$ respectively. Instance $app_U$, just like all other instances, is periodically fed a list $L_U^{out}$ of ephemeral identifiers $e_1, e_2, \dots, e_n$ which are broadcasted in a given order. During $epoch_i$, that is time duration $i$, $app_U$ broadcasts identifier $e_i$ using the device’s bluetooth to all nearby phones. At the same time, $app_U$ is listening to broadcasts from other instances of the application on other devices. When $app_U$ receives ephemeral identifier $e_j'$ from $app_V$ it records in another list $L_U^{in}$. Both lists, $L_U^{out}$ and $L_U^{in}$ are stored with timestamps that indicate when they have been sent or received. Those lists will be cleared out periodically. As for the identifiers, they are provided by a central server in centralized systems, and generated locally in decentralized ones.

When user $U$ is diagnosed as infected in the hospital, they are provided with an activation code that allows them to upload their information to a central server in what is known as a $report$ protocol. Centralized systems would upload $L_U^{in}$ to the server while decentralized systems will upload $L_U^{out}$ instead.

In contrast, a user at any time could use the $status$ protocol to check if they have been in contact with an infected person. Decentralized $app_U$ will check its status by downloading all identifiers from infected individuals available and comparing it against the local list $L_U^{in}$ for any potential matches. As for centralized systems, they use trapdoor $\tau$ to relate pseudonym $psedo_U$ of user $U$ to all identifiers $e_i$ generated by that user’s pseudonym. Thus, to check their status on a centralized system, user  submits  to the server which in return finds all submitted identifiers  by that pseudonym and checks if they have come in contact with an infected person. 
In contrast, a user at any time could use the $status$ protocol to check if they have been in contact with an infected person. Decentralized $app_U$ will check its status by downloading all identifiers from infected individuals available and comparing it against the local list $L_U^{in}$ for any potential matches. As for centralized systems, they use trapdoor $\tau$ to relate pseudonym $psedo_U$ of user $U$ to all identifiers $e_i$ generated by that user’s pseudonym. Thus, to check their status on a centralized system, user $U$ submits $psedo_U$ to the server which in return finds all submitted identifiers $e_i$ by that pseudonym and checks if they have come in contact with an infected person. 

The following graph summarizes the differences between the two types of systems. 

![](https://i.imgur.com/V1f5RZA.png)
*Figure 1. Centralized (left) and decentralized (right) contact tracing systems when $\small V$ is diagnosed*



# Privacy Issues with Tracking People
The first concern arises with relation to trapdoor $\tau$. Anyone who gains access to $\tau$, such as the issuing authority itself, can relate all ephemeral identifiers $e_i$ to pseudonym $psedo_U$. Since the same authority could secretly have recorded the pseudonym of every user $U$ during the registration phase, it follows that they can fully track all users in lieu of having continuous access to $\tau$. 

Decentralized systems are immune to this specific attack but could still face similar consequences by an equivalent approach. When identifiers are generated locally in decentralized systems, that is done using a a temporary key $k$. All identifiers generated from the same key could be linked back to it, and therefore anyone with access to the server and $k$ could link two ephemeral identifiers to the same source $k$. Consider that every user uploads several values of $k$, each having been used for a day, when they are infected. With some data mining and social engineering, an adversary could be able to link this list of keys to a specific user. 

However, the data leakage is quite minimal, especially when compared to how much extra data mining an adversary will have to do on the side. On the other hand, total blind trust needs to be placed in a centralized authority whilst knowing they easily could automate the process of tracking people in their app. *Therefore, we find it reasonable to conclude that decentralized systems offer a smaller risk vis-a-vis tracking people than centralized systems do*. 


# Privacy Issues with Disclosing a User’s Social Graph
If the server is to be trusted and adversaries are assumed to be honest, then centralized systems seem to offer no insight into the social graph. However, opponents to centralized systems do hold the belief that comparing two infection reports $L_U^{in}$ and $L_V^{in}$ could reveal that users affiliated with these two lists have interconnected social graphs if those two lists did have some identifiers in common. It should be noted that such an attack, though valid, still requires a substantial amount of data mining to be linked back to specific users. 

On the other hand, decentralized systems could potentially provide a proof of contact between any given pair of users. Assume a malicious server which wants to confirm if user $U$ has met user $V$ after the former came with a positive case. The malicious server could provide $V$ with $U$’s full reported identifiers as well as some junk ones that do not belong to anyone. If the status of user $V$ later changed to infected, it must have been because of user $U$. Note that this attack does not apply to centralized systems, and that such an attack could be emulated by a single user. 

In short, we can conclude that both systems do in fact potentially reveal a portion of the social graph although to varying magnitudes. *Centralized systems require a lot of effort to reveal a small portion of the social graph, while with relatively less effort it provides an adversary a proof of contact between two users in a decentralized system*. Neither of the two vulnerabilities could be decisively made more severe than the other. 


# Privacy Issues with Identifying Diagnosed People
Decentralized systems seem to be particularly weak against such attacks, with several attacks mentioned in literature which could expose the identity of diagnosed individuals. The main gist of most of these attacks lies in the fact that the list of all pseudonyms of diagnosed individuals is available online for anyone to access. When the assumption that an adversary $A$ has future access to all those pseudonyms, it can out user $B$ as infected in one of several ways. For instance, $A$ could employ a *nerd attack* using an app which not only collects ephemeral identifiers but also memorizes the identity of person $B$ who emits them. $A$ can then wait for the server to confirm $B$ as an infected person. 

Centralized systems on the other hand are way less vulnerable to such attacks. Depseudonymization is still possible though. The main method discussed in most sources involves an adversary $A$ creating a dummy account through which they only come in contact with user $B$. This would certainly inform $A$ if $B$ ever became infected. However, this comes with many difficulties that restrict $A$ significantly. For once, a dummy account might not be a possibility if, for instance, the creation of an account required an untraceable activation code given by a local authority only once to an individual. Also, some systems such as ROBERT deactivate an account after a positive diagnosis. This all means that such an attack is not possible on a wide scale. 

Both centralized and decentralized show symptoms of vulnerabilities which exposed diagnosed individuals. However, the case is much more severe with decentralized systems. While minor adjustments to the registration process could prevent a wide-spread attack in centralized systems, the vulnerability is baked into the core of decentralized systems. Therefore, we conclude that centralized systems offer a great advantage over decentralized ones in protecting the identity of diagnosed people. 


# Opting option
Contact tracing needs to be used voluntarily, which creates a barrier given that most people nowadays care about privacy to varying degrees. The users of a contact tracing service should have the right to opt-in or opt-out reporting their diagnosis, as to protect those who worry for their privacy from unnecessary pressure, coercion, and societal discrimination.

A privacy-freak, grumpy user might prefer decentralized systems since there is no real opting option in centralized systems. The grumpy user would not feel happy and safe if the central server alerted or “identified” him just due to the fact that someone he has encountered has tested positive and revealed his identities to the server, especially when the user does not trust the server. In decentralized systems, the grumpy user’s identities would not be reported by a diagnosed person encountered by him. Therefore, in a decentralized system, a diagnosed, irresponsible user can have less pressure if he does not choose to report for privacy reasons, because the system cannot “identify” him as someone who has the potential to be infected. On the contrary, in a centralized system, the risk of being identified as a potentially infected person is higher, given the fact that his ephemeral identities probably have been reported by the diagnosed person he has encountered.

However, a privacy-freak rational user of decentralized might find out that the personal privacy cost to opt-in report for a decentralized system might be higher than a centralized one, and this leads to less effective decentralized contact tracing if the user has no pressure to opt-in or opt-out report. Additionally, as for a malicious privacy-freak rational user, Bob, he can use a modified app to maximize the benefit of being alerted when at risk and minimize the privacy risks. In a decentralized system, Bob can broadcast useless, dummy identifiers and report the true ones. By doing this, he can still recognize encounters who have reported their identifiers, and even if Bob chooses to report his true identifiers, the report is still useless since encounters cannot match him against the fake dummy identifiers he has broadcast. In centralized systems, Bob needs to report true ephemeral identifiers to get the benefit of being notified when he’s at risk, and if he is diagnosed, he can just choose to not report. Thus, if Bob never reports, he might not need a malicious app in centralized systems due to the small privacy cost.



# Securities Problems: False Encounter Attack
There are a few security issues at the application level, such as replay attacks, which work for centralized and decentralized systems, but decentralized systems make such attacks easier to create malicious false encounters at a larger scale. For example, an adversary $A$ could replay an identifier $e$ from $app_B$ to $app_C$ in order to forge an encounter, so that $C$ receives $e$ and believes it has encountered $B$, even though $B$ might be 1000 miles away. If malicious $A$ wants to negatively affect $C$’s normal life, he can collect ephemeral identifiers $e$ from diagnosed $B$ and replay it to $C$, so $app_C$ will then raise a (fake) alert. 

[Vaudenay](https://eprint.iacr.org/2020/531.pdf) introduced an idea to mitigate this false encounter attack: $app_B$ can send a triplet $(x, r, H(x, r), f_k(h(x, r))$ instead of sending normal $e_i$. In the triplet, $x$ can be the clock value of $B$ at sending, $r$ indicates a random number, $H$ is a hash function, and $f_k$ indicates a message authentication code associated with key $k$. When receiving the triplet, once $appC$ verifies the correctness of $h = H(x, r)$ and $x$, it can then include the $(h, r)$ pair in $L_U^{in}$. To verify $x$, we need to check if the difference between $x$ and the clock at reception is small enough, so an adversary $A$ will fail if he replays the identifiers from $B$ to $C$ at a time significantly different from the time of $B$ at sending. However, if $x$ is just a clock value, this approach cannot prevent relays from $B$ to $C$ far away from each other. We can also include the geographic position in $x$, but it causes another severe privacy problem: an adversary $A$ can tie $(x, r)$ together with $(h, r)$ pairs to secretly store the geographic information of $B$.


# False Report Attack
In centralized systems, the adversary $A$ needs to get $e$ from $C$, and then $A$ can let diagnosed $B$ report $e$ to the server in order to make $C$ receive a false alert. This is a form of false report attack. Unlike in decentralized systems, this kind of attack has a relatively smaller scale.

In decentralized systems, a false report attack can be used to create chaos or a trolling “terrorist attack”. A diagnosed attacker can attach his device to public transportations or animals that move around in a city, causing a large number of fake encounters that will trigger massive alerts for many people in the city. In centralized systems, the attacker can still try to make such trolling attacks by reporting a long list of people’s identifiers, but it can be defeated by limiting the number of identifiers allowed in the report to the server. Therefore, we can see that, again, decentralized systems can make this kind of attack a bit easier. 

# Hybrid Approaches
We now see that either centralized or decentralized systems are imperfect, and let’s see some research directions to design better, hybrid solutions introduced by [Vaudenay](https://eprint.iacr.org/2020/531.pdf).

## Centralized Architecture with Open-Access Server
This approach implements an open-access server on top of a centralized architecture to achieve a hybrid between the two systems, and it requires each client app to generate and upload their public-secret pairs of identifiers before usage of the system:

1. Each $app_U$ set up and upload a list $L_U^{out}$ of $(e_i, s_i)$ pairs anonymously to a server, where $s_i$ is a secret ephemeral identifier associated with $e_i$ 
2. $app_U$ broadcasts $e_i$ periodically, and after $e_i$ is broadcasted for the last time, it’s deleted from $L_U^{out}$ while retaining $s_i$. After receiving broadcasted $e_i$, each $app_V$ stores it in $L_V^{in}$ along with coarse time information. 
3. If $V$ is diagnosed, part of (14 days) $L_V^{in}$ will be uploaded to the server with credentials, and then the server will look up the stored pairs and publish the associated $s_i$.
4. $app_U$ regularly checks new $s_i$ from the server, and an alert will be raised if a $s_i$ matches $L_U^{out}$ .

However, there might be some vulnerabilities in this architecture. For example, if the server breaks the anonymity of the upload, it can track the user.


## Decentralized Architecture with Restricted-Access Server
Another solution is to make the at-risk status verification private by applying some sort of centralized and cryptographic techniques on top of a decentralized system. Here’s the general architecture:

1. $app_U$ set up a list of random identifiers $e_i$ and store them in  $L_U^{out}$ 
2. $app_U$ broadcasts $e_i$ periodically, and after $e_i$ is broadcasted for the last time, it’s erased from $L_U^{out}$. After receiving broadcasted $e_i$, each $app_V$ stores it in $L_V^{in}$ along with coarse time information.
3. If $V$ is diagnosed, part of (14 days) $L_V^{in}$ will be uploaded to the server with credentials, and the server will store it.
4. $app_U$ regularly and jointly runs with the server a protocol, which takes $L_U^{in}$ as input and returns the at-risk status of $U$.

In this architecture, unlike the normal decentralized systems, the server is not open-accessible, and the 4th step above is based on a 2-party protocol that takes two sets $S$ (the set of the server) and $I$ (identities or the set of the app) as input to compute the at-risk status. Since malicious users might be able to identify diagnosed clients by running the protocols many times, the server first needs to limit the number of queries for each account registered by each $app_U$. With the assumption of the limited access, we can mitigate potential leakage of the protocol. The computation of the risk can be derived from deciding whether whether $S ∩ I$ is large enough, and to compute whether  $\#(S ∩ I)$ is large enough, we can use private set intersection cardinality protocol based on [Bloom filters (linear complexity) or Flajolet-Martin sketch (logarithmic complexity)](https://eprint.iacr.org/2020/531.pdf). 


## Decentralized Architecture Reporting Received Identifiers
In this decentralized-based solution, instead of reporting the list of identifiers broadcast from $app_V$, diagnosed user $V$ reports the list of identifiers $e_i$ received and stored in $L_V^{in}$ of $app_V$. This unique approach re-randomizes the identifiers to reinforce privacy. This approach requires the re-randomized $e_i$ match to the associated secret ephemeral identifier $s_i$. For instance, if we set $e_i$ in a Diffie-Hellman form $e_i = (g^{r_i}, g^{s_i*r_i})$, and $Rerand (x, y) = (x^r, y^r)$, so we then can identify whether $y = x^{s_i}$ and match $(x, y)$ to $s_i$. This approach could be further improved since it cannot defeat Sybil attack because a malicious client can create dummy $s_i$  to figure out diagnosed users. Here’s the architecture of this approach:

1. Each $app_U$ generates a list $L_U^{out}$ of random $(e_i, s_i)$ pairs where $s_i$ is a secret ephemeral identifier associated with $e_i$ 
2. $app_U$ broadcasts $e_i$ periodically, and after $e_i$ is broadcasted for the last time, it’s deleted from $L_U^{out}$ while retaining $s_i$. Then, a few weeks later after the $e_i$ being earsed, $s_i$ is also erased. After receiving broadcasted $e_i$, each $app_V$ stores it in $L_V^{in}$ along with coarse time information. 
3. If $V$ is diagnosed, $app_V$ uploads $Rerand(e_i)$ for part of (14 days) $L_V^{in}$ to the server with credentials, and then the server stores it.
4. $app_U$ periodically checks whether new $Rerands(e_i)$ on the server match any $s_i$ in $L_U^{out}$ , and $app_U$ returns whether $U$ is at risk.

# Public-Key-Based Architecture
1. Each $app_U$ generates a list $L_U^{out}$ of $(pk_i, sk_i)$ pairs, and $pk_i = g^{sk_i}$ in a Diffie-Hellman group derived by some common $g$. Then the user posts $pk_i$ on a blockchain (public register), and stores the value of the address $e_i$ where $pk_i$ appears on the blockchain. Therefore, the list $L_U^{out}$ now contains triplets $(e_i, pk_i, sk_i)$.
2. $app_U$ broadcasts $e_i$ (the address) periodically, and after $e_i$ is broadcasted for the last time, the triplets $(e_i, pk_i, sk_i)$ is deleted from $L_U^{out}$. After receiving broadcasted $e_i$, if the triplet used by $app_V$ to broadcast is $(e’_j, pk’_j, sk’_j)$, then $(pk’_j, sk’_j, e_i)$ is stored in $L_V^{in}$ along with coarse time information. 
3. If $V$ is diagnosed, $app_V$ uploads $s_{i,j}$ for (part of) the triplet $(pk’_j, sk’_j, e_i)$ in $L_V^{in}$ to the server. Then $app_V$ gets $pk_i$ at address $e_i$ in the server and calculates $s_{i,j} = pk_i^{sk_j}$ , which is then published by the server. 
4. $app_U$ periodically checks for new reported $s_{i, j}$. Also $app_U$ derives his own list of $s_{i, j}$ and checks whether any $s_i$ matches the published ones. 

The idea of this approach is that each user’s public keys $pk_i$ are anonymously added on a public register or a blockchain, and each user broadcasts $e_i$ the address of the public key in the register, instead of broadcasting the ephemeral identifier. Then, each user can generate a shared Diffie-Hellman ephemeral key from his own secret key $s_i$. However, this approach is imperfect, and one of the issues is that a malicious client $A$ can use a dummy pair for user $B$ in order to identify $B$ when $B$ reports infection of Covid. Limiting the number of Diffie-Hellman keys that a user can register would be a possible way to mitigate this specific issue. 

# Conclusion 
It seems that the war between proponents of centralized and decentralized systems cannot be definitely won by either party. The very nature of either approach makes them more vulnerable to some attacks than others. The decision to choose which one to implement ends up being more of a social problem, deciding which kind or risk is more likely or more acceptable, as opposed to a purely scientific problem. For either approach, we offer some modifications we believe will create generally more secure structures while preserving their nature. 


# Reference
[Vaudenay, S. (2020). Centralized or Decentralized? The Contact Tracing Dilemma](https://eprint.iacr.org/2020/531). *IACR Cryptol. ePrint Arch.*, 2020, 531. 
