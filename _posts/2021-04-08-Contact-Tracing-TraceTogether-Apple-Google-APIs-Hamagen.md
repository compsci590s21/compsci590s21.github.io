---
layout: post
title: Contact Tracing, TraceTogether + Apple-Google APIs, Hamagen
subtitle: By George Wang and Qiulin Li
tags: [Contact Tracing]
---

## Introduction
Contact tracing is a common practice in the public health space. As COVID cases surged since the start of 2020, contact tracing has been considered a promising way to limit the spread of COVID. In contrast to past public health crisis like SARS, it is now more tangible to utilize technology to address such issue, and hence various schemes were proposed over the past year. We will introduce some of those proposals in this blog.

### Contact Tracing
Contact tracing which was historically done manually and hence suffered from several major concerns. While contact tracing can be done manually, it requires a lot of work and is vulnerable to errors accidentally made by human. Such laborious task also make it impossible to process the data in a reasonable time, or at all, were there hundreds of thousands of new cases per day, which still happens in the US. In addition, people that might have been exposed to such disease needs to be notified within a reasonable time window and quarantined if necessary to prevent her from spreading the disease to other people. Therefore, doing contact tracing manually would require additional steps to locate and notify exposed contacts, which may result in them spreading the disease to more people before appropriate steps have been taken. Therefore, digitalizing such process would increase the effectiveness of contact tracing drastically. 

### Privacy Concerns
As contact tracing is a promising technique to address the current COVID crises, several concerns need to be eliminated were an automated system is to be deployed to the general public. Apart from hardware requirements, a major concern is the contacts' privacy. If the system or the information transmitted is insecure, an adversary may be capable of exploiting the system and collect the users' information. A secure and decentralize system would be hence be preferable. 

## Apple-Google Contact Tracing API
The Apple-Google Contact Tracing API was proposed by Apple and Google to reduce the spread of COVID that does not require user identity or GPS tracking. This decentralized protocol is primarily based on bluetooth and belongs to a family of protocols called **DP-3T (Decentralized Privacy-Preserving Proximity Tracing) protocol**. 

### The Contact Tracing Protocol
For any user that opt-in the protocol, the Exposure Notifications System will generate a random ID using the function **ENIntervalNumber**, which returns a timestamp for each 10min window by calculating $ENIntervalNumbe \leftarrow \frac{Timestamp}{60 \times 10}$

The first temporary exposure key (TEK) is hence generated as follows $i \leftarrow floor(\frac{ENIntervalNumber}{{TEKRollingPeriod}}) \times TEKRollingPeriod$, where **TEKRollingPeriod** is the duration which a TEK is valid. The device hence generates the TEK using the **CRNG** function, a cryptographic random number generator. A new key is generated at the end of each **TEKRollingPeriod**. The total number of keys stored in a device is therefore $ceil(\frac{144 \times 14}{TEKRollingPeriod}) = 14$ in this case. With limited number of keys, each with size $16$ bytes, the amount of storage needed for both the server and the client is reasonable. 

<div style="text-align:center"><img src=https://i.imgur.com/vB58Nqn.png />Protocol Schematic Representation</div>

When broadcasting through Bluetooth, **rolling proximity identifiers (RPI)** are used, which are generated using a **rolling proximity identifier key (RPIK)**. The RPIK is generated using the **HKDF** function, and after applying $AES_{128}$, generates a RPI for the moment the roll occurs, and for a specific valid TEK. Such process reduces the probability of hash collisions, which limits the probability of false-positives. Additional metadata are also encyrpted using the $AES_{128}$ with a key derived from the TEK. 

### Positive Diagnosis
#### Reporting
When a user tests positive, her TEKs and the corresponding **ENIntervalNumber** will be uploaded to the server. These TEKs are referrred as **diagnosis keys**. The server will aggregate the diagnosis keys from all self reported clinets and distribute them to all users. For those who have never tested positive, their TEKs will never be uploaded to the server.

#### Fetching
All clients periodically fetchs the list of new diagnosis keys from the server. With those information, each clients can determine the sequence of RPI that users who tested positive broadcasted. The client will proceed to match each identifier stored in her log, with a $\pm2$ hours tolerance window. 


