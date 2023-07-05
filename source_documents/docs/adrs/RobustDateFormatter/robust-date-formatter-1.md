---
title: Provide a robust DateFormatter
navTitle: Provide a robust DateFormatter
---

# Provide a robust DateFormatter

- Authors: Caleb Halford
- Decision: **Option 2** - More research would be needed for determining specific implementation strategy and API design. Please follow the continued [ADR here](robust-date-formatter-2.md). 

<br/>

# Context:

Jira Ticket: [CEPG-104610](https://jira.walmart.com/browse/CEPG-104610)

Currently, there are 5 CustomDateFormatter.swift files in the Walmart project. Ideally, these should be combined and their APIs made available where/when needed.

## The Files:
1. Walmart/Plugins/CXO/Sources/Helper/CustomDateFormatter.swift
2. Walmart/Plugins/Checkout/Sources/Helpers/CustomDateFormatter.swift - (No tests)
3. Walmart/Plugins/EasyReturns/Sources/Helper/CustomDateFormatter.swift
4. Walmart/Plugins/Authentication/Project/Authentication/Auxiliary/Formatter/CustomDateFormatter.swift
5. Walmart/Plugins/WalmartPlus/Project/WalmartPlus/Sources/Sources/WeeklyReserveSlots/Extensions/CustomDateFormatter.swift

4 of the 5 files are fairly similar, but none are exactly the same. There are minor differences in some of the functions that share the same name, and the greatest file length difference between them is approximately 80 lines of code. The remaining file is the most different, but half of it is still duplicate code (File #4 above).

### Other Similar Files:
These files seem to be mostly comprised of duplicate code from the above files:
1. Walmart/Modules/FeatureUI/BookSlotUIShared/Sources/Helpers/Misc/BookSlotUIModelDateFormatter.swift
2. Walmart/Plugins/TippingAndFeedback/TippingAndFeedback/Sources/Extensions/String+DateUtility.swift

### Other DateFormatter Related Duplicates:
These files have functionality that overlaps with the above files, or could be greatly simplified by implementing the functionality already defined in the above files:
1. Walmart/Plugins/Checkout/Checkout/Sources/Helpers/CustomDateComponentsFormatter.swift
2. Walmart/Plugins/CXO/CXO/Sources/Helper/CustomDateComponentsFormatter.swift

<br/>

# Goal:

Provide a common DateFormatter library that teams can use for all Date/String conversions.

<br/>

# Options:

## Option 1: Condense and Combine

Combine the files into a single file that contains all the needed extensions and functions for the various use-cases where it is needed.

### Pros
- Should require minimal refactoring by the affected feature owners.
- Fastest and easiest solution to implement.

### Cons
- Extensions are applied to all Date and Calendar objects globally, to everyone who imports the containing module, whether they need them or not. 
- High potential for naming clashes.

<br/>

---
## Option 2: GlassDateFormatter

Combine the functionality of all the duplicate files into a new GlassDateFormatter that is made available to the project. I'd also want to simplify the interface and restructure the date string output functions to expose their functionality more efficiently.

### Example:

Current: (Below is snippet from one of the CustomDateFormatter files above.)
```swift
/// e.g., Sun, Wed, Fri
func dayOfWeekShort(in timeZone: TimeZone = ModelDateFormatter.defaultTimeZone) -> String {
    ModelDateFormatter.dayOfWeekShort(in: timeZone).string(from: self)
}

/// e.g., Sun (5/22), Wed (5/25), Fri (5/27)
func dayWithSuffixMonthAndDayShort(in timeZone: TimeZone = ModelDateFormatter.defaultTimeZone) -> String {
    ModelDateFormatter.dayWithSuffixMonthAndDayShort(in: timeZone).string(from: self)
}

/// e.g., Sunday, Wednesday, Friday
func dayOfWeek(in timeZone: TimeZone = ModelDateFormatter.defaultTimeZone) -> String {
    ModelDateFormatter.dayOfWeek(in: timeZone).string(from: self)
}

/// e.g., 1/15, 6/8, 12/31
func monthDayShort(in timeZone: TimeZone = ModelDateFormatter.defaultTimeZone) -> String {
    ModelDateFormatter.monthDayShort(in: timeZone).string(from: self)
}

/// e.g., Jan 15, Jun 8, Dec 31
func monthDayMedium(in timeZone: TimeZone = ModelDateFormatter.defaultTimeZone) -> String {
    ModelDateFormatter.monthDayMedium(in: timeZone).string(from: self)
}

/// e.g. 01/01/2020
func monthDayYear(in timeZone: TimeZone = ModelDateFormatter.defaultTimeZone) -> String {
    ModelDateFormatter.monthDayYear(in: timeZone).string(from: self)
}

/// e.g 2020-01-01
func yearMonthDay(in timeZone: TimeZone = ModelDateFormatter.defaultTimeZone) -> String {
    ModelDateFormatter.yearMonthDay(in: timeZone).string(from: self)
}
```
Output from above:
```swift
Date().dayOfWeekShort()
"Fri"

Date().dayWithSuffixMonthAndDayShort()
"Fri (9/2)"

Date().dayOfWeek()
"Friday"

Date().monthDayShort()
"9/2"

Date().monthDayMedium()
"Sept 2"

Date().monthDayYear()
"09/02/2022"

Date().yearMonthDay()
"2022-09-02"
```

<br/>

The above could become something like this:
```swift
Date().format(.dayShort).string()
"Fri"

Date().format(.dayShort).suffix(.monthDayShort).string()
"Fri (9/2)"

Date().format(.day).string()
"Friday"

Date().format(.monthDayShort).string()
"9/2"

Date().format(.monthDayMedium).string()
"Sept 2"

Date().format(.monthDayYear).string()
"09/02/2022"

Date().format(.isoDate).string()
"2022-09-02"

// Custom
Date().format("MMM d, yyyy").string()
"Sep 2, 2022"
```

The proposed syntax above may not be shorter, but it is more expressive, understandable, and extendable.

### *Added post team share:*
Apple has made improvements to the Date Formatter APIs for iOS 15 and newer: (John Liedtke)  
https://developer.apple.com/documentation/foundation/date/formatstyle
  
So it would be worth investigating how to implement a wrapper on DateFormatter to implement something similar and expose it for our deployment target of iOS 14 and newer. This implementation strategy has the additional advantage of providing a utility that should be familiar to other iOS devs in the organization.

We would also seek to provide convenience methods for common date formats used across teams.

### Pros
- Functionality is made available only where needed.
- Better organization of functionality and code.

### Cons
- Implementation would be more involved as compared to Option 1.
- Would require a good deal of refactoring to implement.

<br/>

---
## Option 3: GlassDate + GlassDateFormatter

This is a bit out of scope for this SPIKE, but I feel this is an opportunity to rethink how we work with dates in the Glass mobile app. Using the existing functionality defined in the CustomDateFormatter files as a "Technical Requirements" template, we can define a novel API (From the Glass App perspective) for working with Dates and Strings. For example, borrowing from popular Date libraries such as [DateTools](https://github.com/MatthewYork/DateTools) and [SwiftDate](https://github.com/malcommac/SwiftDate), one of the features they have in common are the extensions on `Int` which look interesting and provide a natural method of working with dates:

```swift
let timeAgoDate = 2.days.earlier
print("Time Ago: ", timeAgoDate.timeAgoSinceNow)
print("Time Ago: ", timeAgoDate.shortTimeAgoSinceNow)

//Output:
//Time Ago: 2 days ago
//Time Ago: 2d
```

### Pros
- Same benefits as option 2
- Simplifies the creation and manipulation of Dates

### Cons
- Would require the most development effort to create and implement such a solution as compared to the other options above.

<br/>

---
## Decision Outcome
**Option 2** - More research would be needed for determining specific implementation strategy and API design. Please follow the continued [ADR here](robust-date-formatter-2.md).

