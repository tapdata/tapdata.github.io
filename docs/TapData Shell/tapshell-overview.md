---
title: Tapdata Shell Overview
parent: Tapdata Shell
has_children: false
nav_order: 1
---
# Tapdata Shell Overview

## Overview

The Tapdata Shell, `tapshell`, is a fully functional  environment for interacting with Tapdata. You can use the Tapdata Shell to :

- Explore & Manage Tapdata Data Store 
- Explore & Manage TapModels 
- Author, test, run & monitor the Jobs & Pipeliens
- Publish APIs

「WIP」tapshell is available as a standalone package in the Tapdata download center.



## Download and Install `tapshell`

To learn how to download and install the `tapshell`, see [Install `tapshell`](../Deployment/install-and-start.md).

## Connect to a Tapdata

Once you have installed the Tapdata Shell , you can connect to a Tapdata by using the command below. 

```
tapshell --host 127.0.0.1:3030 -u username
```

To specify a  host and port, you can use: `--host`.

`tapshell` prompts you for a password, which it masks as you type.To provide a password with the command instead of using the masked prompt, you can use the `-p` option.



## Learn More

- [Tapshell Usage](./tapshell-usage.md)

  
