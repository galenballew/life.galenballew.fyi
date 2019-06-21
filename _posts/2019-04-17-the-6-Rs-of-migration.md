---
title: 'The 6 Rs of Cloud Migration'
date: 2019-04-24 00:00:00
excerpt: TL;DR on this AWS blog post.
categories: [tldr, cloud]
tags: [tldr, cloud, aws]
featured_image: '/images/feature/7.jpg'
#scroll_image: '/images/'
comments: true
share: true
---

# Abstract

This post is a quick summary of 6 common migration strategies that Amazon observes from its customers. You can read the original post [here](https://aws.amazon.com/blogs/enterprise-strategy/6-strategies-for-migrating-applications-to-the-cloud/). It's part of a series of posts about cloud migration. 

---

# The Six R's

## 1. Rehosting
#### AKA "lift-and-shift"
Rehosting does not take advantage of any cloud-native capabilities. It is simply running your existing legacy app on cloud infrastructure. Most rehosting can be done using automated tools, however a manual migration can be useful to learn how to apply the workload in the new cloud environment. Additionally, applications are often easier to optimize or re-architect once they are already running in the cloud. This party because of people, talent, and process and also partly because migration of the application, data, and traffic is already done (the hard part). 

## 2. Replatforming
#### AKA "lift-tinker-and-shift"
Replatforming conserves the core architecture of an application, but leverages some cloud capabilities. An example might be replacing a database instance in your data center with a database-as-a-service platform that handles some of the operational load. Another example might be replacing queue or pub/sub service with AWS SQS. 

## 3. Repurchasing
#### AKA "pay someone else to do it better, for less"
This strategy is when companies move to a SaaS platform. Common examples are moving your CMR to Salesforce or an HR system to Workday.

## 4. Refactoring / Re-architecting
#### AKA "harder, better, faster, stronger"
This strategy is a full blown re-imagination of the applications core architecture. It's usually driven by business needs for new features, scale, or performance that are not feasible on-premise. It can also be part of a larger strategic initiative like shifting towards portable workloads (containers), container management platforms such as Kubernetes (I like to think of them as abstracted-compute platforms), and multi-cloud or multi-data center postures. 

## 5. Retire
#### AKA "goodbye and farewell"
Sometimes it's the better business decision to shut down an application entirely. The resources (people, money, compute) freed up from this can be put to better use. 

## 6. Retain
#### AKA "kick the can down the road"
Randy Pausch said "Never make a decision until you have to." If an application does not meet the bar for taking action via one of the other five migration strategies, it will be retained. 

