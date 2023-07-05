---
title: Glass CLI - Glass Environment
navTitle: Glass CLI - Glass Environment
---

# **Glass Environment Command**

## **Overview**

## **Subcommands**
---
### **`glass environment setup`**
#### **Description**
Setup the environment (download dependencies, initialize githooks)
#### **Options**
- `--path`, `-p`: The path of the directory that contains the xcode project.
#### **Flags**
- `-v`, `--verbose`: Enables verbose logging for the command.
- `--update`, `-u`: Applies --update to any carthage commands.
- `--use-submodules`: Applies --use-submodules to any carthage commands.
#### **Example Usage**
```
glass environment setup
```
---
### **`glass environment destroy`**
#### **Description**
Destroy the environment
#### **Options**
- `--path`, `-p`: The path of the directory that contains the xcode project. Defaults to the current directory.
#### **Flags**
- `-v`, `--verbose`: Enables verbose logging for the command.
- `--clear-carthage-cache`, `-c`: Clears the carthage cache.
- ` --eviscerate`, `-e`: Additionally resets simulators and runs `git clean -fdx`
#### **Example Usage**
```
glass environment destroy
```
---
### **`glass environment validate`**
#### **Description**
Validates the environment. `glass environment setup` runs this command as well.
#### **Options**
- `--path`, `-p`: The path of the directory that contains the xcode project. Defaults to the current directory.
#### **Flags**
- `-v`, `--verbose`: Enables verbose logging for the command.
#### **Example Usage**
```
glass environment validate
```

##### [Return to index](../index.md)