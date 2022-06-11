---
title: TapData Shell Overview
parent: TapData Shell
has_children: false
nav_order: 1
---
# TapData Shell Overview

## Overview

The TapData Shell, `tapshell`, is a fully functional  environment for interacting with TapData. You can use the TapData Shell to :

- Explore & Manage TapData Data Store 
- Explore & Manage TapModels 
- Author, test, run & monitor the Jobs & Pipeliens
- Publish APIs

「WIP」tapshell is available as a standalone package in the TapData download center.



## Download and Install `tapshell`

To learn how to download and install the `tapshell`, see [Install `tapshell`](../Deployment/install-and-start.md).

## Connect to a TapData

Once you have installed the TapData Shell , you can connect to a TapData by using the command below. 

```
tapshell --host 127.0.0.1:3030 -u username
```

To specify a  host and port, you can use: `--host`.

`tapshell` prompts you for a password, which it masks as you type.To provide a password with the command instead of using the masked prompt, you can use the `-p` option.



## Learn More

- [Tapshell Usage](./tapshell-usage.md)

  
