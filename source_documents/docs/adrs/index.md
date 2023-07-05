# Architecture Decision Records (ADRs)

This is a decision record for choices we've made along our journey. The goal here is to share the problem we faced, the options we considered, and the direction we took and why. If the available options change or the problems are no longer affecting us, please propose a change!

This is based on the work of [Michael Nygard](https://cognitect.com/blog/2011/11/15/documenting-architecture-decisions.html) and the ADR process in the [Walmart Engineering Community](https://engineering.walmart.com/adrs).

# Log

- [Architecture Design Pattern (MVVM+C)](architecture-design-pattern.md) - Determine a design pattern for writing highly testable code
- [Mobile Platform](https://confluence.walmart.com/display/GPONEW/Walmart+Mobile+Platform) - Determine the mobile architecture for the unified Walmart mobile application experience
- [UI Construction Guidelines](ui-construction-guidelines.md) - Determine if storyboards should be used
- [Shared UI Components](shared-ui-components.md) - Determine a solution for sharing feature-specific UI components across features 
- [Multi Market Mono Repo](multi-market-mono-repo.md) - Determine a solution for hosting market specific (US, Canada, Mexico, etc.) iOS code in Glass mono repo
- [Generic Plugins](generic-plugins.md) - Determine a solution for converting the current plugins into market-agnostic plugins
- [Platform Code Organization](platform-code-organization.md) - Determine how and where Platform capabilities should live.
- [Google Ad Manager SDK integration](google-ad-manager-display-ad-integration.md) - Defines solution of Display Ad for international market via Google Ad Manager
- [User Idle State Notifying](user-idle-state-notifying.md) - How should plugins be notified that the user has gone idle while using the app.
