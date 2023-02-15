# Creating a store-specific custom app

A custom app is an app that you or a developer builds exclusively for your Shopify store (unlike a public app, which is built to be used by many stores). You can use custom apps to add features to your Shopify admin, access your store's data directly using Shopify's APIs, or extend your online store to other platforms using the Storefront API. Sometimes such a custom app is referred to as a "private app".

## Prerequisites

1. [Enable custom app development from the Shopify admin](https://help.shopify.com/en/manual/apps/custom-apps#enable-custom-app-development-from-the-shopify-admin)
2. [Create and install a custom app](https://help.shopify.com/en/manual/apps/custom-apps#create-and-install-a-custom-app)
3. :warning: Have a secure copy of the Admin API access token from the previous step.

## Basic concepts

A store-specific custom app does not use the OAuth process to authenticate - it uses the secrets established during the app creation and install process to access the API.  As a result, there are no sessions to be retrieved from incoming requests and stored in a database, etc.

When initializing `shopifyApi` in a custom app, set the `isCustomStoreApp` configuration property to `true`, and set the `apiSecretKey` to the **Admin API access token** obtained during the installation process (step 2 in the [prerequisites](#prerequisites)).

## Example

### Initialization

```js
import "@shopify/shopify-api/adapters/node";
import { shopifyApi, LATEST_API_VERSION, Session } from "@shopify/shopify-api";
import { restResources } from "@shopify/shopify-api/rest/admin/2023-01";

const shopify = shopifyApi({
  apiKey: "App_API_key",
  apiSecretKey: "Admin_API_Access_Token", // Note: this is the API access token, NOT the API Secret Key
  apiVersion: LATEST_API_VERSION,
  isCustomStoreApp: true,                     // this MUST be set to true (default is false)
  scopes: [],
  isEmbeddedApp: false,
  hostName: "my-shop.myshopify.com",
  // Mount REST resources.
  restResources,
});
```

> **Note** The `apiSecretKey` is **NOT** set to the API secret key but to the **Admin API access token**.

### Making requests

API requests, either using `shopify.clients.Rest` or `shopify.clients.Graphql`, or using the REST resources, require a `session` parameter.  Since there are no sessions in a store-specific custom app, a `session` parameter needs to be created.

To create the session object, the `id`, `state` and `isOnline` properties must be populated but will be ignored.  Only the `shop` parameter is required when making the API requests from a store specifc custom app and the value must match the stop on which the custom app is installed.

The library provides a utility method to create such a session, `shopify.session.customAppSession`.

```js
const session = shopify.session.customAppSession("my-shop.myshopify.com");

// Use REST resources to make calls.
const { count: productCount } = await shopify.rest.Product.count({ session });
const { count: customerCount } = await shopify.rest.Customer.count({ session });
const { count: orderCount } = await shopify.rest.Order.count({ session });

console.log(
  `There are ${productCount} products, ${customerCount} customers, and ${orderCount} orders in the ${session.shop} store.`
);
```
