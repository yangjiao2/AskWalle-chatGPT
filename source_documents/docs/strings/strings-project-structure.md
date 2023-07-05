---
title: Glass iOS Strings - Project Structure
navTitle: Glass iOS Strings - Project Structure
---

# Glass iOS Strings Project Structure

## File Organization

In order to ensure that tooling can work properly, we enforce the following project structure.

| Path | Description | Auto Generated |
|------|-------------|----------------|
|*PluginResourcesDirectory*/en.lproj/Localizable.strings| Default english translations provided by design| NO|
|*PluginResourcesDirectory*/<lang/locale code>.lproj/Localizable.strings| Translated versions of the default Localizable.strings for the given language and region.| YES |

```
FeaturePlugin
├─ Resources/
│   ├─ en.lproj/
│      ├─ Localizable.strings      // English strings from the design team are put in the default Localizable.strings
│   ├─ en-CA.lproj
│      ├─ Localizable.strings          // Auto generated file that contains English translations for Canada
│   ├─ fr.lproj
│      ├─ Localizable.strings          // Auto generated file that contains French translations
│   ├─ fr-CA.lproj
│      ├─ Localizable.strings          // Auto generated file that contains French translations
```

## Default Translations

The default translation set for every plugin exists in `Plugin/Resources/en.lproj/Localizable.strings`. These
strings are those that are included on the design (currently english for the U.S.). These are the
strings that will be presented to the user as a fallback, if we are unable to match any strings
based on their locale.

###### [Return to index](index.md)
