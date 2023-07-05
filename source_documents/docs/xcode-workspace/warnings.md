---
title: Warnings
navTitle: Warnings
---

# Warnings

All build warnings are **strongly** discouraged in the Glass workspace because they are distracting and if there are too many, which can occur in a corpulent monorepo, they will mask **serious** issues during development.

## Treat warnings as errors

While warnings do not generate errors locally, they will generate errors in CI since `SWIFT_TREAT_WARNINGS_AS_ERRORS` is set to `YES`. Some projects may generate warnings today, but their warnings are slowly being resolved and will generate errors in the future.

## Deprecations

In a small Xcode project, deprecations using `@deprecated` annotations are useful because they generate a small number of warnings that can be resolved quickly over time. However, in a monorepo Xcode project, a simple deprecation can engender hundreds, if not thousands of warnings. Therefore, in order for meaningful warnings to be visible to developers working on features, `@deprecated` annotations are disallowed in the Glass workspace.

### Recommendation

If your team needs to deprecate an API, the deprecation should be tracked internally by your team while the Platform team develops a tech debt/deprecation calendar tooling. Track this [ticket](https://jira.walmart.com/browse/CEPG-107335) to know when such tooling is available. Additionally, if the migration of a deprecated API is _simple_, please perform the migration for the consumers of the API rather than relying on them to perform it at a later date.
