---
title: Securing sensitive keys
navTitle: Securing sensitive keys
---

# Securing sensitive keys

This section provides way to secure sensitive information/keys used in Glass. There are some third party SDKs used in the app (like Braze, PerimeterX, etc) which requires a separate API Key. All these keys should be added as **Preprocessor Macros** so that we avoid risk of exposing sensitive information as plain text in swift files. 

Follow these steps to secure senstive information for each of the configurations

## Debug Builds

  The keys used in **Debug** configuration (used in non-Testflight/Appstore builds) should be defined as plain text under *GCC_PREPROCESSOR_DEFINITIONS* in the `SecureKeys-Debug.xcconfig` file, make sure to updated it in the necessary markets:

  Example: *MyTPSDK* using secure key *123WASD* should be defined as 
  
```json
MyTPSDK="123WASD"
```
  * USA Market: [markets/usa/SecureKeys-Debug.xcconfig](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/markets/usa/Configuration/SecureKeys-Debug.xcconfig)
  * CA Market: [markets/ca/SecureKeys-Debug.xcconfig](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/markets/canada/Configuration/SecureKeys-Debug.xcconfig)
  * MX Market: [markets/mx/SecureKeys-Debug.xcconfig](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/markets/mexico/Configuration/SecureKeys-Debug.xcconfig)

## Release Builds

  The keys used in **Release** configuration (used in Testflight/Appstore builds) should be encrypted. Follow [these](docs/secure-keys/encrypt-sensitive-keys.md) steps to generate the encrypted value for your secure key. Add this encrypted value in submission_candidate.properties and quick_submission_candidate.properties files for the locale you want to make it available. 
### Examples:
  1.[submission_candidate.properties CANADA](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/e221af334ffed4607bed6485e4e040bbeb7b3e00/env/canada/submission_candidate.properties)
  2. [submission_candidate.properties USA](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/e221af334ffed4607bed6485e4e040bbeb7b3e00/env/usa/submission_candidate.properties)
  3. [submission_candidate.properties MEXICO](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/e221af334ffed4607bed6485e4e040bbeb7b3e00/env/mexico/submission_candidate.properties)


  Example: *MyTPSDK* using secure key *123WASD* will have an encrypted value as
  
  ```yaml
  ENC[RH0g6FONqjzeMJdE/TESJQ==]
  ```
  
  Add the encrypted value in all *.looper.xxx.yml* files as 
  
  ```yaml
  MyTPSDK_KEY: ENC[knf6+0/z7sROlknvYUuiQw==]
  ```

Now, Go to [script-helpers.sh](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/scripts/script-helpers.sh#L22) and add checks for MyTPSDK**_KEY** in [this](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/33b16fcd7a310b4b8572b04b4ba64f8593731d98/scripts/script-helpers.sh#L22-L38) method. This step is to ensure we catch any issues with the encrypted keys so that it won't impact the functionalities in release builds.
  
  ```yaml
  function check_encrypted_keys()
  ```

Define your secure key under *GCC_PREPROCESSOR_DEFINITIONS* in [SecureKeys-Release.xcconfig](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Configuration/SecureKeys-Release.xcconfig)
  
```json
MyTPSDK_KEY="MyTPSDK_PROD"
```

  * USA Market: [markets/usa/SecureKeys-Release.xcconfig](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/markets/usa/Configuration/SecureKeys-Release.xcconfig)
  * CA Market: [markets/ca/SecureKeys-Release.xcconfig](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/markets/canada/Configuration/SecureKeys-Release.xcconfig)
  * MX Market: [markets/mx/SecureKeys-Release.xcconfig](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/markets/mexico/Configuration/SecureKeys-Release.xcconfig)


> **NOTE:** Ensure to maintain the same naming convention as we have for other keys.


## Accessing Preprocessor Key

Follow these steps to access your secure key

1. Declare your constant in [SecureKeys.h](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/markets/usa/Walmart/SecureKeys/SecureKeys.h)

  ```objective-c
  extern NSString* const kMyTPSDKKey;
  ```

2. Define your constant in [SecureKeys.m](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/markets/usa/Walmart/SecureKeys/SecureKeys.m)

  ```objective-c
  NSString *const kMyTPSDKKey = @OS_STRINGIFY(MyTPSDK_KEY);
  ```

3. Add your key in [WalmartKeys.swift](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/markets/usa/Walmart/SecureKeys/WalmartKeys.swift#L14)

  ```swift
  public let myTPSDKKey = kMyTPSDKKey
  ```

In your plugin module, access the key as 

```swift
  
// Access your secure key
let secureKeyProviding: SecureKeyProviding = container.get()
let _ = secureKeyProviding.myTPSDKKey
```

> **Note:** Make sure `SecureKeyProviding` is registered on container before accessing

```swift
container.register(SecureKeyProviding.self) { WalmartKeys() }
```
