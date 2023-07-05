---
title: Glass iOS Tech Debt Calendar
navTitle: Glass iOS Tech Debt Calendar
---

# **Glass Automated Tech Debt Calendar**
- [**Glass Automated Tech Debt Calendar**](#glass-automated-tech-debt-calendar)
  - [**Overview**](#overview)
  - [**The TechDebt Registry**](#the-techdebt-registry)
    - [**Evaluating the Registry**](#evaluating-the-registry)
    - [**Adding an Entry**](#adding-an-entry)
  - [**Process for New Entries**](#process-for-new-entries)
    - [**Enforcement Date Expectations**](#enforcement-date-expectations)
  - [**What if my plugin is in violation?**](#what-if-my-plugin-is-in-violation)
    - [**Requesting an Exception**](#requesting-an-exception)
  - [**Contributing**](#contributing)
    - [**Implementing a Rule Evaluator**](#implementing-a-rule-evaluator)

## **Overview**
The Glass iOS Monorepo is massive and ever changing! Thus, Platform needs a way to effectively drive changes across the repo without incurring the churn and cost of massive PR's that touch nearly every plugin. 

Introducing the GlassCLI driven Automated TechDebtCalendar! In it's simplest form, the TechDebtCalendar is a json based registry where rule entries are saved and then evaluated using rule evaluators written in Swift. 

A description of why we don't use deprecations warnings can be found [here](../xcode-workspace/warnings.md).

**Currently the supported rule types are:**
- **SwiftLint**:
  - **Standard Rules**: The standard rule set that comes Packaged in SwiftLint.
  - **Custom Rules**: Regex based custom rules.

Don't see the required rule type here? Ping @glass-platform-ios to get a ticket created, or better yet checkout the [contribution section](#contributing) and add support for it to GlassCLI.

## **The TechDebt Registry**
The source of truth for the Automated TechDebt Calendar is the TechDebtRegistry.json file. There are two types of objects that make up the registry:
- Entries:
An entry in the calendar consists of all the details needed to enforce a tech debt rule.

- RuleEvaluator Configurations:
RuleEvaluator configurations consist of the necessary configuration options to make a RuleEvaluator work. For example, the base disabled rule set for SwiftLint.
### **Evaluating the Registry**
It's very easy to evaluate the state of the Glass iOS Monorepo in relation to the registry. But first, there is one thing to be aware of. The registry can be evaluated on active rules only or be configured to take upcomming entries into account. 
Violations only:
```swift
glass tech-debt evaluate --mode violationsOnly
```

Warnings and Violations:
```swift
glass tech-debt evaluate --mode warningsAndViolations
```

### **Adding an Entry**
Entries can be added to the TechDebtRegistry.json either manually or by using the convenience command in GlassCLI.

**Steps:**
1. Run `glass tech-debt add-entry`
    - This will prompt you to select a rule type. If the needed type is not present, you will need to implement the RuleEvaluator in GlassCLI and register it for the rule type. See the [contribution section](#contributing) for details.
2. Open the TechDebtRegistry.json file.
3. Fill out the new entry with the required details.
4. Save and close the TechDebtRegistry.json file.
5. Run `glass tech-debt generate-associated-files`
   - This is a crucial step as it triggers the generation of files that the rule evaluators require. For example, the swiftlint.yml files. 

## **Process for New Entries**
All new entries to the TechDebtCalendar **MUST** follow the below process:
1. The need to introduce a new rule is determined (This will likely, but not necessarily, be done by the Platform team)
2. The new rule is added to the registry with a relevant AND appropriate enforcement deadline. 
   - See the [enforcement deadline expectations](#enforcement-date-expectations) for more details
3. Write a migration guide for the new rule and include it in the [docs/tech-debt/migration-guides](https://gecgithub01.walmart.com/walmart-ios/glass-app/tree/development/docs/tech-debt) directory. **THIS IS REQUIRED!**
4. Open a pull request to include the new entry and any associated changes to it's RuleEvaluator's generated files.
   - This pull request **MUST** include any necessary migration guides to support the teams that will be making change
5. Work with Platform to get this pull request approved, ALL RULE ENTRIES WILL REQUIRE PLATFORM APPROVAL.
6. Once approved and before merging the pull request, make an announcement in the #glass-platform-ios-announcements slack channel explaining the rationale for the new entry, the deadline and linking out to the associated migration guide.

### **Enforcement Date Expectations**
**TEAMS MUST BE GIVEN ADEQUATE TIME TO COMPLETE ANY CHANGES!** For this reason, a minimum of 3 weeks will be required for all new entries. The complexity of the changes required must also impact the enforcement deadlines.

## **What if my plugin is in violation?**
If a plugin fails to fix any rule violations before their enforcement deadline, all pull requests changing that plugin will be blocked. For this reason, it is paramount to get ahead of any required changes in order to prevent disruptions in development work. 

### **Requesting an Exception**
The tech debt registry supports the creation of exceptions for rule entries. These exceptions will require Platform approval and there fore must be adequately justified.

While exceptions are a feature of the TechDebtRegistry they are not permanent and will have an associated expiration. If the violation(s) are not fixed by the exceptions expiration date, pull requests into the  plugin will once again be blocked. For this reason, it is best to plan ahead for tech debt and keep your plugin(s) from being in violation of the Automated TechDebt Calendar's rules.

To request an exception, reach out to @glass-platform-ios and provide the following details:
   - What rule is an exception needed for? 
   - Why was your team not able to come into compliance with the rule before the enforcement deadline? 
   - How long of an extension is needed? 
   - What Plugin(s) is the extension needed for?

## **Contributing**
### **Implementing a Rule Evaluator**
Rule evaluators are (typically) simple wrappers around an existing static analysis tool, however they do not have to be. In order to create a new one, all one must do is implement an object that conforms to the follow protocol:
```swift
protocol RuleEvaluator {
    func evaluate() async throws -> TechDebt.Registry.EvaluationResult
    func generateAssociatedFilesIfNeeded() async throws
}
```
Once the object is created you can wire it up to the Registry by adding it to the list of evaluators in the `TechDebt.RegistryManager` object in GlassCLI.

While there is no technical limit to a `RuleEvaluator`'s capabilities, it's important to follow the below guidelines.
1. Even though these will only be run on CI, it is important to keep things as fast as possible. 
2. Keep Evaluator's specialized, evaluators should not be jack of all trades and should only evaluate a limited set of rule types.
3. If you're evaluator requires tertiary files (ex swiftlint.yml) to function, these must be generated from the Registry during calls to your evaluator's implementation of `generateAssociatedFilesIfNeeded()`. Evaluators MUST not support the manual maintenance of these files. 