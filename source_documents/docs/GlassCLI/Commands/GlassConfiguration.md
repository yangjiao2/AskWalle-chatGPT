---
title: Glass CLI - Glass Configuration
navTitle: Glass CLI - Glass Configuration
---

# **Glass Configuration Command**

## **Overview**
Root command for interacting directly with .glassconf files in the glass repo.

## **Discussion**
The GlassConf system is the backbone of GlassCLI and it's integration into both the daily development and CI lifecycles of the glass ecosystem. The files are standard json with a special file extension. It's main usage is to store metadata associated with plugins and apps in the glass monorepo, which is used to unlock unique build tooling and functionality for those components with minimal effort. 

## **Code Owners**
The GlassConf system places the source of truth for code owners local to the plugin in question, while also providing the required meta data for the teams who own the component to glass cli's build tooling. Future versions of GlassCLI will use the .glasscong files to generate the github CODEOWNERS file.

### **Example**
```
{
  "pluginName" : "WalmartPlatform",
  "owners" : [
    {
      "team" : {
        "name" : "Glass Platform iOS",
        "slackHandle" : "@glass-platform-ios",
        "githubTeam" : "@walmart-ios\/glass-platform-ios"
      },
      "ownedFilePaths" : [
        "Platform\/Modules\/WalmartPlatform"
      ]
    }
  ],
  "relativePath" : "Platform\/Modules\/WalmartPlatform"
}
```

---
## **Subcommands**
### **`glass configuration init`**
#### **Description**
Creates a glassconf file at a specific path.
#### **Options**
- `-p`, `--plugin`: The name of the plugin to create a .glassconf file for. This name will be used in all future commands utilizing the glass conf system
- `-d`, `--destination`: The path to the directory where the glassconf file should be created. It is recommended to do this in the root of the plugin or app target.
- `--glass-repo-root`: The path to the root of the glass repo, defaults to the current directory.
#### **Flags**
-  `-v`, `--verbose`: Enables verbose logging for the command.
#### **Example Usage**
```
glass configuration init --destination /Path/To/MyPlugin --plugin MyPlugin
```

---
### **`glass configuration add-capability`**
#### **Description**
Adds a capability to a specific .glassconf file. Once the capability is added, you must fill out the required details in the modified .glassconf file. See the [capabilities](#capabilities) section below for more information on the supported glass conf capabilities.
#### **Options**
- `-c`, `--capability`: The capability to add to the plugin. (Supports tab completion for possible values!)
- `-p`, `--plugin`: The name of the plugin to add a capability to.
- `--glass-repo-root`: The path to the root of the glass repo, defaults to the current directory.
#### **Flags**
- `-v`, `--verbose`: Enables verbose logging for the command.
#### **Example Usage**
```
glass configuration add-capability --capability localization --plugin MyPlugin
```

---
### **`glass configuration validate`**
#### **Description**
Validates all or a specific .glassconf file(s). All .glassconf files are validated as part of a precommit check.
#### **Options**
- `-p`, `--plugin`: The name of the plugin to validate. **If not passed, all .glassconf files will be validated.**
- `--glass-repo-root`: The path to the root of the glass repo, defaults to the current directory.
#### **Flags**
- `-v`, `--verbose`: Enables verbose logging for the command.
#### **Example Usage**
```
glass configuration validate --plugin MyPlugin
```

---
### **`glass configuration inspect`**
#### **Description**
Allows for the inspecting of specific .glassconf files.
#### **Options**
- `-p`, `--plugin`: The name of the plugin to inspect.
- `--glass-repo-root`: The path to the root of the glass repo, defaults to the current directory.
#### **Flags**
- `-v`, `--verbose`: Enables verbose logging for the command.
#### **Example Usage**
---

## **Capabilities**
### **Overview**
The GlassConf systems allows for easily expanding the functionality of a plugin or app via it's optional capabilities mechanism. To add a capability to your plugin its as easy as running the [add-capability command](#glass-configuration-add-capability) and then filling out the required information for that capability in the modified .glassconf file. 

---
### **Localization**
A simple capability that allows a glass plugin or app to integrate the platform provided build tooling. For usage of strings tooling see the following [documentation](Strings.md).

#### **Required Fields**
- `resourceDirectoryPath`: The path to the plugins resource directory. **MUST BE RELATIVE TO THE REPO's ROOT DIRECTORY**
- `supportedLanguages`: Any array of supported localization locales. Will be used for validation of strings in the platform provided build tooling. Future versions of glass cli may require a base set of supported languages.

#### **Example**
```
"capabilities" : [
    {
      "type" : "localization",
      "resourceDirectoryPath" : "Plugins/MyPlugin",
      "supportedLanguages" : [ 
        "en",
        "en-US",
        "en-CA",
        "es-419", 
        "es-MX",
        "fr-CA"
      ]
    }
  ]
```

##### [Return to index](../index.md)