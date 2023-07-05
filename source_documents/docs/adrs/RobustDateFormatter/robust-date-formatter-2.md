---
title: Robust DateFormatter API design
navTitle: Robust DateFormatter API design
---

# Provide a robust DateFormatter

- Author: Amisha Chordia
- Decision: Option 1 

<br/>

# Context:

Jira Ticket: [CEPG-118403](https://jira.walmart.com/browse/CEPG-118403)

Continued effort on creating a robust date formatter.
For more context and better understanding of this document, please refer the [previous ADR](robust-date-formatter-1.md) on DateFormatters.

<br/>

# Goal:

Design more expressive, understandable and modern Platform API for DateFormatters based on the outcome of [previous ADR](robust-date-formatter-1.md) and the newer [iOS 15+ FormatStyles](https://developer.apple.com/documentation/foundation/date/formatstyle).

<br/>

# Options:

## Option 1: Extension driven

Backward compatible wrapper around iOS 15 FormatStyles, provided as extensions on Date & Time. 

### Example:
**Public API:**

```swift

public struct DateTimePreference {
    public let locale: Locale
    public let calendar: Calendar
    public let timeZone: TimeZone

    public init(locale: Locale = .autoupdatingCurrent,
                calendar: Calendar = .autoupdatingCurrent,
                timeZone: TimeZone = .autoupdatingCurrent) {
        self.locale = locale
        self.calendar = calendar
        self.timeZone = timeZone
    }
}

public enum DateFormatStyle {
    // Weekday
    case weekday_wide           // "EEEE"       ex: Sunday, Wednesday, Friday
    case weekday_abbreviated    // "EEE"        ex: Sun, Wed, Fri

    // Date
    case monthDay_twoDigits     // "M/d"        ex: 1/15, 6/8, 12/31
    case monthDay_abbreviated   // "MMM d"      ex: Jun 8, Dec 31
    case monthDayYear_default   // "MM/dd/yyyy" ex: 01/01/2020
}

public extension Date {
    func formatted(style: DateFormatStyle,
                   dateTimePreference: DateTimePreference = .init()) -> String
    {
        if #available(iOS 15.0, *) {
            return formatted(style: style, dateTimePreference: dateTimePreference)
        } else {
            // Do the traditional date formatter stuff (to be removed when we make iOS 15 min support)
        }
    }
}

public extension String {
    func toDate(style: DateFormatStyle,
                dateTimePreference: DateTimePreference = .init()) -> Date?
    {
        if #available(iOS 15.0, *) {
            return format(formatStyle: style, dateTimePreference: dateTimePreference)
        } else {
            // Do the traditional date formatter stuff (to be removed when we make iOS 15 min support)
        }
    }
}
```

**Usage:**

```swift
Date().formatted(style: .weekday_wide)
Date().formatted(style: .monthDay_abbreviated, dateTimePreference: .init(locale: Locale(identifier: "es")))
```

**Internal Implementation:**

```swift
private extension Date {
    @available(iOS 15.0, *)
    func formatted(style: DateFormatStyle,
                   dateTimePreference: DateTimePreference = .init()) -> String
    {
        let formatStyle = Date.FormatStyle(locale: dateTimePreference.locale,
                                           calendar: dateTimePreference.calendar,
                                           timeZone: dateTimePreference.timeZone)

        switch style {
        case .weekday_wide:
            return self.formatted(formatStyle.weekday(.wide))

        case .monthDay_twoDigits:
            return self.formatted(formatStyle.month(.twoDigits).day())

        case .monthDay_abbreviated:
            return self.formatted(formatStyle.month(.abbreviated).day())

        case .monthDayYear_default:
            return self.formatted(formatStyle.month(.twoDigits).day().year(.defaultDigits))
        }
    }
}

private extension String {
    @available(iOS 15.0, *)
    func toDate(formatStyle: DateFormatStyle,
                dateTimePreference: DateTimePreference = .init()) -> Date?
    {
        let style = Date.FormatStyle(locale: dateTimePreference.locale,
                                     calendar: dateTimePreference.calendar,
                                     timeZone: dateTimePreference.timeZone)
        switch formatStyle {
        case .weekday_wide:
            return try? Date(self, strategy: style.weekday(.wide))

        case .monthDay_twoDigits:
            return try? Date(self, strategy: style.month(.twoDigits).day())

        case .monthDay_abbreviated:
            return try? Date(self, strategy: style.month(.abbreviated).day())

        case .monthDayYear_default:
            return try? Date(self, strategy: style.month(.twoDigits).day().year(.defaultDigits))
        }
    }
}
```


### Pros
- Very similar to the new [FormatStyle API](https://developer.apple.com/documentation/foundation/date/formatstyle) on iOS 15. A small refactor, when iOS 14 is deprecated, would make it way more cleaner and closer to Apple's API.
- With 98% of our user base on iOS 15 and above, performance would benefit from using the newer API than the traditional `static DateFormatter` objects. 

### Cons
- Extensions are applied to all Date and Time objects globally, to everyone who imports the containing module, whether they need them or not. But this may not necessarily be a con, since it is helpful when autocompletion shows up our custom APIs when dealing with dates.

<br/>

---
## Option 2: Enum driven

Backward compatible wrapper around iOS 15 FormatStyles, provided as methods on enum instead of Date/String extensions.


### Example:
**Public API:**

```swift
public enum DateFormatStyle {
    // Weekday
    case weekday_wide           // "EEEE"       ex: Sunday, Wednesday, Friday
    case weekday_abbreviated    // "EEE"        ex: Sun, Wed, Fri

    // Date
    case monthDay_twoDigits     // "M/d"        ex: 1/15, 6/8, 12/31
    case monthDay_abbreviated   // "MMM d"      ex: Jun 8, Dec 31
    case monthDayYear_default   // "MM/dd/yyyy" ex: 01/01/2020

    public func toString(value: Date,
                         dateTimePreference: DateTimePreference = .init()) -> String {
        if #available(iOS 15.0, *) {
            return value.formatted(style: self, dateTimePreference: dateTimePreference)
        } else {
            // Do the traditional date formatter stuff (to be removed when we make iOS 15 min support)
        }
    }

    public func toDate(value: String,
                       dateTimePreference: DateTimePreference = .init()) -> Date? {
        if #available(iOS 15.0, *) {
            return value.toDate(style: self, dateTimePreference: dateTimePreference)
        } else {
            // Do the traditional date formatter stuff (to be removed when we make iOS 15 min support)
        }
    }
}
```

**Usage:**

```swift
DateFormatStyle.weekday_wide.toString(value: Date())
DateFormatStyle.monthDay_abbreviated.toString(value: Date(), dateTimePreference: .init(locale: Locale(identifier: "es")))
```

**Internal Implementation:**
Same as Option 1


### Pros
- Functionality is made available only where needed.
- All pros from Option 1

### Cons
- To deal with DateTime APIs like, stringFromISODurationString or daysSinceDate or similar where we may not necessarily be using the enum at all, we would be most likely adding extensions anyway. This would then get a little messy with extensions and enums methods coexisting.  
- Since these are not extensions on Date & Strings, auto-complete wouldn't suggest these while working with Dates. Hence, there is a slight more chance that these would be overlooked and the team may again create their own extensions.

<br/>

---
## Decision Outcome
Option 1 - Extensions on Date and String types 

