---
title: 「WIP」Plan Your Deployment
parent: Deployment
has_children: false
nav_order: 3
---

# Plan Your Deployment
This document will be updated to the official version on June 30.

Tapdata is designed with light weight and low up front cost in mind. You may choose an appropriate configuration based on your needs. 


## Hardware Requirements

Tapdata can be installed on physical servers, virtual machine or docker. 

Recommended Spec:

- CPU Cores: 8+
- RAM: 16GB+
- Storage: MongoDB Nodes only. SSD or high performance disk, 50GB+ 

 
## Minimum Configuration - Not for Mission Critical Use Cases

This is suitable for development, testing or non-critical produciton use cases that can afford prolonged downtime as this is a non-HA setup. 

- Tapdata Server:  1x
- MongoDB Servers: 1x



## Standard Configuration 

- Tapdata Server:  2x
- MongoDB Servers: 3x


## When to scale out your Tapdata Cluster

As a very rough guide, each Tapdata server with 8c can handle:

- 16 concurrent data pipelines, assuming each pipeline processes up to 1000 events per second, with each event is less than 1KB, or
- 100 concurrent API requests per second

When the number of data pipelines or API requests get bigger and bigger. We can perform horizontal scaling by simply adding more Tapdata nodes.  