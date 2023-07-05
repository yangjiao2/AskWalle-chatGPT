---
title: Contribution Guidelines
navTitle: Code Review Guidelines
---

# Code Review Guidelines

- Take time to read through _every_ line of the code that your team owns.

- Run code when it's unclear how the code will impact the user (both customers and users of libraries like platform)

- View xcodeproj file changes locally when it is unclear what changes were made (build settings)

- Is there a JIRA ticket? If not, is the reason for the change clearly described in the PR description?

- Do you possess the domain knowledge to properly review the code? If not, consider asking someone more familiar with the code to assist with reviewing.

## What to look for

### Best Practices

- Are the team's [contribution guidelines](contribution-guidelines.md) and best practices being followed?

- Are iOS development best practices being followed?

- Are general software engineering best practices being followed?

### Functionality

- Does the code do what the author intended?

- Does the code address the requirements in the ticket?

    - Is the problem being solved in the PR clear? 
    
    - Did the author describe the problem and solution in the description?
    
    - Did the author include screenshots or videos for UI changes?

- Are there edge cases that should be addressed?

- Is the code making proper use of APIs (system, platform, 3rd party)?

- Are there any developer oversights?
    - Are callbacks being invoked?
    
    - Is memory properly managed (no retain cycles)?
    
    - Is there any potential for race conditions or deadlocks?

### Complexity

- Is the code easy to read or can it be simplified? If the code is unclear consider asking the author to simplify (changing the implementation) or document the code (code comments).

- Is the code over-engineered?
    - Is the code more generic than it needs to be?

    - Is the code adding functionality that is not currently needed? 

### Documentation

- Are all public APIs clearly and concisely documented?
    - If a framework has a PlatformAPI or PluginAPI associated with it, the PlatformAPI/PluginAPI should be fully documented. Public members of the plugin implementation do not require documentation as that should be covered by the PlatformAPI/PluginAPI documentation.

- Are any comments required to explain implementation? 
    - Consider whether the code can be simplified but for the cases when it cannot, the author should write some comments.

- Is a migration guide needed for a breaking change?

### Naming

- Do names clearly communicate their intent?

- Is the naming consistent with the other parts of the API or file?

- Does the naming follow the [Swift API design naming guidelines](https://www.swift.org/documentation/api-design-guidelines/#naming)?

### Code Placement

- Is code added to the proper module?
    - Is feature code being added to the correct feature plugin?
    
    - Is platform code being added to the correct module or plugin?

- Does this code belong in platform? 

### Tests

- Have test cases been added to cover new functionality?

- Are the tests useful and making correct assertions about the code's functionality?

- Will the tests fail when functionality they are testing is broken?

## Writing Feedback

- Be courteous and respectful

- Make comments about the _code_ and not the _developer_

- Remember that we're a team. Use terminology like "we" and avoid singling out a developer. Examples: "have we considered...", "should we think about...", "can we avoid..."

- Use open-ended questions instead of making opinionated statements

- Offer suggestions without insisting those suggestions are the only way to proceed
    
- Label priority of comments to help the author prioritize their work. (The exact label names do not matter.) Examples:
    - `Nit`: Minor impact to the code
    
    - `Suggestion`: An improvement or alternative approach to consider
    
    - `Follow-up`: Out of scope for the PR, but something that should be addressed in a future PR
    
    - `Required`: Should be addressed before PR is merged

# Reference

- https://google.github.io/eng-practices/review/reviewer/
- https://stackoverflow.blog/2019/09/30/how-to-make-good-code-reviews-better/
