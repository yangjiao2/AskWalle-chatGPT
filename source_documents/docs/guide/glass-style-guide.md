Glass Style Guide
========================

This style guide is based on US platform team's PR review comments  
and incorporates feedback from usage across multiple feature plugin 
projects within Walmart. It is a living document.

Our overarching goals are clarity, consistency and brevity, in that order.

Table of Contents 
-----------------

-   [Making Plugins Generic](#making-plugins-generic)
    -   [Plugin Configuraion](#plugins-configurations)
    -   [Demo Configuraion](#demo-configuraion)
    -   [Rest Endpoints](#rest-endpoints)
    -   [ReadMe](#readme)
    -   [Environment Platform API](#environment-platform-api)
    -   [App Info](#app-info)
 
Making Plugins Generic
-------

### Plugin Configuraion

Container should not be injected as part of the plugin configuration.
While initizing plugin, container and configuration should be separate parameters.

``` 
{
container.register(AuthenticationAPI.self) {
            AuthenticationPlugin(container: $0,
                                 configuration: CanadaAuthConfiguration())
        }
}
```

### Demo Configuraion

Demo Configuration are created for Demo Apps. They are outside the framework.
Make sure target membership is set to Demo Apps.

``` 
AccountPluginDemoConfiguration Target membership is set to AccountApp    
```

### Rest Endpoints

When refactoring plugins identify REST endpoints that is not accessing the 
GraphQL servers and document the findings


### ReadMe

Do not add target membership when adding README files.

### Environment Platform API

Start using EnvironmentPlatformAPI configurations instead of LegacyEnvironment.
// TODO : Get sample code implementation from US Platform team

### App Info

Start using AppInfoPlatformAPI configurations.
// TODO : Get sample code implementation from US Platform team

