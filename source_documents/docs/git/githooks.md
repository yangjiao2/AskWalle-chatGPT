---
title: Githooks
navTitle: Githooks
---

## Githooks

The `glass-app` repository utilizes the following [githooks](https://git-scm.com/docs/githooks) to boost productivity and quality.

### `prepare-commit-msg`

| Name              | Description                                                                                                                                                                                                                                                                                                                                        |
|-------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| pre-commit-msg.py | Prefixes the commit message with a JIRA ticket identifier if the identifier can be located with the regex `([A-Z]{1,10}-\d+)`. For example, if the branch is named `john/platform/CEPG-47635-yikes`, `CEPG-47635` will be inserted into the commit message template automatically. This does not work with `git commit -m "..."` and some Git GUIs. |

### `pre-commit`

| Name                                     | Description                                                                                                                                                                                                                                                                 |
|------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| check-fixtures.sh                        | An efficient fixture (`response.json`) cleanup script. The script performs organizes a `response.json` file in a way that avoids common merge conflicts. If changes are made to fixture files, the pre-commit fails with the expectation of including them in a new commit. |
| enforce-branch-name.sh                   | Enforces that a branch name must contain lowercase letters in n-1 path components to prevent macOS file system issues (e.g. `component/component/JIRA-123`).                                                                                                                |
| glass-verify-pre-commit.sh | An aggregate pre-commit command, `glass verify pre-commit`, that performs numerous verifications:<br/><br/>- CODEOWNERS<br/>- Configuration Validation<br/>- SwiftLint<br/><br/>See [Change Verifications](./change-verifications.md) more more info.                                                                                       |
| scan-file-size.sh                        | Prevents large files from being mistakenly committed to the repo.                                                                                                                                                                                                           |
| scan-strings.sh                          | Prevents prohibited files such storyboards from being added to the repo.                                                                                                                                                                                                    |

### `pre-push`

| Name                    | Description                                                               |
|-------------------------|---------------------------------------------------------------------------|
| verify-xcode-project.sh | Sorts all Xcode project files lexographically to prevent merge conflicts! |
