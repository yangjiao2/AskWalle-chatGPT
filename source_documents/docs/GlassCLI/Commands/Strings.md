---
title: Glass CLI - Glass Strings
navTitle: Glass CLI - Glass Strings
---

# **Glass Strings Command**

## **Overview**
Root command for all things localizable strings related. **A MAJORITY OF STRINGS COMMANDS UTILIZE THE GLASS CONF SYSTEM**

## **Discussion**
You can find a detailed documentation guide on setting up your plugin to use the Platform provided localization build tooling [here](../../Strings/index.md).
Future versions of Glass CLI will unlock automatic string publishing and downloading.

## **Subcommands**
### **`glass strings download`**
#### **Description**
Deals with the downloading of localized strings from a TMS (Translation Managment Service).
#### **Options**
-  `-p`, `--plugin`: The name of the plugin to download localized strings for.
#### **Flags**
-  `-v`, `--verbose`: Enables verbose logging for the command.
#### **Example Usage**
```
glass strings download --plugin MyPlugin
```

___
### **`glass strings publish`**
#### **Description**
Deals with the publishing of localized strings from a TMS (Translation Management Service).
#### **Options**
-  `-p`, `--plugin`: The name of the plugin to download localized strings for.
#### **Flags**
-  `-v`, `--verbose`: Enables verbose logging for the command.
#### **Example Usage**
```
glass strings publish --plugin MyPlugin
```

___
### **`glass strings migrate`**
#### **Description**
Prepares a module for migration to the ICU format and localization build tooling. See the [loccalization guide](../../Strings/index.md) for details.
#### **Options**
- `--resource-directory-path`: The path to the resource directory of the plugin.
#### **Flags**
-  `-v`, `--verbose`: Enables verbose logging for the command.
#### **Example Usage**
```
glass strings migrate --resource-directory-path Plugins/MyPlugin/MyPluginResources
```

___
### **`glass strings validate-icu-format`**
#### **Description**
Validates that all localized strings are using valid ICU formatting.
#### **Options**
-  `-p`, `--plugin`: The name of the plugin to validate the ICU format of. If not passed, validation will be run against all .glassconf files that include the optional localization 
#### **Flags**
-  `-v`, `--verbose`: Enables verbose logging for the command.
#### **Example Usage**
```
glass strings validate-icu-format --plugin MyPlugin
```

___
### **`glass strings validate-localizable-folder-structure`**
#### **Description**
Validates that all localized resource directories are setup correctly.
#### **Options**
-  `-p`, `--plugin`: The name of the plugin to validate the localizable directory structure of. If not passed, validation will be run against all .glassconf files that include the optional localization capability.
#### **Flags**
-  `-v`, `--verbose`: Enables verbose logging for the command.
#### **Example Usage**
```
glass strings validate-localizable-folder-structure --plugin MyPlugin
```

___
### **`glass strings validate-translations`**
#### **Description**
Validates that a translation exists for each english key/value pair for all supported locales. By default this will only run on CI unless the force validations flag is passed.
#### **Options**
-  `-p`, `--plugin`: The name of the plugin to validate the localizable directory structure of. If not passed, validation will be run against all .glassconf files that include the 
#### **Flags**
-  `-f`, `--force-validation`: Forces the validation to run even when not on CI. Can be used to locally validate translations.
-  `-v`, `--verbose`: Enables verbose logging for the command.
#### **Example Usage**
```
glass strings validate-translations --plugin MyPlugin --force-validation
```

___
### **`glass strings generate-enum`**
#### **Description**
Generates a `LocalizedString` enum from the strings file for a module. See the [strings enum codegen guide](../../Strings/strings-enum-codegen.md) for a thorough walkthrough of setting up this tooling.
#### **Options**
- `--path`: The path to the base strings file.
- `--output`: The path of the output file.
#### **Flags**
-  `-v`, `--verbose`: Enables verbose logging for the command.
#### **Example Usage**
```
glass strings generate-enum --path Plugins/MyPlugin/Resources/base.lproj/Localizable.strings --output Plugins/MyPlugin/Localizable/LocalizedString.swift
```

##### [Return to index](../index.md)










