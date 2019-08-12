---
title: 'GCP Professional Data Engineer Certification'
date: 2019-07-12 00:00:00
excerpt: Being a plumber has never been sexier.
categories: [cloud]
tags: [gcp, certification, devops, cicd]
featured_image: '/images/feature/2.jpg'
#scroll_image:
comments: true
share: true
---
# Abstract

https://cloud.google.com/certification/data-engineer
---


Resources: 

Coursera

# 


DE notes

Functions is reading from pub/sub. publisher sends x100 as many messages as expected but there are no stackdriver log errors. what are the reasons? what can stack driver tell you?
you imported dataset into BQ. you need to make sure it is exactly the same as before you loaded it. how do you do this? HASH()?
note that i didn't need to study any of the ML stuff
your think you need to scale your bigtable instance. using key visualizer, what do read and write pressure mean? if you have too few nodes, even a well designed row key will end up bottlenecking with sustained writes
you have a bigtable instance. you need to run an OLAP workload once an hour but you don't want to bottleneck the main service. what do? make another cluster and implement multi or single cluster routing. use profiles to set what the clusters should be used for.
your time has crazy custom C++ TF ops in their DNN. training is taking too long. how can you optimize with minimal changes to code? TPU? TPU with GPU enabled in code? GPU with GPU enabled in code?
basically understand limitations of BQ updates and all possible workarounds
dont do it
put it in BT or another federated source
put updates in new table, make view of updates + history, update history daily??
what do if you have 1 million updates you need to apply??
