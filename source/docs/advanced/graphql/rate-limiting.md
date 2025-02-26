---
id: rate-limiting
title: Rate Limiting
---

<blockquote class="feature--new">
  _Available since version `1.0.0-beta.5`_
</blockquote>

You may want to prevent abuses of your application, for instance to prevent malicious users to send many contact form requests or scan some information with bots.
Front-Commerce’s core contains tooling that allows you to add basic rate limiting in a very granular way to any field (Query or Mutation).

This page explains how to add rate limiting to your resolvers, and how to tweak the configuration.

<blockquote class="info">
  This feature is a lightweight wrapper around the [`graphql-rate-limit`](https://www.npmjs.com/package/graphql-rate-limit) package.
</blockquote>

## Add rate limiting to a resolver

Let’s say that you have implemented an `incrementProductCounter` Mutation that allows you to increment a counter for a product. Its resolver might look like this:

```js
// my-module/server/modules/clicks-counters/resolvers.js
export default {
  Mutation: {
    incrementProductCounter: (_, { sku, incrementValue = 1 }, { loaders }) => {
      return loaders.Counter.incrementBySku(sku, incrementValue)
        .then(() => ({
          success: true
        }));
    }
  }
};
```

It makes sense to prevent abuses by allowing single users to only increment it once per hour. Front-Commerce provides a way to add this limitation with a few lines of code.

You can use the `limitRateByClientIp` decorator from the `"server/core/graphql/rateLimit"` module to add rate limiting capabilities to your resolver as follow:

```diff
// my-module/server/modules/clicks-counters/resolvers.js
+ import { limitRateByClientIp } from "server/core/graphql/rateLimit";
+
export default {
  Mutation: {
-    incrementProductCounter: (_, { sku, incrementValue = 1 }, { loaders }) => {
-      return loaders.Counter.incrementBySku(sku, incrementValue)
-        .then(() => ({
-          success: true
-        }));
+    incrementProductCounter: limitRateByClientIp(
+      { max: 5, window: "1m" },
+      (_, { sku, incrementValue = 1 }, { loaders }) => {
+          return loaders.Counter.incrementBySku(sku, incrementValue)
+            .then(() => ({
+              success: true
+            }));
+      }
+    )
  }
};
```

The difference here is that we replaced the resolver declaration with a `limitRateByClientIp` call.
`limitRateByClientIp` accepts 2 parameters:

1. the rate limit definition: an object with `max` (number of accesses allowed) and `window` (duration, in a format supported by [ms](https://github.com/vercel/ms)) properties
2. the original resolver, to call when conditions are satisfied and the user is allowed to access it

In case of abuses, GraphQL will return an Error for the field that has been limited. Here is an example:

> You are trying to access 'incrementProductCounter' too often

## Use a persistent store

By default, rate limiting is keeping a log of accesses in memory. It is a volatile implementation (data is lost upon server restart) that may not work in a distributed architecture (with several fronts behind a load balancer for instance).

Front-Commerce allows you to inject a custom store implementation, that will be used for all the rate limiting strategies. To do so, you will have to override the [`config/rateLimit.js`](/docs/reference/configurations.html#config-rateLimit-js) configuration file and expose a `store` factory.

Here is how you would replace the default in-memory store with a redis one:

```js
// my-module/config/rateLimit.js
import { RedisStore } from "graphql-rate-limit";
import redis from "redis";

export default {
  store: () => {
    // Return your GraphQLRateLimit compatible store implementation here
    // see https://www.npmjs.com/package/graphql-rate-limit#redis-store-usage
    return new RedisStore(
      redis.createClient({ host: "127.0.0.1" })
    );
  }
};
```

The store interface is minimalist, and you could implement a custom one if required. Please refer to `graphql-rate-limit`’s documentation for further information, or the [TypeScript definition](https://github.com/teamplanes/graphql-rate-limit/blob/master/src/lib/store.ts).

## Strategies

As of today, the only rate limiting strategy provided is the `limitRateByClientIp` one detailed above. However we have designed the code to allow to easily support more use cases as they are identified.

Please feel free to open an issue or [contact us](mailto:contact@front-commerce.com) if you have a different use case.