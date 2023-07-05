---
title: Glass CLI - Glass Generate
navTitle: Glass CLI - Glass Generate
---

# **Glass Environment Command**

## **Overview**
Deals with the generation of Xcode Projects.

## **Subcommands**
### **`glass generate plugin`**
#### **Description**
Generates plugins in the glass repo.
#### **Options**
- `-s`, `--style`: The style of plugin. The current options are `platform` and `feature`.
- `-n`, `--name`: The name of the plugin.

#### **Flags**
- `-v`, `--verbose`: Enables verbose logging for the command.
- `-i`, `--include-test-app`: **Not yet implemented!**
#### **Example Usage**
```
glass generate plugin --style feature --name MyFancyNewPlugin
```

##### [Return to index](../index.md)