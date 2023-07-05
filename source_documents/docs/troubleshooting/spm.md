# SPM Troubleshooting

SPM package resolution issues can manifest as

- "Server SSH Fingerprint Failed to Verify", or 
- "There is no XCFramework found at '.../Developer/Xcode/DerivedData/Walmart-abc/SourcePackages/artifacts/glass-app/XYZ.xcframework'." or 
- "Missing package..."

<img src="images/ssh-fingerprint.png" alt="ssh fingerprint screenshot" width="400"/>

Double click on error message and click "Trust" button to trust GitHub certificate .

<img src="images/authenticity-trust.png" alt="authenticity trust screenshot" width="400"/>

You may have to reset and/or resolve the packages in Xcode after this.

<img src="images/resolve-packages.png" alt="resolve packages screenshot" width="200"/>

If there are still issues, perhaps this terminal command will also give some useful output to resolve the issue.

```
xcodebuild -resolvePackageDependencies -destination 'generic/platform=iOS' -workspace Walmart.xcworkspace -scheme Walmart
```
