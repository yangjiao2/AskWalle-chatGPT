# Style Guide
We should strive for a uniform coding style in order to:

- Make it easier to read and begin understanding unfamiliar code.
- Make code easier to maintain.
- Reduce simple programmer errors.
- Reduce cognitive load while coding.
- Keep discussions on diffs focused on the code's logic rather than its style.
- Help new programmers learn good Swift practices and idioms.

Note that brevity is not a primary goal. Code should be made more concise only if other good code qualities (such as readability, simplicity, and clarity) remain equal or are improved.
Make sure to read [Apple's API Design Guidelines](https://swift.org/documentation/api-design-guidelines/).

Beyond the rules that follow, some other good practices to follow are:
- Include JIRA tickets with any TODOs
- Don't commit commented out code, that's what source control tools are for.

And remember, use a style for a while and it becomes your style. Code on 😃.

## SwiftFormat  

We are using [SwiftFormat](https://github.com/nicklockwood/SwiftFormat) to help us adhere to the same code style and conventions. SwiftFormat will automatically reformat changes in the *_staged_* files in order to follow project conventions.
Formatting is automatically run as part of the pre-commit hooks before SwiftLint so that linting is done against the reformatted code.

### Install

To install do this
```shell
./scripts/setup-mint.sh
```

### SwiftFormat for Xcode Extension

It is recommended to install the [SwiftFormat for Xcode extension](https://github.com/nicklockwood/SwiftFormat#xcode-source-editor-extension) for SwiftFormat. This extension allows you to manually invoke formatting while developing. Note you may have to enable the extension in *System Preferences - Extensions*.
Once you have opened the SwiftFormat for Xcode application, update the rules it uses to our rules using the *File-Open* menu to open the hidden `.swiftformat` file at the root of this git repo. You need to do this any time after the `SwiftFormat for Xcode` app is restarted.
**Tip**: If hidden files don't show you can press *Command + Shift + . (period)*

### Glass CLI Format
Staged files can be manually formatted using 
```shell
glass format staged (-v)
# or just 
glass format
```
This will also update the corresponding non-staged files.

Individual file or folders can also be formatted outside of the index.
```shell
glass format path "Plugin/Foo/CoolClass.swift" (-v)
```

**[⬆ back to top](#style-guide)**

## Tips

- Stage (add) your changes separately from committing. This allows you run `glass format` to see the changes outside of the automatic reformatting applied by the Git hook.
- Use the [SwiftFormat for Xcode extension](#swiftformat-for-xcode-extension) as you work. Setup and use keyboard shortcuts for SwiftFormat to format a selection or to format the current file.
- Consider doing one or more PRs just to reformat your plugin code separately from your normal code changes.

### Handling issues

Be aware you may have to manually adjust the reformatted code due to SwiftLint related warnings after the code has been formatted. This is expected. 

Some SwiftLint rules may have been manually disabled and the disable directive may now be superfluous. 

Other rules may get triggered. Normally it is straightforward to identify the issue and adjust the code. Usually this results in more readable/consistent code.

If you have no good fix to either a SwiftLint or SwiftFormat issue you can disable particular SwiftFormat rules using directives in a similar manner as supported by SwiftLint. 

If disabling a SwiftFormat rule, you may need to undo the particular changes done by SwiftFormat adding the directive. The Xcode extension is useful here as you can undo changes made using it within Xcode. Whereas using `glass format` or the pre-commit hook will require you to revert using your Git tool of choice.

You can disable rules for specific files or code ranges by using `swiftformat:` directives in comments inside your Swift file. To temporarily disable one or more rules inside a source file, use:

```swift
// swiftformat:disable <rule1> [<rule2> [rule<3> ...]]
```

To enable the rule(s) again, use:

```swift
// swiftformat:enable <rule1> [<rule2> [rule<3> ...]]
```

To disable all rules use:

```swift
// swiftformat:disable all
```

And to enable them all again, use:

```swift
// swiftformat:enable all
```

To temporarily prevent one or more rules being applied to just the next line, use:

```swift
// swiftformat:disable:next <rule1> [<rule2> [rule<3> ...]]

let foo = bar // rule(s) will be disabled for this line
let bar = baz // rule(s) will be re-enabled for this line
```

There is no need to manually re-enable a rule after using the next directive.

NOTE: The `swiftformat:enable` directives only serves to counter a previous swiftformat:disable directive in the same file. It is not possible to use swiftformat:enable to enable a rule that was not already enabled when formatting started.

### Validate the changes

Run `swiftlint --quiet` or `swiftlint <path to file/folder>` after formatting to check for any resulting SwiftLint issues that may need addressed after the code has changed. e.g. You may have silenced a SwiftLint warning but that may no longer be needed and you see an issue like this Superfluous Disable Command Violation. You need to remove that disable directive.

### Help

Ask for advice from Platform team early if you have any questions or issues.

**[⬆ back to top](#style-guide)**

## Style rules

## Naming
- Use `PascalCase` for type and protocol names, and `lowerCamelCase` for everything else.
- Name booleans like  `isSpaceship`,  `hasSpacesuit`, etc. This makes it clear that they are booleans and not other types.
- Refer to [API Design Guide](https://www.swift.org/documentation/api-design-guidelines/#naming)

**[⬆ back to top](#style-guide)**

### Acronyms
Capitalize the acroynym fully rather than just the first character. 
Currently UUID is automated (as it could break a lot of our code where both Id/ID and Url/URL are widely used),
but try to do this for ID, URL and others too.

This
```swift
let url: URL
let destinationURL: URL
let id: ID
let screenID = "screenId"
let validURLs: Set<URL>
let validUrlschemes: Set<URL>

let uniqueIdentifier = UUID()

/// Opens URLs based on their scheme
struct URLRouter {}

/// The ID of a screen that can be displayed in the app
struct ScreenID {}
```

Not this
```swift
let url: URL
let destinationUrl: URL
let id: ID
let screenId = "screenId"
let validUrls: Set<URL>
let validUrlschemes: Set<URL>

let uniqueIdentifier = UUID()

/// Opens Urls based on their scheme
struct UrlRouter {}

/// The Id of a screen that can be displayed in the app
struct ScreenId {}
```

[![SwiftFormat: acronyms](https://img.shields.io/badge/SwiftFormat-acronyms-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#acronyms)

**[⬆ back to top](#style-guide)**

### And operator
Prefer && operator over commas.

This
```swift
if true && true {}
```

Not this
```swift
if true, true {}
```

[![SwiftFormat: andOperator](https://img.shields.io/badge/SwiftFormat-andOperator-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#andOperator)

**[⬆ back to top](#style-guide)**

### AnyObject protocol
Prefer `AnyObject` protocol over `class`.

This
```swift
public protocol FooProtocol: AnyObject {
```

Not this
```swift
public protocol FooProtocol: class {
```

[![SwiftFormat: anyObjectProtocol](https://img.shields.io/badge/SwiftFormat-anyObjectProtocol-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#anyObjectProtocol)

**[⬆ back to top](#style-guide)**

### Blank line after imports
Have a blank line after import statements.

This
```swift
import A
import B
@testable import D

class Foo {
    // foo
}
```

Not this
```swift
import A
import B
@testable import D
class Foo {
    // foo
}
```

[![SwiftFormat: blankLineAfterImports](https://img.shields.io/badge/SwiftFormat-blankLineAfterImports-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#blankLineAfterImports)

**[⬆ back to top](#style-guide)**

### Blank lines around MARK
Have a blank line before and after MARK: comments.

This
```swift
func foo() {
    // foo
  }

  // MARK: bar

  func bar() {
    // bar
  }
```

Not this
```swift
func foo() {
    // foo
  }
  // MARK: bar
  func bar() {
    // bar
  }
```

[![SwiftFormat: blankLinesAroundMark](https://img.shields.io/badge/SwiftFormat-blankLinesAroundMark-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#blankLinesAroundMark)

**[⬆ back to top](#style-guide)**

### No Blank lines at end of scope
Do not have a trailing blank line at the end of a scope.

This
```swift
func foo() {
    // foo
}
```

Not this
```swift
func foo() {
    // foo

}
```

[![SwiftFormat: blankLinesAtEndOfScope](https://img.shields.io/badge/SwiftFormat-blankLinesAtEndOfScope-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#blankLinesAtEndOfScope)

**[⬆ back to top](#style-guide)**

### No Blank lines at start of scope
Do not have a trailing blank line at the start of a scope.

This
```swift
func foo() {
    // foo
}
```

Not this
```swift
func foo() {

    // foo
}
```

[![SwiftFormat: blankLinesAtStartOfScope](https://img.shields.io/badge/SwiftFormat-blankLinesAtStartOfScope-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#blankLinesAtStartOfScope)

**[⬆ back to top](#style-guide)**

### Insert blank line before declarations
Have a blank line before class, struct, enum, extension, protocol or function declarations.

This
```swift
func foo() {
    // foo
}

func bar() {
    // bar
}
```

Not this
```swift
func foo() {
    // foo
}
func bar() {
    // bar
}
```

[![SwiftFormat: blankLinesBetweenScopes](https://img.shields.io/badge/SwiftFormat-blankLinesBetweenScopes-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#blankLinesBetweenScopes)

**[⬆ back to top](#style-guide)**

### Don't use block comments
Don't use block comments. Use single line style comments.

This
```swift
// foo
// bar

/// foo
/// bar
``` 

Not this
```swift
/*
 * foo
 * bar
 */

/**
 * foo
 * bar
 */
```

[![SwiftFormat: blockComments](https://img.shields.io/badge/SwiftFormat-blockComments-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#blockComments)

**[⬆ back to top](#style-guide)**

### Braces
Use K&R style where opening brace appears on the same line.

This
```swift
if foo {
    // foo
}
```

Not this
```swift
if foo
{
    // foo
}
```

[![SwiftFormat: braces](https://img.shields.io/badge/SwiftFormat-braces-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#braces)

**[⬆ back to top](#style-guide)**

### Single blank lines
Don't use consecutive blank lines between lines of code. Use a single blank line.

This
```swift
func foo() {
    let x = "bar"
    
    print(x)
}
```

Not this
```swift
func foo() {
    let x = "bar"
    
    
    print(x)
}
```

[![SwiftFormat: consecutiveBlankLines](https://img.shields.io/badge/SwiftFormat-consecutiveBlankLines-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#consecutiveBlankLines)

**[⬆ back to top](#style-guide)**

### Single spaces in text
Just use a single space between text.

This
```swift
let foo = 5
```

Not this
```swift
let foo   = 5
```

[![SwiftFormat: consecutiveSpaces](https://img.shields.io/badge/SwiftFormat-consecutiveSpaces-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#consecutiveSpaces)

**[⬆ back to top](#style-guide)**

### Use doc comments
Use /// over // preceding declarations. Use // elsewhere.

This
```swift
/// A placeholder type used to demonstrate syntax rules
public enum Foo {
    // bar
}
```

Not this
```swift
// A placeholder type used to demonstrate syntax rules
public enum Foo {
    /// bar
}
```

[![SwiftFormat: docComments](https://img.shields.io/badge/SwiftFormat-docComments-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#docComments)

**[⬆ back to top](#style-guide)**

### `else` on same line
The `else` statement following `if { ... }` appears on the same line as the closing brace. e.g. 

This
```swift
if true {
    1
} else { 2 }
```

Not this
```swift
if true {
    1
}
else { 2 }
```

Also applies to `catch` after `try` and `while` after `repeat`.
If `guard` statement spans multiple lines, `else` on next line after `guard` closing parenthesis, otherwise on same line.

This
```swift
guard let foo = bar,
        bar > 5
else {
    return
}
```

Not this
```swift
guard let foo = bar,
        bar > 5 else {
    return
}
else { 2 }
```

[![SwiftFormat: elseonsameline](https://img.shields.io/badge/SwiftFormat-elseonsameline-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#elseonsameline)

**[⬆ back to top](#style-guide)**

### Empty braces
Remove whitespace inside empty braces.

This
```swift
func foo() {}
```

Not this
```swift
func foo() {

}
```

[![SwiftFormat: emptyBraces](https://img.shields.io/badge/SwiftFormat-emptyBraces-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#emptyBraces)

**[⬆ back to top](#style-guide)**

### enums for namespaces
Types used for hosting only as namespaces for static members should be enums (an empty enum is the canonical way to create a namespace in Swift as it can't be instantiated).

This
```swift
enum Foo {
    static let bar = "bar"
}
```

Not this
```swift
struct Foo {
    static let bar = "bar"
}
```

[![SwiftFormat: enumNamespaces](https://img.shields.io/badge/SwiftFormat-enumNamespaces-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#enumNamespaces)

**[⬆ back to top](#style-guide)**

### Access control in extensions
Access control modifiers in extensions shall be on the declarations not the top-level extension.

This
```swift
extension Foo {
    private var publicProperty: Int { 10 }
    public func publicFunction1() {}
    private func publicFunction2() {}
    internal func internalFunction() {}
    private func privateFunction() {}
    private var privateProperty: Int { 10 }
}
```

Not this
```swift
private extension Foo {
    var publicProperty: Int { 10 }
    public func publicFunction1() {}
    func publicFunction2() {}
    internal func internalFunction() {}
    private func privateFunction() {}
    fileprivate var privateProperty: Int { 10 }
}
```

[![SwiftFormat: extensionAccessControl](https://img.shields.io/badge/SwiftFormat-extensionAccessControl-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#extensionAccessControl)

**[⬆ back to top](#style-guide)**

### `case let` pattern inline
`let` statements attached to case patterns should be inline. e.g.

This
```swift
if case .foo(let bar, let baz) = quux {
    // inner foo
}
```

Not this
```swift
if case let .foo(bar, baz) = quux {
    // inner foo
}
```

[![SwiftFormat: hoistPatternLet](https://img.shields.io/badge/SwiftFormat-hoistPatternLet-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#hoistPatternLet)

### Indentation
- Use 4 spaces to indent lines. 
- Do not indent **#ifdef** lines.
- Do not indent multiline strings.
- `case` statements within `switch` structures are not indented.

This
```swift
let foo = """
hello
world
"""

switch foo {
case bar:
    break
case baz:
    break
}
```

Not this
```swift
let foo = """
	hello
	world
	"""

switch foo {
case bar:
    break
case baz:
    break
}
```

[![SwiftFormat: indent](https://img.shields.io/badge/SwiftFormat-indent-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#indent)

**[⬆ back to top](#style-guide)**

### Use isEmpty
Prefer isEmpty over comparing count against zero.

This
```swift
if foo.isEmpty
```

Not this
```swift
if foo.count == 0
```

[![SwiftFormat: isEmpty](https://img.shields.io/badge/SwiftFormat-isEmpty-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#isEmpty)

**[⬆ back to top](#style-guide)**

### MARK top-level types and extensions (Recommended)
Add a mark comment before top-level types and extensions.

This
```swift
// MARK: - FooViewController

final class FooViewController: UIViewController { }
```

Not this
```swift
final class FooViewController: UIViewController { }
```

[![SwiftFormat: markTypes](https://img.shields.io/badge/SwiftFormat-markTypes-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#markTypes)

**[⬆ back to top](#style-guide)**

### Modifier order
There should be a consistent order of modifiers `public`, `override` etc for all definitions.
The access control modifier is first followed by others as per the `.swiftformat` configuration file.

This
```swift
public private(set) lazy weak var foo: UIView?
```

Not this
```swift
lazy public weak private(set) var foo: UIView?
```

[![SwiftFormat: modifierOrder](https://img.shields.io/badge/SwiftFormat-modifierOrder-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#modifierOrder)

**[⬆ back to top](#style-guide)**

### Number formatting
Use consistent grouping for numeric literals. Groups will be separated by _ delimiters to improve readability.
See `.swiftformat` file for details.


[![SwiftFormat: numberFormatting](https://img.shields.io/badge/SwiftFormat-numberFormatting-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#numberFormatting)

**[⬆ back to top](#style-guide)**

### Prefer key path syntax
Use `KeyPath`-based syntax rather than anonymous closures.

This
```swift
let barArray = fooArray.map(\.bar)
```

Not this
```swift
let barArray = fooArray.map { $0.bar }
```

[![SwiftFormat: preferKeyPath](https://img.shields.io/badge/SwiftFormat-preferKeyPath-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#preferKeyPath)

**[⬆ back to top](#style-guide)**

### Don't include redundant self
Only use `self.` prefix when it is actually needed.

This
```swift
func foobar(foo: Int, bar: Int) {
    self.foo = foo
    self.bar = bar
    baz = 42
}
```

Not this
```swift
func foobar(foo: Int, bar: Int) {
    self.foo = foo
    self.bar = bar
    self.baz = 42
}
```

[![SwiftFormat: redundantSelf](https://img.shields.io/badge/SwiftFormat-redundantSelf-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#redundantSelf)

**[⬆ back to top](#style-guide)**

### Use type inference
Don't include types where they can be easily inferred.

This
```swift
let view = UIView()
let view1 = UIView()
```

Not this
```swift
let view: UIView = UIView()
let view1: UIView = .init()
```

[![SwiftFormat: redundantType](https://img.shields.io/badge/SwiftFormat-redundantType-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#redundantType)

### Sorted declarations
Where is makes sense sort groups of declarations alphabetically.
`// swiftformat:sort`can help with this.

This
```swift
// swiftformat:sort
enum FeatureFlags {
    case barFeature
    case fooFeature
    case upsellA(
        fooConfiguration: Foo,
        barConfiguration: Bar
    )
    case upsellB
}
```

Not this
```swift
enum FeatureFlags {
    case upsellB
    case fooFeature
    case barFeature
    case upsellA(
        fooConfiguration: Foo,
        barConfiguration: Bar
    )
}
```

[![SwiftFormat: sortdeclarations](https://img.shields.io/badge/SwiftFormat-sortdeclarations-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#sortdeclarations)

**[⬆ back to top](#style-guide)**

### Sorted imports
Sort import statements alphabetically.

This
```swift
import Bar
import Foo
```

Not this
```swift
import Foo
import Bar
```

[![SwiftFormat: sortedImports](https://img.shields.io/badge/SwiftFormat-sortedImports-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#sortedImports)

**[⬆ back to top](#style-guide)**

### Space around braces
Include spaces around braces

This
```swift
foo.filter { return true }.map { $0 }
```

Not this
```swift
foo.filter{ return true }.map{ $0 }
```

[![SwiftFormat: spaceAroundBraces](https://img.shields.io/badge/SwiftFormat-spaceAroundBraces-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#spaceAroundBraces)

**[⬆ back to top](#style-guide)**

### Space around brackets
Include spaces around brackets

This
```swift
foo as [String]
```

Not this
```swift
foo as[String]
```

[![SwiftFormat: spaceAroundBrackets](https://img.shields.io/badge/SwiftFormat-spaceAroundBrackets-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#spaceAroundBrackets)

**[⬆ back to top](#style-guide)**

### Space around comments
Include spaces before and/or after comments

This
```swift
let a = 5 // assignment
func foo() { /* ... */ }
```

Not this
```swift
let a = 5// assignment
func foo() {/* ... */}
```

[![SwiftFormat: spaceAroundComments](https://img.shields.io/badge/SwiftFormat-spaceAroundComments-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#spaceAroundComments)

**[⬆ back to top](#style-guide)**

### Space inside comments
Add leading and/or trailing space inside comments.

This
```swift
let a = 5 // assignment
func foo() { /* ... */ }
```

Not this
```swift
let a = 5 //assignment
func foo() { /*...*/ }
```

[![SwiftFormat: spaceInsideComments](https://img.shields.io/badge/SwiftFormat-spaceInsideComments-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#spaceInsideComments)

**[⬆ back to top](#style-guide)**

### Space around generics
Don't use space around angle brackets.

This
```swift
Foo<Bar>()
```

Not this
```swift
Foo <Bar> ()
```

[![SwiftFormat: spaceAroundGenerics](https://img.shields.io/badge/SwiftFormat-spaceAroundGenerics-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#spaceAroundGenerics)

**[⬆ back to top](#style-guide)**

### Space inside generics
Don't use space inside angle brackets.

This
```swift
Foo<Bar>
```

Not this
```swift
Foo< Bar >
```

[![SwiftFormat: spaceInsideGenerics](https://img.shields.io/badge/SwiftFormat-spaceInsideGenerics-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#spaceInsideGenerics)

**[⬆ back to top](#style-guide)**

### Space around operators and ranges
Use space around operators.
Have no space around ranges.

This
```swift
a + b + c
0..<i
```

Not this
```swift
a+b+c
0 ..< i
```

[![SwiftFormat: spaceAroundOperators](https://img.shields.io/badge/SwiftFormat-spaceAroundOperators-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#spaceAroundOperators)

**[⬆ back to top](#style-guide)**

### Space around parentheses
Use space around parentheses as follows:-
- There should generally be no space between an opening paren and the preceding identifier, unless the identifier is one of certain keywords (see SwiftFormat code for details).
- There should be no space between an opening paren and the preceding closing brace
- There should be no space between an opening paren and the preceding closing square bracket
- There should be space between a closing paren and following identifier
- There should be space between a closing paren and following opening brace
- There should be no space between a closing paren and following opening square bracket

This
```swift
init(foo)
switch (x) {
```

Not this
```swift
init (foo)
switch(x){
```

[![SwiftFormat: spaceAroundParens](https://img.shields.io/badge/SwiftFormat-spaceAroundParens-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#spaceAroundParens)

**[⬆ back to top](#style-guide)**

### Space inside parentheses
No spacing immediately inside parentheses.

This
```swift
(a, b)
```

Not this
```swift
( a, b )
```

[![SwiftFormat: spaceInsideParens](https://img.shields.io/badge/SwiftFormat-spaceInsideParens-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#spaceInsideParens)

**[⬆ back to top](#style-guide)**

### Space inside braces
Use space inside curly braces.

This
```swift
foo.filter { return true }
```

Not this
```swift
foo.filter {return true}
```

[![SwiftFormat: spaceInsideBraces](https://img.shields.io/badge/SwiftFormat-spaceInsideBraces-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#spaceInsideBraces)

**[⬆ back to top](#style-guide)**

### Space inside square brackets
No space inside square brackets.

This
```swift
[1, 2, 3]
```

Not this
```swift
[ 1, 2, 3 ]
```

[![SwiftFormat: spaceInsideBrackets](https://img.shields.io/badge/SwiftFormat-spaceInsideBrackets-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#spaceInsideBrackets)

**[⬆ back to top](#style-guide)**

### TODOs
Use correct formatting for TODO:, MARK: or FIXME: comments.

This
```swift
// TODO: fix this properly
// MARK: - UIScrollViewDelegate
```

Not this
```swift
// TODO fix this properly
// MARK - UIScrollViewDelegate
```

[![SwiftFormat: todos](https://img.shields.io/badge/SwiftFormat-todos-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#todos)

**[⬆ back to top](#style-guide)**

### Trailing closures
Use trailing closure syntax where applicable.

This
```swift
DispatchQueue.main.async { ... }
```

Not this
```swift
DispatchQueue.main.async(execute: { ... })
```

[![SwiftFormat: trailingClosures](https://img.shields.io/badge/SwiftFormat-trailingClosures-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#trailingClosures)

**[⬆ back to top](#style-guide)**

### No trailing comma in collection literals

This
```swift
let array = [
    foo,
    bar,
    baz
]
```

Not this
```swift
let array = [
    foo,
    bar,
    baz,
]
```

[![SwiftFormat: trailingCommas](https://img.shields.io/badge/SwiftFormat-trailingCommas-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#trailingCommas)

### Trim trailing whitespace
Remove trailing space at end of a line.

[![SwiftFormat: trailingSpace](https://img.shields.io/badge/SwiftFormat-trailingSpace-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#trailingSpace)

**[⬆ back to top](#style-guide)**

### Shorthand syntax
Use shorthand syntax for Arrays, Dictionaries and Optionals.

This
```swift
var foo: [String]
var foo: [String: Int]
var foo: ((Int) -> Void)?
```

Not this
```swift
var foo: Array<String>
var foo: Dictionary<String, Int>
var foo: Optional<(Int) -> Void>
```

[![SwiftFormat: typeSugar](https://img.shields.io/badge/SwiftFormat-typeSugar-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#typeSugar)

**[⬆ back to top](#style-guide)**

### Unused arguments
In closures, unused arguments should be defined as `_`.

This
```swift
request { _, data in
    self.data += data
}
```

Not this
```swift
request { response, data in
    self.data += data
}
```

[![SwiftFormat: unusedArguments](https://img.shields.io/badge/SwiftFormat-unusedArguments-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#unusedArguments)

**[⬆ back to top](#style-guide)**

### Void return
- Prefer `-> Void` over `-> ()` when used for return in declarations.
- Prefer `() ->` over `Void ->` when using Void as a parameter in declarations.

This
```swift
let foo: () -> Void
let bar: () -> Void
```

Not this
```swift
let foo: () -> ()
let bar: Void -> Void
```

[![SwiftFormat: void](https://img.shields.io/badge/SwiftFormat-void-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#void)

**[⬆ back to top](#style-guide)**

### Closing Parenthesis
Position closing paren on same line as last argument

This
```swift
func multilineFunction(foo: String,
                       bar: String) -> String {}
```

Not this
```swift
func multilineFunction(foo: String,
                       bar: String
) -> String {}
```

`closingparen same-line` option.
[![SwiftFormat: wrapArguments](https://img.shields.io/badge/SwiftFormat-wrapArguments-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#wrapArguments)

**[⬆ back to top](#style-guide)**

#### Line wrapping
Each line should have a maximum column width of 120 characters. 
(Corresponding SwiftLint rule - warning: 120, error: 200)

[![SwiftFormat: wrap](https://img.shields.io/badge/SwiftFormat-wrap-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#wrap)

**[⬆ back to top](#style-guide)**

### Don't use Yoda conditionals
Keep constant values on the right-hand-side of expressions.

This
```swift
foo != true
```

Not this
```swift
true != foo
```

[![SwiftFormat: yodaConditions](https://img.shields.io/badge/SwiftFormat-yodaConditions-7B0051.svg)](https://github.com/nicklockwood/SwiftFormat/blob/master/Rules.md#yodaConditions)

**[⬆ back to top](#style-guide)**
