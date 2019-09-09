---
title: 'GCP Professional Cloud Developer Certification'
date: 2019-08-12 00:00:00
excerpt: Become the AppDev wizard you were meant to be.
categories: [cloud]
tags: [gcp, certification, devops, cicd]
featured_image: '/images/feature/1.jpg'
#scroll_image:
comments: true
share: true
---
# Abstract

These are the raw format notes that I took while studying for the [Google Cloud Professional Cloud Developer Certification](https://cloud.google.com/certification/cloud-developer). I've linked to the various materials that I used to study in the **Resources** section at the bottom. I passed the test and am [officially certified!](https://www.credential.net/profile/galenballew/wallet)

---

# Code and Environment Management
- Use git or something similar
- do not store jar file or other binaries in git
  - build them on demand
- do not store dependencies in git
  - install them via a requirements file at build
  - remember to use explicit versions!
- don't store configuration constants within the code itself
  - use a separate configuration file
  - set them as environment varibles 

Benefits of microservice architecture compared to a monolithic application:
- service boundaries roughly match business boundaries. e.g., payments are processed via the payments service, invoicing is done by the invoicing service.
- services can be updated, deployed, and **scaled** independently of each other.

In order to achieve the promise of microservices, each service must be stateless. Being stateful causes availability issues if an instance of a service goes down and accessing a shared state is a bottleneck for scalability. Stateless service instances can start up quickly and shutdown gracefully. 

--- 

# Security, Reliability, and Migration
- implement healthchecks on storage, database, compute, network, etc
  - you can use this to route requests to unhealthy enpoints to a status page and return a 200.
- collect, monitor, analyze logs. you can use log-based metrics to enable event triggering (e.g., autoscaling, notification, etc)
- applications must be be resilient to both transient and long-lasting errors when accessing data/resources in a distributed system.
  -  for transient errors, use `retry` logic with exponential backoff 
  -  for long running errors, use flags/toggles to avoid/darken the service until it can be restored
     -  called "implementing a circuit breaker"
- perform high-availability/load testing and develop a disastery recovery plan
  - this is in addition to functional and performance testing
  - execute your disaster recovery plan as part of a scheduled "game day" by simulating failures
    - example failure scenarios include region/zone failure, deployment rollback, or network connectivity errors
- automate wherever possible, working towards a DevSecOps methodology via CICD pipelines
- use the strangler method to re-architect applications
  -  migrate one component of a monolithic application at a time while leaving the original application fully functional
  -  requests can be easily directed to the new or old appliction
  -  this is a low risk migration pattern since it will not affect any business critical needs

---

# Datastore
- Objects are called `entities`.
- Entities are either root entities or have `ancestors`.
- Entities are composed of their `key` and `properties`
  - key is composed of namespace, entity `kind`, id, and ancestor id
  - properties can have one or more values
- operations on one or more entities are called `transactions` and are _atomic_
  - atomicity means that either ***all*** operations in a transaction are applied or ***none*** are.
- entities of the same kind do not need to have a consistent property set

- built-in indexes
  - automatic index of each property of each kind
- composite indexes
  - index multiple property values 
  - defined in an index configuration file
  - too many will increase latency to achieve consistency

3 types of queries:  
  1) use `keys-only` when you only need the entity key  
  2) use `projection-query` when you only need specific properties from the entity or properties included in the query filter   
  3) use `ancestor-query` when you need strongly consistency  

Datastore is excellent for structured data that is non-relational. It scales extremely well. However, unlike relational databases, it does not support `join` operations. It does not support inequality filtering on multiple properties. It does not support filtering based on results of a subquery. Practically speaking, this means that you will often make two or more inequality queries and then compute the intersection. 

Remember that Datastore stores keys lexographically. High read/write activity to a local neighborhood of keys (or a relatively new key) will result in bottlenecks. For numeric keys, you can use the allocateIds() method to get keys that are distributed for performance. Outside of that, avoid negative numbers, the number zero, and monotomically increasing numbers.

If you need higher read capacity of a portion of the key range, you can use replication. 

Datastore transactions can fail when they run longer than 60 seconds, there are too many concurrent writes to the same entity group, or a transaction operates on more than 25 different entity groups. Datastore can return exceptions in cases where the transaction will eventually be committed succesfully - therefore, design your transactions to be idempotent. Idempotent means that a the final state will be the same even if a transaction is processed multiple times.

--- 
# Monitoring and Tuning Performance
Be sure to measure and visualize The 4 Golden Metrics:  
1) Latency  
   - Differentiate between successful and unsuccessful requests  
2) Traffic  
3) Errors  
4) Saturation  

---

# Resources: 
- [12 best practices for user account, authorization and password management](https://cloud.google.com/blog/products/gcp/12-best-practices-for-user-account)
- [Choosing an App Engine environment](https://cloud.google.com/appengine/docs/the-appengine-environments)
- [Developing Applications with Google Cloud Platform Specialization](https://www.coursera.org/specializations/developing-apps-gcp)
- [Google API Design Guide](https://cloud.google.com/apis/design/)
- [Big Table Schema Design](https://cloud.google.com/bigtable/docs/schema-design)
- [Big Query Batch](https://cloud.google.com/bigquery/batch)
