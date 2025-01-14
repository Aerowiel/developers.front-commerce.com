---
id: using-magento-configuration
title: Using Magento Configuration
---

In Magento's ecosystem, for every feature there are configurations (in the Stores > Settings > Configurations page). Each one of them will influence how your shop runs and you will need to have access in your Front-Commerce application to give merchants more autonomy.

As a developer this means that you will need a way to fetch them. In Front-Commerce you have two possibilities:

- Reuse Magento's GraphQL schema (see [Magento2 GraphQL schema](./graphql.html))
- Fetch configurations with a REST endpoint

In this guide, we will focus on the REST endpoint.

<blockquote class="warning">This feature is available with `front-commerce` >= 2.1.0 and `front-commerce/magento2-module` >= 2.0.0.</blockquote>

## Fetch configurations from `/frontcommerce/storeConfigs` Magento endpoint

The REST endpoint is a custom one added by Front-Commerce's Magento module.

```
http://magento.test/rest/V1/default/frontcommerce/storeConfigs?keys[]=design/header/welcome
```

This endpoint is protected and only users having the permission `Magento_Config::config` have the right get data from this endpoint. In Front-Commerce everything is already setup so that you don't need to care about this permission. But if you want to test this endpoint manually (with cURL, Postman or the tool of your choice), please make sure that you have added an oAuth2 authentication with the correct secrets.

Moreover, this endpoint only exposes keys that were previously defined in the DI of your Magento. By default only `design/footer/absolute_footer` and `design/header/welcome` will be available on the endpoint. This allows to avoid exposing sensitive information and reduces the attack surface of your API. However, you can add in your own `di.xml` the following XML to ensure that more keys are available in your endpoint.

```diff
<?xml version="1.0" ?>
<config xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:noNamespaceSchemaLocation="urn:magento:framework:ObjectManager/etc/config.xsd">
    <!-- ... -->
+    <type name="\FrontCommerce\Integration\Model\Config\ConfigKeyChecker">
+        <arguments>
+            <argument name="acceptList" xsi:type="array">
+                <item name="your/config/path" xsi:type="string">your/config/path</item>
+                <item name="your/config/path2" xsi:type="string">your/config/path2</item>
+            </argument>
+        </arguments>
+    </type>
</config>
```

In this case we've decided to expose the keys `your/config/path` and `your/config/path2`. However you can expose any configuration from your Magento.

Once you've set the new arguments to the `ConfigKeyChecker` class, you need to update your DI in magento by running the following command: `php bin/magento setup:di:compile`.

The endpoint should now expose your config to any authenticated requests with the `Magento_Config::config` permission.

```
// Request (with additional Authorization headers)
http://magento.test/rest/V1/default/frontcommerce/storeConfigs?keys[]=your/config/path
// Response
[
    {
        "key": "your/config/path",
        "value": "My config value"
    },
]
```

## Fetch configurations in your Front-Commerce application

Once you've exposed the configurations on your Magento endpoint, you will need to use these configurations in Front-Commerce. There are two ways to do so:

- If you need these configurations client side, you can expose a configuration in your GraphQL
- If you need these configurations server side, you can use the MagentoConfig loader

### Expose a configuration in your GraphQL

When exposing a new field in GraphQL, you will write a GraphQL module in order to [extend the GraphQL schema](/docs/essentials/extend-the-graphql-schema.html) and then write something along those lines in your `schema.gql`:

```graphql
# my-module/server/modules/custom/schema.gql
extend type Query {
  fieldName: String
}
```

In the case of a configuration coming from Magento, it is the same thing but you need to add a `@magentoConfig` directive after your field to tell your GraphQL which configuration key is associated with this field.

```graphql
# my-module/server/modules/custom/schema.gql
extend type Query {
  fieldName: String @magentoConfig(key: "your/config/path")
}
```

<blockquote class="important">
Please make sure to add `Magento2/Store` to your GraphQL module's dependencies to make sure that the `@magentoConfig` directive is available.

```diff
// my-module/server/modules/custom/index.js
export default {
    // ...
+    dependencies: ["Magento2/Store"],
    // ...
}
```

</blockquote>

Under the hood, this directive will automatically create a resolver for this field that will fetch the configuration from Magento. You can use this field however you want in your React application.

Please note that in this example, we are adding the field to the `Query` type, but it can be any type in your GraphQL schema.

#### My field type is not a String

In the previous example, we have used a String to represent the configuration. however in some cases you don't want to expose a String, but an Int, or even more complex types like `Wysiwyg` or `Product`. To do so, you will need to:

- update your schema with the correct type
- add a resolver that transforms the configuration value in what you need to represent this type

Let's say that our configuration is a `Wysiwyg`. This means that you will need to operate the following changes:

```diff
# my-module/server/modules/custom/schema.gql
extend type Query {
-  fieldName: String @magentoConfig(key: "your/config/path")
+  fieldName: Wysiwyg @magentoConfig(key: "your/config/path")
}
```

```diff
// my-module/server/modules/custom/resolvers.js
export default {
  Query: {
+    fieldName: ({ magentoConfig }, _, { loaders }) => {
+      // the configuration value from the @magentoConfig is available in
+      // the magentoConfig key of the first argument of your resolver
+      return loaders.Wysiwyg.parse(magentoConfig, "MagentoWysiwyg")
+    }
  }
}
```

### Use the MagentoConfig loader

If you can't use the `@magentoConfig` directive, you can also fetch the configuration from a loader available in Front-Commerce: `loaders.MagentoConfig`.

<blockquote class="important">
Please make sure to add `Magento2/Store` to your GraphQL module's dependencies to make sure that the `@magentoConfig` directive is available.

```diff
// my-module/server/modules/custom/index.js
export default {
    // ...
+    dependencies: ["Magento2/Store"],
    // ...
}
```

</blockquote>

This means that in your resolvers, you can write the following code:

```diff
// my-module/server/modules/custom/resolvers.js
export default {
  Query: {
+    fieldName: ({ magentoConfig }, _, { loaders }) => {
+      return loaders.MagentoConfig.load("your/config/path")
+    }
  }
}
```
