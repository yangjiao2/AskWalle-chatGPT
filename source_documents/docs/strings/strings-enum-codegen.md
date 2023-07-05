---
title: Glass iOS Strings - LocalizedString enum CodeGen
navTitle: Glass iOS Strings - LocalizedString enum CodeGen
---

# Glass iOS - LocalizedString enum CodeGen

GlassCLI provides a tool to generate an `LocalizedString` enum to represent a localized strings file. See [here](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Plugins/Scanner/Scanner/Sources/Utility/LocalizedString.swift) for an example. 

## Setup Instructions 

To code-generate an enum for your localized strings, follow the steps below:

1. Add a new "Run script phase" to your module (framework, not resource bundle). Ensure that it is ordered **before** the "Compile Sources" phase. Name the phase something meaningful such as "Codegen LocalizedString"
2. Add the following to the script:

	```
	cd "${SRCROOT}"/../../ # <-- This should cd into the root directory of the repo
	./scripts/localized-string-codegen.py
	```
3. Add your `en` localization file as an _input_ file. e.g.
	
	```
	${SRCROOT}/Scanner/Sources/Resources/en.lproj/Localizable.strings
	```

4. Add your `LocalizedString.swift` as an output file. Do **not** use `SRCROOT`, instead use `DERIVED_FILE_DIR` followed by the path to the `LocalizedString` file. e.g.

	```
	$(DERIVED_FILE_DIR)/Scanner/Sources/Utility/LocalizedString.swift
	```
	
	The run-script phase should look something like this after the previous steps:
	
	<img src="resources/localized-string-code-gen-example.png" alt="missing image" width="1000"/>
5. Add an extension onto `Bundle` named `resources` that returns your module's resource bundle containing the localized strings file. e.g.

	```swift
	private final class Scanner {}

	extension Bundle {
	
	    static var scanner: Bundle {
	        moduleBundle(for: Scanner.self)
	    }
	
	    static var resources: Bundle {
	        scanner
	    }
	}
	```

6. Build your module to code-generate the `LocalizedString` enum. The code-generation script only runs if you add a new localized string to the file.

7. Done.

## Tutorial complete

###### [Return to index](index.md)

