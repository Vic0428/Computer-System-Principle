# 14. Data Privacy

**Protecting Your Data** from Anyone!

 What’s the target of **data privacy system**?

- **Allow** data to be **used**
- **Protect** data **from being stolen**

Basic data privacy **method** -> ZKP, OT, HE, sMPC, TEE, DP

**Systems** which try to enforce data privacy

## Zero-Knowledge Proof (ZKP)

 Alice tries **prove** to Bob that she has the answer to a difficult problem. 

### ZKP

- Completeness: Alice **can** construct the proof if she has A
  (You **can** prove anything that’s right)
- Soundness: Alice **cannot** construct the proof if she doesn’t have A
  (You **cannot** prove anything that’s wrong)
- Zero-Knowledge: Bob knows **nothing about A**.  [link](https://towardsdatascience.com/what-are-zero-knowledge-proofs-7ef6aab955fc)

### Interactive Zero-Knowledge Proof

[link](http://gauss.ececs.uc.edu/Courses/c6053/lectures/PDF/zero.pdf)

<img src="lec14.assets/image-20200109093038818.png" alt="image-20200109093038818" style="zoom:50%;" />

![image-20200109133137445](lec14.assets/image-20200109133137445.png)

## Oblivious Transfer

What’s oblivious transfer?

![image-20200110130014704](lec14.assets/image-20200110130014704.png)

The simple protocol: 1-out-of-2 Oblivious Transfer

![image-20200110131110091](lec14.assets/image-20200110131110091.png)

Actually, there are more OT protocols

- Different number of selected messages
  - 1-out-of-2 OT
  - O-out-of-n OT
  - K-out-of-n OT
- Implementation method
  - Non-adaptive OT
  - Adaptive OT
  - Public Verifiable OT

## Homomorphic Encryption

What’s homomorphic encryption?
<img src="lec14.assets/image-20200110131430143.png" alt="image-20200110131430143" style="zoom:50%;" />

SWHE (limited) and FHE (full)
<img src="lec14.assets/image-20200110131654443.png" alt="image-20200110131654443" style="zoom:50%;" />

[More detailed explanation in wikipedia](https://en.wikipedia.org/wiki/Homomorphic_encryption)

## Secure Multi-Party Computing

The problem is that multiparty want to work together to calculate a function

- But we need to enforce **data privacy** for each party

(Alice has data1 and Bob has data2, but they want to compute $f(data1, data2)$)

Yao’s Protocol

- Two party computing
- Semi-host adversary
  - Each party must **follow the protocol** 
- Generic protocol
  - Can securely compute **any functionality** 
- Garbled Circuits + Oblivious Transfer

Millionaire Problem

**Garbled Circuits** -> (Learn in the future) 

Good material from [ZHIHU](https://www.zhihu.com/people/li-tian-tian-13/posts)

## Trusted Execution Environment

![image-20200110160547688](lec14.assets/image-20200110160547688.png)

But what’s trusted execution environment?
![image-20200110161534235](lec14.assets/image-20200110161534235.png)

![image-20200110161612010](lec14.assets/image-20200110161612010.png)

TrustZone divides one system to two worlds: normal world (REE) and secure world (TEE)

The normal world has three layers

- One for hypervisor
- One for kernel
- One for application

The secure world has three layers

- One for TEE OS
- One for TEE application
- Monitoring mode to switch between the two worlds (done through `smc` instruction)

## Differential Privacy

![image-20200110132412334](lec14.assets/image-20200110132412334.png)

Allow user to perform a **random function** M on data set, but get nothing about **individual entry of D**

What is $\epsilon$ DP?
<img src="lec14.assets/image-20200110132655890.png" alt="image-20200110132655890" style="zoom:33%;" />

Secure properties

- Robustness to post-processing
  - You can perform any operations on the result of M, and **get nothing of individual entry of D**. 
- Composability
- Group privacy

![image-20200110132930631](lec14.assets/image-20200110132930631.png)

When DP is enabled
<img src="lec14.assets/image-20200110133048217.png" alt="image-20200110133048217" style="zoom:50%;" />

But how to implement a DP algorithm?

- Adding noise to the function we want to compute
  - Translate function f into a random algorithm $M$
- Existing mechanisms
  - Laplace mechanism
  - Gaussian mechanism

## SAGE: Privacy Accounting and Quality Control in the Sage Differentially Private ML Platform

Sosp 2019

### Motivation

ML should capture general trends from the data, but often **captures specific information about individual entries** in the dataset.  

## Oblivious Multi-Party Machine Learning on Trusted Processors

USENIX Security 2016

### Motivation

Large attack surface: **sensitive data could be stolen**. 

### Threat model

Assumptions: 

- Code does not leak secrets 
- Do not consider leakage through out time or power channels
- Any party can be malicious

<img src="lec14.assets/image-20200110191644376.png" alt="image-20200110191644376" style="zoom:50%;" />

SGX’s Vulnerabilities

![image-20200110191744223](lec14.assets/image-20200110191744223.png)

## Differential privacy systems

### Data privacy + ML

- Sage (SOSP 19)
- Oblivious multi-party ML (UNESIX Security 16)
- Chiron, ...

### Data privacy + Database

- CryptDB (SOSP 11)
- EnclaveDB (S&P 18)

### Data privacy + Data analysis

- Opaque(NSDI 17)

More ...

## Reference

1. [CSP Lecture 14](https://ipads.se.sjtu.edu.cn/courses/csp/slides/CSP_14_Data_Privacy.pptx)

