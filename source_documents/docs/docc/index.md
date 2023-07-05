# DocC

Use [DocC](https://developer.apple.com/documentation/docc) to document your module's API.

## DocC Websites

DocC websites are published nightly to [Github pages](https://gecgithub01.walmart.com/pages/walmart-ios/glass-app/#modules) using the `glass publish-docs` command. All DocC catalogs in the repo are published and will be listed under the [Modules](https://gecgithub01.walmart.com/pages/walmart-ios/glass-app/#modules) heading of the docs index page. See [Enabling DocC](#enabling-docc) for information on how to set up a DocC catalog for your module.

## Enabling DocC

1. In Xcode, add a new file to your module's Xcode project.
2. Select Documentation Catalog.
3. Make sure that the name of the documentation catalog matches the name of your module's build scheme.
