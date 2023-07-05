# Investigate multi dimensional plugin configurations

- Author: Veer Busani
- Decision: Option 1

## Introduction

In the new market-based world of Glass, we need the ability to configure plugins for those markets. However, to reduce code duplication and keep things DRY we need to be able to inherit configurations where applicable.

We also need to keep in mind that multi-dimensional inheritance can exist and must be accounted for. For example, H&W currently needs to inherit several configurations from Glass US but if/when it is shipped to Canada that inheritance would need to come from the Canada config instead.

## Motivation

Markets would benefit with multi dimensional plugin configuration and remove code duplication. Only necessary configuration would need to be injected and this ADR would propose abstraction of configuration. Currently, code duplication and redundancy of plugin configurations is visible since each market would need to inject their configuration separately. 

 ### Guiding Principles
 - Reduce code redundancy for markets when injecting configurations
 - Abstraction of configuration where possible

 ## Considered Options

 ### Option 1. Place the configurations directly in `PluginAPIs`

 This solution would ensure that all of the configurations would reside directly with their respective APIs and injected along with them. This option would also create a new Configuration Module, which will have all of the concrete implementation of the Configurations.

  #### Pros:
 - Reduces code duplication as the configuration would be injected directly at the plugin API level
 - Market will only inject the additional configuration as needed

 #### Cons:
 - Maintenance of configuration would need to be checked at two levels

 #### Example

 Taking `OnboardingConfiguration` as example - 

  ```
  
  struct OnboardingConfiguration: OnboardingPluginConfiguration {

    /// Common configuration
    let legacyAppURL: URL = URL(string: "walmartapp://")! 

    /// Market specific configuration
    let webURL: URL = URL(string: "https://www.walmart.ca")!

    let privacyURL: URL = URL(string: "https://www.walmart.ca/en/help/legal/privacy-policy")!

  }

  ```
Per this option, the `OnboardingPluginConfiguration` will be placed directly in the `OnboardingAPI.swift` under the `PluginAPIs` plugin. 


```
  File: OnboardingAPI.swift

  public protocol OnboardingPluginConfiguration {
    /// Common Configration
    var legacyAppURL: URL { get }

    // Market Specific configuration
    var webURL: URL { get }
    var privacyURL: URL { get }
  }

  public protocol OnboardingAPI {}

```

Then, in the new `ConfigurationPlugin` module, the concrete implementation of the Configuration would be added for Market Specific configuration and extension of the protocol to inject the common configuration

```
 Module: ConfigurationPlugin

 extension OnboardingPluginConfiguration {
    let legacyAppURL: URL = URL(string: "walmartapp://")!
  }
 
 struct OnboardingConfiguration: OnboardingPluginConfiguration {
 	/// Concrete implementation
  }

```

This will provide the ability for markets to inject only the market specific configuration as needed. 

 ### Option 2. Externalize configurations

 This solution would externalize all of the configuration outside the project and it would be injected at build time.

  #### Pros:
 - Common place where all of the configuration variables are placed
 - Easy maintenance of the configuration values

 #### Cons:
 - Would need to use Tuist for codegen

 #### Example

 With `Tuist` as an example, the configurations can be converted into a template format and can be used for codegen

 Taking `OnboardingConfiguration` as example -

  ```
  
  public protocol OnboardingPluginConfiguration {
    var legacyAppURL: URL { 
	"{{ argument.legacyAppURL }}"
    }
    
    var webURL: URL { 
    	"{{ argument.webURL }}"
    }

    var privacyURL: URL { 
	"{{ argument.privacyURL }}"
    }
  }
  
  ```

Next, `Tuist` can be used to generate the actual `OnboardingPluginConfiguration.swift` on build time. The values will be externalized and injected.