### Privacy Concerns
Overall, we observe a significantly better privacy protection of this contact tracing protocol than centralized ones. At OS level, the key scheudle is defined by the operating system components and fixed, which prevents applications from incorporating information used for contact tracing. The protocol also reduces risk of privacy loss by requiring a TEK to correlate between a client's RPIs, with the TEKs changing each day. In addition, it's computationally infeasible for an adversary to find a RPI collision without TEKs. This reduces the risk of impersonation attacks. 

## Hamagen and Hashomer

### Introduction
Hashomer is a contact tracing scheme tailored for the Israeli Ministry of Health’s (MoH) “Hamagen” application.

**Hamagen System.** HaMagen is an app that can tell you have been in the presence of anyone who has been diagnosed with coronavirus. The basic architecture includes copies of the application that run on clients’ phones, and a central server run by the MoH. Following are the basic properties of the operation of the system:

1. When the application is installed on a phone it generates a personal random master key.
2. The application periodically transmits ephemeral IDs using BLE messages. The ephemeral IDs include a 16 byte pseudo-random value that is generated by the application.
3. When the application receives such an ephemeral ID, it saves it together with the current location, and the time when it was received.
4. The application never sends anything over any channel other than BLE. The only exception is by an explicit request of the user, and only when the user was identified as a COVID infected person or as being exposed to a COVID infected person.
5. A user who was identified as a COVID patient can ask the application to send a message to the MoH, which will enable the identification of all messages sent by the user in the last 14 days.
6. The server broadcasts to all users the required to identify the ephemeral IDs sent by new infected persons.

**Privacy Requirements.** Below are the privacy requirements for the system.
1. No information is revealed to the MoH without the user’s explicit consent.
2. If the application learns that the user was exposed to a COVID patient, this information is only revealed to the user and is not sent to any other party.
3. Data collection and usage must be limited to the purpose of proximity tracing.
4. The information sent by a COVID patient to the MoH must not identify the users who were in close proximity to the infected persons.
5. The information sent from the MoH to users, must not include any information related to the identify infected persons.
6. Both the application and the server must delete all information older than 14 days. This includes both secret keys and stored beacons.
7. The application must allow the user to “snooze” the sending of beacons in privacy critical situations.
8. The application must allow a user to delete keys associated with specific time intervals.
9. The application must allow users to delete their history of exposure alerts, and the ephemeral IDs that caused the alerts.


**Security Requirements.** Below are the security requirements for the system.
1. The application should prevent relay attacks, cloning attacks and copying attacks.
2. The scheme will provide COVID-19 positive users with the ability to send a commitment to their keys to the MoH.
3. The design should prevent users from claiming that they were in contact with an infected person.

### Hashomer
#### Key Derivation and Ephemeral Id
**Key Derivation.** A tree-like key derivation scheme is used here. The goal is to allow the server to broadcast separate keys for each epoch. This approach derives all keys in two steps. First derive daily keys from the master key, and from these daily keys derive epoch keys. This allows the server to store the daily or epoch keys in a lexicographic order, without information that allows to link contacts across different days or epochs. Note that from a privacy perspective it is always preferable to only store epoch keys. However, this approach supports the option of storing and sending daily keys in case that the of number of infected persons is large and it is required to optimizing the bandwidth. The key derivation is depicted below.

<div style="text-align:center"><img src=https://i.imgur.com/Lv06x9t.png />The key derivation mechanism</div>

**Ephemeral IDs.**
Let the number of time units in an epoch be $n = T_E/T_U$. The epoch key is used for generating n ephemeral IDs. The computation is done in the following way:

- The ephemeral ID for time unit $s$ in epoch $j$ of day $i$ is a 16 byte value that is generated in the following method. The application sets a random 4 byte value, UserRand, as a truncation of the epoch verification key $K^{i,j}_{epochVER}$. It also computes a 5 byte GeoHash of its current location. It then runs the following computation.
    - Mask$_{i,j,s}$ = AES$(K^{i,j}_{epochENC}, s)$
    - UserRand = Truncate$(K^{i,j}_{epochVER}, 4)$
    - Plain$_{i,j,s}$ = 0 (3 BYTES) || GeoHash (5 Bytes) || UserRand (4 Bytes) || 0 (4 BYTES)
    - C$_{i,j,s}$ = Plain$_{i,j,s} \oplus$ Mask$_{i,j,s}$
    - EphID$_{i,j,s}$ = Truncate$(C_{i,j,s}, 12)$ || Truncate$(AES(K^{i,j}_{epochMAC}, C_{i,j,s}),4)$

The computation is depicted below.

