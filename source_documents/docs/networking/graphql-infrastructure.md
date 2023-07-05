---
title: Networking - GraphQL infrastructure
navTitle: GraphQL infrastructure
---

# GraphQL infrastructure

We use the [Apollo iOS SDK](https://github.com/apollographql/apollo-ios) to generate Swift versions of the objects defined in your GraphQL schema and to decode these objects from JSON service responses. This guide will walk you through the process of setting up Apollo code generation in a new feature plugin.

## Prerequisite

Apollo code generation is done by communicating with a node binary that is downloaded during `glass environment setup`. If you have not run this command, run it before proceeding in the tutorial.

You can also download the binary directly by executing `glass apollo setup`.

## Make a 🏡

First off, we need to create a framework target within your plugin's project where your GraphQL code will live.

1. Click on Plugin Xcode Project.
2. Hit + at bottom left to add target.
3. Select Framework in iOS.
4. Name it {Feature}GQL (e.g. MyPluginGQL).

Then we need to configure it with the Framework.xcconfig.

1. Click on your project under Projects in the left pain.
2. Under Configurations, expand Debug, Profile and Release and assign Framework.xcconfig to your newly created target.
3. Select your newly created target in the left pane.
4. Go to Build Settings.
5. Click Customized at top (Between Basic and All).
6. Click in settings area, select all (cmd + a), hit delete.

Next, we need to link this new target to the main app target.

1. Select app project in the left pane (e.g. Walmart).
2. Click on Build Phases at the top.
3. Under Link Binary With Libraries, Hit +, select your `<Plugin>`GQL framework.
4. Under Dependencies, Hit +, select your `<Plugin>`GQL framework.

Finally, we need to create the directories for where your GraphQL files will actually live.

```sh
mkdir -p Plugins/MyPlugin/MyPluginGQL/Sources/GraphQL/{Queries,API}
```

`Queries` is where you will create GraphQL queries and `API` is where Apollo will put its generated Swift API based on your schema and queries. 

Go ahead and add the `Sources` folder to your Xcode project, making sure that "Create Groups" is selected. Make sure to select your GQL target.

## Download your schema

Before we can really get going, we need to download the schema from our GraphQL service that will serve as Apollo's source of truth. Most feature teams will use the Glass federated GraphQL service, which is available at the endpoints listed below. Using the `--environment` (`-e`) convenience downloads the schema from the _shadow_ endpoint.

**Tip**: Use `glass apollo update -p MyPlugin -e [environment]` to also perform code generation! This is useful when you want to generate new types from the schema.

  - **Development:** 
    ```
    glass apollo download-schema -p MyPlugin -e development
    ```
    * Shadow: http://ce-gateway.ce-orchestration.shadow-dev.k8s.nonprod.walmart.com/graphql
    * Regular: http://ce-gateway.ce-orchestration.dev.k8s.nonprod.walmart.com/graphql

  - **Staging:**

    ```
    glass apollo download-schema -p MyPlugin -e staging
    ```

    * Shadow: http://ce-gateway.ce-orchestration.shadow-stage.k8s.nonprod.walmart.com/graphql
    * Regular: http://ce-gateway.ce-orchestration.stage.k8s.nonprod.walmart.com/graphql

  - **Teflon:** 
    ```
    glass apollo download-schema -p MyPlugin -e teflon
    ```

    * Shadow: http://ce-gateway.ce-orchestration.shadow-teflon.k8s.nonprod.walmart.com/graphql
    * Regular: http://ce-gateway.ce-orchestration.teflon.k8s.nonprod.walmart.com/graphql

  - **Production:** 
    ```
    glass apollo download-schema -p MyPlugin -e production
    ```

    * Shadow: http://ce-gateway.ce-orchestration.shadow-prod.k8s.prod.walmart.com/graphql
    * Regular: http://ce-gateway.ce-orchestration.k8s.prod.walmart.com/graphql

The script can also download a schema from an arbitrary URL:

```sh
glass apollo download-schema -p MyPlugin --schema "http://my.service.com/graphql"
```

**Important**: You must specify either a schema url (`--schema`) or an environment (`-e`), but not both!

  * You can find more URLs and details about the Shadow Gateway in this page:
    https://confluence.walmart.com/pages/viewpage.action?pageId=840789289

After running the script, you should now have a `schema.graphqls` file at located at `Plugins/MyPlugin/MyPlugin/Sources/GraphQL`. It is not necessary to add this file to your Xcode project, but you may if you wish!

```text
******************************** Schema File Migration **************************************
The schema file has migrated from schema.json to schema.graphqls. Commit the removal of
schema.json and the addition of schema.graphqls. You may also need to remove the schema.json
reference from the Xcode project and add schema.graphqls.
*********************************************************************************************
```

**Migration notice**: Previously, a JSON version (`schema.json`) of the schema was downloaded. With the migration to the Apollo Swift Package, a `schema.graphqls` is downloaded instead. If you encounter a Plugin that still uses a `schema.json` file, follow the migration warning above and delete the `schema.json` file while keeping the new `schema.graphqls` file.

**Important**: You should always commit the schema file you used to generate the GraphQL code. In addition, only the production schema can be committed to development.

[Troubleshooting GraphQL Schema Download](https://gecgithub01.walmart.com/walmart-ios/glass-app/blob/development/docs/troubleshooting/graphql-schema-troubleshooting.md)

### Which "version" should merge into development?

Because the intention of the `development` branch is to be able to cut releases from it, only schema generated from the **Production GraphQL** service should merge into `development`. If you are working on a new contract that is in one of the lower environments, like Staging for example, then changes should merge into a feature branch (or wait) until the new contract has been deployed to the Production service.

## Code generation


To generate GraphQL Swift code for a given plugin, execute the following command where MyPlugin is the name of your plugin:

```
glass apollo codegen -p MyPlugin
```

You can force code generation by executing `glass apollo codegen -p MyPlugin` from the root of the repository.

## Git

All code generated swift files, `.graphql` files, `operations.json` file, and the schema file MUST be committed. Essentially, all files in your GraphQL target folder should be committed. Failure to do so will cause your build to fail.

## Next steps

Now that we've got the logistics out of the way, everything from here on out should be smooth sailing ⛵️

Head over to the [GraphQL client documentation](graphql-client.md) to learn how to make a request to your GraphQL service.
