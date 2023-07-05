---
title: Glass iOS Strings - Migrating your plugin to ICU
navTitle: Glass iOS Strings - Migrating your plugin to ICU
---

# Glass iOS - Migrating your plugin to ICU

This tutorial outlines the steps necessary to take your existing module and set it up to support
localized content.

## Summary of steps
- Step 0. Update Glass CLI (if necessary).
- Step 1. Run the `glass strings migrate` command for your plugin.
- Step 2. Add new string file to xcode.
- Step 3. Migrate existing iOS-specific c style var arg format specifiers to use ICU.
- Step 4. Update your code base to use the `StringsAPI` instead of `NSLocalizedString`.
- Step 5. Clean up old files, that were created during migration.

## Step 0. Update Glass CLI

Make sure you're running the latest version of Glass CLI by running:

```bash
./scripts/install-glass-cli.swift
```

## Step 1. Run the String Migration Command

Execute the `glass strings migrate` command for your plugin.

Example:
```
glass strings migrate --resource-directory-path <path to your plugin's resource directory>
```

This will do three things:

1. Setup the new file organization as required.
2. Rename and relocate (as needed) any old strings files. These will act as a safe guard against anything going wrong during the migration process.
3. Move all existing strings to the new file. Strings that require formatting will be located below the \[ICU_CONVERSION_NEEDED\] needed comment.

At this point, your plugin's directories will look like this:

```
feature-foo
├─ Resources/
│   ├─ en.lproj/
│      ├─ Localizable.strings           // The new strings file to be used by the plugin
│      ├─ Localizable-old.strings       // Copy of the old strings file, should anything go wrong during migration
│      ├─ Localizable-old.stringsdict   // Copy of the old stringsdict file, should anything go wrong during migration. (if one exists)
```

## Step 2. Add new string file to xcode

Add the new string file to your plugin's xcode project by dragging the `en.lproj` folder from Finder to your `Resources` folder. Select "Copy items if needed", "Create Groups", and make sure your Resources bundle is selected. Make sure you also remove the references to the old string file(s).

## Step 3. Migrate existing iOS-specific c style var arg format specifiers to use ICU.

In the new Localizable.strings, all strings below the \[ICU_CONVERSION_NEEDED\] comment need to be converted from using cvarg style string format specifiers to using ICU.

1. Move the string from above the conversion needed comment.
2. Convert the c style var arg format specifiers to ICU formatting.
3. Write a test to ensure parity
4. Rinse and repeat until all strings are converted.
5. Sanity check.
6. Delete the \[ICU_CONVERSION_NEEDED\] comment.

Example of iOS String using cvarg style format specifiers:

```
"my_feature_hello" = "Hello %@"
```

Example of same string with ICU formatting:

```
"my_feature_hello" = "Hello {name}"
```

Take a look at the [ICU Formatting Guide](strings-icu-formatting-guide.md) and the [Developer Best Practices](strings-developer-best-practices.md) for more guidance and examples on writing easy to use ICU strings.

Explicit examples of all supported formatting can also be found in the [GoldenCaseTests.swift](../../Platform/Modules/Strings/StringsTests/GoldenCaseTests.swift) file in the Strings project.

It is highly recommended to thoroughly sanity check your plugin's UI to ensure everything is still working as expected.

## Step 4. Update your Plugin to use the new StringsAPI

Going forward all user facing localized strings must go through the StringsAPI. It wraps NSLocalizedString so even non formatted strings should be using it for consistency and governance purposes. In the future a swiftlint rule preventing direct usage of `NSLocalizedString` will be added. 

The API offers several easy to use methods that are statically accessible on the String type:

```swift
public extension String {
    private static let stringAPI: StringsAPI = StringsAPI()

    static func localized(forKey key: String, using bundle: Bundle) -> String {
        let localizable = Self.stringAPI.localizable(for: bundle)

        return localizable.getString(forID: key)
    }

    static func localized(forKey key: String, in bundle: Bundle, using arguments: ICUArgument...) -> String {
        let localizable = Self.stringAPI.localizable(for: bundle)

        return localizable.getFormattedString(forID: key, using: arguments)
    }

    static func localized(forKey key: String, in bundle: Bundle, using arguments: [ICUArgument]) -> String {
        let localizable = Self.stringAPI.localizable(for: bundle)

        return localizable.getFormattedString(forID: key, using: arguments)
    }
}
```

While it is very easy to use the above strings it is recommended to add a plugin level extension to wrap the above so you don't need to pass the bundle in every time:

```swift
extension String {
    static func localized(forKey key: String) -> String {
        .localized(forKey: key, using: .module)
    }

    static func localized(forKey key: String, using arguments: ICUArgument...) -> String {
        .localized(forKey: key, in: .module, using: arguments)
    }
}

extension Bundle {
    static var module: Bundle { .init(for: MyPluginFile.self) }
}
```

Many features already centralize their use of Localized strings, so converting will be easy as changing:

```swift
return NSLocalizedString(key, bundle: .module, comment: "")
```
to
```swift
return .localized(forKey: key)
```

Or in cases where arguments are required:
```swift
let someArg = "Test Arg"
let string = NSLocalizedString(key, bundle: .module, comment: "")
return String(format: string, someArg)
```
to
```swift
return .localized(forKey: key, .named(key: "argumentKey",value: someArg))
```

If your module is not already centralizing it's access of Localizable resources. It is recommended you refactor it to do so as it will not only make your life easier going forward but also allow for easier onboarding of engineers new to the project. 

### Example Test

1. Localizable.strings must be included in `{MyPlugin}Resource` target __only__.
2. Your test target should include `{MyPlugin}Resource` in "Copy Bundle Resources" build phase.
3. In your tests you can access `.localized(...)` helpers that you have in your plugin target if you import it as `@testable`.

```swift
@testable import MyPlugin

class MyStringTests {
    func testGreeting() {
        XCTAssertEqual(String.localized(forKey: "greeting"), "Hello World")
    }
}
```

## Step 6. Clean-Up and Next Tutorial

At this point you have migrated your plugin to using ICU and the new StringsAPI! Congrats! Now it's time to clean up the no longer needed files. You can now delete `en.lproj/Localizable-old.strings` and `en.lproj/Localizable-old.stringsdict`.

Now you proceed to add the build tooling in the [Build Tooling Tutorial](strings-localization-tutorial.md).

## Tutorial complete
###### [Return to index](index.md)
