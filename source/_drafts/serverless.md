---
title: serverless
toc: true
date: 2023-09-25 14:55:59
tags:
    - design
    - architecture
---

Serverless, not no-server. There's a server which we can't operate. Not like EC2, this is a server on cloud,  which we can login and install sth there, etc.

# An example

[Amazon Serverless Use Cases](https://aws.amazon.com/serverless/)

# what's serverless

1. BAAS

2. FAAS

# Key properties

1. **<u>stateless</u>**: 
2. **<u>event-driven</u>**: only triggered when an event happens (pull/push mode).
3. asynchronous:
4. **<u>use by requirement / ephemeral</u>**: start and expand instances when needed. Thus, it's expecially suitable for those functions that are not always running.
5. **<u>simple and run fast</u>**: if it's complicated, it may takes time to run, which may be very expensive.
6. **<u>third party involved</u>**:

## When not suitable?

1. **<u>high realtime response requirement</u>**: it's event-driven and the function only start when there's the right event. All these communication and environment set-up take time.

2. **<u>long running period</u>**: Aws Lambda or Azure Functions, Google Functions, often charge by using (requests, time-running, etc.). Thus if the function is designed to be always running, it may be extremely expensive. Therefore, 

3. **<u>high SLA requirements:</u>** In the serverless architecture, you often need to rely on many other cloud services and not all dependent services may have the required SLA.

# Features







## Benefits











## Drawbacks

# Applications

## when to use

## when not?





# To be continued

1. my own serverful faas platform using opensource framework.
