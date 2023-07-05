---
title: Glass iOS Strings - Developer Best Practices
navTitle: Glass iOS Strings - Developer Best Practices
---

# Glass iOS - Developer Best Practices

This document contains a set of best practices for working with localized content in iOS Glass.

## Authoring ICU for translators

When authoring ICU, it's important to consider how the translators will translate the copy.  They
won't have the same context you do during translation.  In order to ensure an accurate translation
add as much context as you can in your ICU.

**Use meaningful variable names**

In the example below, we add context to the string by using a descriptive ICU variable name.  That
way, when the translator reads the default english version, they know we're trying to say
"Hello" to a person and not to an object.

Example

```plist
"my_feature_hello" = "Hello {userFirstName}!";
```

**Prefer simple strings that are concatenated into complex strings via named variables**

Although ICU can support very complex nested formatting statements. They can be very hard to read and understand not only by translators but by your fellow engineers.
So instead of a single complex string that has 5 separate format args, it is recommend to create 5 simple strings that are all added together via a string with named args.
For an example of this in practice see the [GoldenCaseTests](../../Platform/Modules/Strings/StringsTests/GoldenCaseTests.swift).

###### [Return to index](index.md)
