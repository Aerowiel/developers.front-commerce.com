---
id: add-http-endpoint
title: Add custom endpoints to your server
---

The main entrypoints of a Front-Commerce application are frontend URLs displaying your actual website and the GraphQL schema. But in some specific cases, you may want to extend your Node.js server (<abbr title="Backend For Frontend">BFF</abbr>) with additional endpoints.

For instance, this can be the case when you send a link within an email to one of your customers. If an action should be triggered when the user clicks on it, you don't want to add failure opportunities by displaying a webpage which will then trigger a GraphQL mutation. You want to trigger the action directly, and then redirect the user to an actual page.

This can also be useful if you want to add new pages that are used in an administration interface. For instance, you may want to provide an iframe that will let you preview how your WYSIWYG component will be displayed in Front-Commerce. This is what we are going to explain in this documentation.

Technically, Front-Commerce’s server is based on [Express](http://expressjs.com/). To add your [custom route](https://expressjs.com/en/guide/routing.html) which will add your custom endpoint, you need to register it from your own module.

## What is a module?

A module is declared using the [`modules` entry from the `.front-commerce.js` file](/docs/reference/front-commerce-js.html#modules). Each module can contain client code (See [Extend the theme](/docs/essentials/extend-the-theme.html)), and server code (current guide).

Once the module is declared, Front-Commerce will automatically add your custom server entrypoints following your module configuration in `my-module/server/module.config.js`.

<blockquote class="note">
**Important:** GraphQL modules, are modules related to the server, but they won't let you extend anything else but your GraphQL Schema. See [Extend the GraphQL Schema](/docs/essentials/extend-the-graphql-schema.html) for more information.
</blockquote>

## Register additional routes

In the case of the WYSIWYG preview, let's say that when a user visits the `wysiwyg-preview` URL, we want to display static files that will let anyone preview how an html string will be transformed by [Front-Commerce's Wysiwyg component](https://gitlab.com/front-commerce/front-commerce/tree/main/src/web/theme/modules/Wysiwyg).

To do so, we need to create our server config file `my-module/server/module.config.js`, and configure its `entrypoint` key.

```js
import router from "./express";

export default {
  endpoint: {
    // The path added to your Front-Commerce server
    path: "/wysiwyg-preview",
    // The router used for this path
    router: router
  }
};
```

If we translated this in a standard express application, it would look like this:

```js
import router from "./express";

app.use("/wysiwyg-preview", router());
```

<blockquote class="note">
See our express middleware to understand how the magic works: [`withCustomRouters`](https://gitlab.com/front-commerce/front-commerce/blob/daade2b43df41d4a387a04fe87f432afc1f7208b/src/server/express/withCustomRouters.js#L23).
</blockquote>

Thus, the `my-module/server/express.js` file within your module should be a function returning either a standard express route (`(req, res, next) => { /* ... */ }`) or [an express router](https://expressjs.com/en/api.html#router).

Now, if we want to display our WYSIWYG preview iframe, it could look like this:

```js
import path from "path";
import cors from "cors";
import { Router } from "express";
import config from "config/website";

const ONE_HOUR = 60 * 60;

export default () => {
  const router = new Router();

  // We make sure that the resources are only available for
  // our administration interface
  router.use(
    cors({
      origin: config.admin
    })
  );

  // And then, we render a static page that let's you
  // preview some wysiwyg content
  router.use(
    express.static(path.join(process.cwd(), "build/wysiwyg"), {
      setHeaders: function(res, url, stat) {
        res.setHeader("Cache-Control", `public, max-age=${ONE_HOUR}`);
      }
    })
  );

  return router;
};
```

## Add a global server middleware

We've added our middleware that is now available under the `/wysiwyg-preview` path. But what if we want to add a middleware that should be used before any requests to your application?

To do so, you will need to add the option `__dangerouslyOverrideBasePathChecks` to your endpoint configuration.

```js
import router from './express'

export default {
  endpoint: {
    __dangerouslyOverrideBasePathChecks: true,
    path: "/",
    router: router
  }
};
```

Indeed, if you don't set this key to `true`, Front-Commerce will check that you don't setup an invalid path.

For instance, if your application is a migration from an older website, you might want to intercept all your requests to redirect old URLs to new ones. In `express`, you can do this by creating a middleware. But this means that all the URLs need to pass by this middleware. This is what `__dangerouslyOverrideBasePathChecks` stands for: it lets you define the path you want (`/` in this case) even if it seems dangerous. Once it is set, the middleware can add the redirects for the relevant URLs.

## Subdirectory based urls and global middleware

If your store URLs are `/fr` and `/en`, you may want to add a root middleware that would for instance handle `/.well-known` urls.

Using `endpoint.__dangerouslyOverrideBasePathChecks` as illustrated in the previous section would still mount the returned router under each sub-directory (`/fr/.well-known` and `/en/.well-known`).

In this case, you must use the `rootEndpoint` key. It has the exact same signature than `endpoint` but will mount the returned router at the global level.

```js
import router from './express'

export default {
  rootEndpoint: {
    __dangerouslyOverrideBasePathChecks: true,
    path: "/",
    router: router
  }
};
```