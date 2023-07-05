---
title: TempoAPI
navTitle: TempoAPI
---

# TempoAPI

## Steps to add new module
- Add module type to `TempoModuleTypeMetadata.swift` in `PlatformAPIs`
- If your fragment is going in `TempoUIGQL` (recommended) also add the module type to `SharedModuleMetadata`.
- When you are calling `getFlattenedModules(for: request.layoutId, moduleMetadata: moduleMetadata)` make sure to include your modules in the `moduleMetadata` you are using.
e.g. Adding support for a new module type.
```
        let moduleMetadata: [TempoModuleTypeMetadata] = [
            GenericModuleMetadata<ExistingFragment>(moduleType: TempoModuleType.existing),
            GenericModuleMetadata<FooFragment>(moduleType: TempoModuleType.foo) // <- Added support for new module
        ]
```
- In your code calling `getFlattenedModules` you will receive `AnyTempoModule`s for any successfully received and decoded modules. You should also handle the errors (TODO: Platform add Splunk logging of fails).
- You can further decode the modules of your new type using code inside an initializer `init(module: AnyTempoModule)` like -  
`let configs: FooFragment = try module.convertConfigsFragment()`
- From your new fragment type you can create the view model etc and render the module. e.g. 
```
struct FooModel {
    init(fragment: FooFragment) throws {
        ...
    }
```
}

## How to get support

Use the `#glass-ios-tempo` channel for support.
