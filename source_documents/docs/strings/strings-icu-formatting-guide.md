---
title: Glass iOS Strings - ICU Formatting Guide
navTitle: Glass iOS Strings - ICU Formatting Guide
---

# Glass iOS - ICU Formatting Guide

This document provides examples of how to write some of the more common ICU format messages. For a more in 
depth view, see the [StringsAPI tests](../../Platform/Strings/Tests/GoldenCaseTests.swift) for examples.

## Interpolation

```plist
"string_resource_reference_impl_named_interpolation" = "Hello {userFirstName}!";
```

## Number Formatting: Percents

```plist
"string_resource_reference_impl_number_percent" = "You have downloaded {downloadPercent, number, percent}!";
```
## Number Formatting: Simple

```plist
"string_resource_reference_impl_number" = "You lifted {weightInLbs, number} lbs.!";
```

## Date Formatting: Short

```plist
"string_resource_reference_impl_date" = "XBox X will be released on {releaseDate, date, short}!";
```

## Plurals

```plist
"string_resource_reference_impl_plural" = "{numberOfItems, plural,
=0 {There are no results.}
=1 {There is one result.}
other {There are # results.}
}";
```

## Select

```plist
"testSelectStatementVariableNested" = "Look out, it's {creature, select,
alien {from out of this world!}
predator {extremely dangerous!}
other {just a little baby {creature}!}
}";
```

**For more examples of how to format ICU strings take a look at the following:**
- [GoldenCaseTests](../../Platform/Modules/Strings/StringsTests/GoldenCaseTests.swift)
- [Official Unicode Documentation](https://unicode-org.github.io/icu/userguide/format_parse/messages/)
- [The Missing Guide to the ICU Message Forma](https://phrase.com/blog/posts/guide-to-the-icu-message-format/#:~:text=ICU%20has%20four%20predefined%20date,one%20of%20the%20predefined%20formats.)
- [Quick Guide to ICU Message Format](https://localizely.com/blog/quick-guide-to-icu-message-format/)

###### [Return to index](index.md)