<div style="text-align:center"><img src=https://i.imgur.com/45VHYm6.png />The key derivation mechanism</div>


#### Communication over BLE

**Sending and Receiving Ephemeral IDs.** The transmitting application sends over BLE communication the ephemeral ID corresponding to the current time unit. The receiving application receives over BLE communication an ephemeral ID corresponding to the current time unit. It stores the received ephemeral ID together with its own current location data. The stored location data should be sufficient for computing the GeoHash of the current location and of adjacent locations.


**“Snoozing” Transmission and Keys Redaction.** The application will allow the user to “snooze” the sending of ephemeral IDs. The application will allow the user to redact keys for specific epochs. After the deletion of the required keys only the the remaining subset of keys from the tree will be stored on the device.

**Reporting to the Ministry of Health.** When an application user learns that she carries the corona virus, the user is given the option to send to the MoH information that enables to compute the ephemeral IDs sent by application in the last 14 days.

**Updates Sent from the Ministry of Health to Users.** The server receives from the user the keys for the last 14 days, as well as the verification key and the identity commitment key. (If the system received only the key $K_{DAYmaster}^{d-14}$ or daily keys, it can can compute all the corresponding pre-epoch keys used in each of these days.

**Computing the Update Message.** For every infected person that sends a master key, and for every day, the MoH will make a decision whether to publish the epoch keys of the infected person for that day (or a subset of them), or to publish only the day key (which will save communication but will enable users to link different exposures to that user).

**Checking for Exposure.** The client application stores internally all ephemeral IDs that it received in the last 14 days, together with the corresponding times and locations. We assume that this data is sorted by the reception time.

The client application only needs to compare the values that it received in a specific time to the ephemeral IDs that were sent in corresponding times, rather than to all ephemeral IDs. Recall that the system divides time to epochs and to smaller time units (corresponding in our example to one hour and 5 minutes, respec- tively). The application scans the ephemeral IDs that it received in sorted time order, and defines for each ephemeral ID the time units in which it could have been sent.

The exact correspondence between reception time and transmission time depends on system dependent parameters such as jitter and other issues. For example, a message that was received at the beginning of a time unit might have been sent in that time unit or in the preceding one. Let us denote the set of all these time units as the potential transmission time units. For each time unit in this set, the client application can identify the ephemeral IDs that it received and could have potentially been sent in this time unit.

The client application receives from the MoH sets of day keys and of epoch keys corresponding to new COVID-19 positive persons. It then scans the set of potential transmission time units. Let $i$, $j$ correspond to a day and epoch that contain a potential transmission time unit. Let the set $S_{i,j}$ include all epoch keys that were reported as being used by COVID-19 positive persons in that epoch.

The application decides if the user should be notified as function of the number of matches, and other auxiliary data (such as RSSI records). If this decision is made then the application reports to the user that it identified an exposure in this specific epoch.



## TraceTogether
We finally briefly introduce TraceTogether, Singapore's privacy preserving contact tracing program to limit COVID transmission. Similar to previous protocols, TraceTogether is alo realized by BLE technology. TraceTogether stores limited amount of data, only including a user's contact / mobile number, identification details, and a user ID. We contacting with another user, both devices will generate a temporary ID which is secure and can only be decoded by MOH. This temporary ID also ensures that no thierd-party applicatoins can track a user's identity. The contact data are stored locally on a user's device, and are automatically deleted after 25 days. 

We also consider TraceTogether an efficient decentralized contact tracing protocol. Such conclusion is also backed by the fact that the Signapore Government managed to limit the spread of COVID effectively, while not significantly impacting it's citizens' daily lives.

 


## Conclusion 
We have learned three applications about the key scheduling of broadcasts and the privacy-preserving protocols. Using these systems, there's a high possibility that we can limit the spread of the coronavirus by using this system to alert participants about possible exposure, through someone they have recently been in contact with who has subsequently been positively diagnosed.

## Reference
[Apple-Google Exposure Notification](https://covid19-static.cdn-apple.com/applications/covid19/current/static/contact-tracing/pdf/ExposureNotification-CryptographySpecificationv1.2.pdf?1)
[TraceTogether](https://www.tracetogether.gov.sg/common/privacystatement/index.html)
[Hashomer – A Proposal for a Privacy-Preserving Bluetooth Based Contact Tracing Scheme for Hamagen](https://github.com/eyalr0/HashomerCryptoRef/blob/master/documents/hashomer.pdf)
