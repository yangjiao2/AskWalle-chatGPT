---
title: Glass CLI - Glass Apollo
navTitle: Glass CLI - Glass Apollo
---

# **Glass Apollo Command**

## **Overview**
The `glass apollo` command represents the entry point for interacting with things Apollo GraphQL related in the glass ecosystem. 

## **Subcommands**
### **`glass apollo register-persisted-queries`**
#### **Description**
Handles the registration of apollo persisted queries to the mGQL backend
#### **Options**
- `--path`, `-p`: The path to the operations.json file that needs to be registered. If left out program will scan for and register all operations in repo.
#### **Flags**
- `-v`, `--verbose`: Enables verbose logging for the command.
- `--allow-self-signed-certificate`, `-a`: When set self-signed SSL certificates will be permitted
#### **Example Usage**
```
glass apollo register-persisted-queries --path /Path/To/My/operations.json
```

##### [Return to index](../index.md)