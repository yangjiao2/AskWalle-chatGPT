---
title: Third Party Code
navTitle: Third Party Code
---

# Third Party Code

Usage of third party code in Glass is **not** encouraged. However, if there is a strong business need, the third party code, including, but not limited to, SDKs and frameworks can go through an evaluation process and get a signoff from all the stake holders. These [guidelines](https://confluence.walmart.com/display/GPONEW/Guidance+for+3rd+Party+Code+in+Glass+apps) can be referred to help make right decision while evaluating any third party code.   

## Upgrading frameworks

If you’ve modified the [Cartfile](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/Cartfile), or you want to update to the newest versions of each framework (subject to the requirements specified), simply run the this command.

```
glass environment setup --update
```

### Prebuilt binaries

To upgrade one of the locally hosted prebuilt binaries listed in `Cartfile`, follow the steps below:

1. Upload and deploy the prebuilt binary using the [`static-artifact-deployer`](https://gecgithub01.walmart.com/walmartlabs-wmusiphone/static-artifact-deployer).
2. Add or update the framework's local [binary specification file](https://github.com/Carthage/Carthage/blob/master/Documentation/Artifacts.md#example-binary-project-specification) that is located in the root of the repo in `BinaryDependencySpecification/`.
3. Run `glass environment setup --update`.
4. Commit changes to the Cartfile and Cartfile.resolved files.

## Frameworks

Following are the third party SDKs used in Glass.

- [Appboy/Braze](braze-framework.md)
- [Apollo](apollo-framework.md)
- [Charts, Swift-algorithms, Swift-numerics](charts-framework.md)
- [Digimarc](digimarc-framework.md) [ ⚠️ **TBU**]
- [Firebase](firebase-framework.md)
- [Flipp/SFML](flipp-framework.md) [ ⚠️ **TBU**]
- [Fraud Force](fraud-force-framework.md) [ ⚠️ **TBU**]
- [PerimeterX](perimeterx-framework.md)
- [Quantum Metric](quantum-metric-framework.md) [ ⚠️ **TBU**]
- [Threatmetrix](threatmetrix-framework.md) [ ⚠️ **TBU**]
- [Viewinspector](viewinspector-spm.md) [ ⚠️ **TBU**]

