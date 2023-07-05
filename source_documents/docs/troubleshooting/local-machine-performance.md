---
title: Local Machine Performance
navTitle: Local Machine Performance
---

# Problem Statement
Life as iOS engineer in a large codebase can be... slow. Clean builds can take longer than it takes to make coffee (using your preferred pour-over process). Running the test suite(s) locally can take much longer with UI tests. Even just opening Xcode can cause indexing to saturate every CPU making simple tasks like writing Slack messages a struggle for an indefinite time. CI builds continue to improve but unexpected failures and slow build times leave room for future improvements.

# Paths for Improvement
## Xcode Indexing
### Background Priority
Xcode's default behavior of crippling the machine until all indexing is complete is not ideal. Other large engineering teams have run into this and addressed it using options like [index importing](https://github.com/MobileNativeFoundation/index-import) but an immediate option available is in `scripts/toggle-indexing-priority.sh`. The script uses `renice` to [alter the Xcode indexing process priority](https://apple.stackexchange.com/questions/24998/can-i-manually-limit-the-cpu-used-by-a-process/141520#141520) once Xcode is running. The indexing process ID changes with each launch of Xcode so the script has to be re-run each time you want it to lower the priority.

### Enable Indexing Logging
There are [lots of user defaults in Xcode](https://github.com/ctreffs/xcode-defaults) to enable more visibility into indexing progress. Like these to enable progress reporting and logs:

```bash
defaults write com.apple.dt.Xcode IDEIndexerActivityShowNumericProgress -bool YES
defaults write com.apple.dt.Xcode IDEIndexShowLog -bool YES
```

## High Performance Swift
The best way to reduce compile times is to write better Swift! There are [tons](https://medium.com/rocket-fuel/optimizing-build-times-in-swift-4-dc493b1cc5f5) of resources available online about writing Swift code that compiles faster. The Swift language repo contains an [excellent doc about optimization tips](https://github.com/apple/swift/blob/main/docs/OptimizationTips.rst). There's also [internal Confluence pages](https://confluence.walmart.com/pages/viewpage.action?pageId=315477211) with great examples.

Hopefully in the future long compile times can either become part of PR checks or nightly CI reports. Until then, using the tips in some of the articles above can help track down places where small changes can save significant time for every developer.