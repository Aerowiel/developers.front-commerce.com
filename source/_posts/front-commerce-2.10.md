---
title: '2.10: first version of the Proximis connector, Mondial Relay & Adyen module updates, Magento 2.4.3 compatibility'
date: 2021-10-14
---

**The 2.10 release brings Front-Commerce to the unified commerce market**. With Proximis headless capabilities, merchants can now offer a great experience to their customers no matter where they are.

We've improved our Front-Commerce modules offer too, with a shipping and a payment provider that solve B2C merchants needs.

**In September, 3 new Front-Commerce storefronts went live** with very different experiences: [B2C](https://www.kaporal.com/), [Omnichannel B2C](https://www.terreseteaux.fr/) and [B2B](https://www.e-robertet.com/)! We would like to congratulate our partners PH2M and Smile for these successful launches 👏👏

Our [website](https://www.front-commerce.com/) has a new look. We've simplified our message and put the focus on everyone who works with us. What do you think?

<!-- more -->

## Proximis × Front-Commerce: first version ready!

After several alpha releases, the [Proximis](https://www.proximis.com/) connector for Front-Commerce is available. This first version covers the main customer journeys with a redirection to Proximis' default external checkout.

You can contact us for more details and see it in action!

<div class="center">
  <p>
    <a class="link primary button" href="https://www.front-commerce.com/contact/">Request a personal demo</a>
  </p>
  <p>
    [Watch the replay of our live event in Paris with ZeTrace & Proximis (FR)](https://www.youtube.com/watch?v=ZlhUVXueUSk)
  </p>
</div>

## Mondial Relay for Magento2

Magento2 merchants can now use Mondial Relay shipping with Front-Commerce. We've collaborated with [Magentix](https://mondialrelay.magentix.fr/fr/magento-2/) to provide a seamless integration out-of-the box.

This module leverages [the shipping modules introduced in 2.5](/blog/2021/03/11/front-commerce-2.5/#Shipping-modules-Colissimo-Mondial-Relay) and the Front-Commerce [`<Map />` component](/docs/advanced/features/display-a-map.html).

## Adyen update for Magento2

You can now use the 7.2 version of [Adyen Payment's official Magento2 plugin](https://github.com/Adyen/adyen-magento2).

Recent versions (7.x) of this plugin contained breaking changes for headless integrations. We've updated our payment module to support these new APIs.

We also made Adyen frontend components shared between Magento2 and Front-Commerce payments implementations.

## Magento 2.4.3 compatibility

FC now supports Magento 2.4.3 officially. Some stores are already using it in production.

**Please note that [this official patch about the Web API 20 items limit in arrays](https://support.magento.com/hc/en-us/articles/4406893342093) is required for Front-Commerce to work.**

## Other changes

- fixed SSR rendering for categories with filter (critical regression introduced in 2.8.0 and already fixed in 2.8.3 and 2.9.1)
- only in-stock configurations are now selectable upon cart item edition (it prevents selecting a configuration leading to an add to cart error messages)
- options are now prefilled correctly in the modal when updating a Cart item
- the browser scrollbar is now visible after creating a new address
- browser form autofill feature is now better supported by the SmartForm inputs
- a properly rewritten URL is now used on the Magento 1 category pages for category facets when no search datasource is configured (instead of `/category/:id`)
- you can now react to `<Map />` component changes using the new `onBoundsChanged` prop
- the Payline environment is now fetched from the Magento configuration in the [Magento1 payment module](/docs/advanced/payments/payline.html#Magento1-module)
- product cross sells are now more resilient to Magento2 errors
- the [browserslist](https://github.com/browserslist/browserslist) database was updated to target only necessary browsers in final JS bundles
- fixed a typo in the french translation: selectioné -> selectionné

Fixes from the 2.10 version have also been backported into previous minor versions. The following patch versions were released: [2.4.8](https://gitlab.com/front-commerce/front-commerce/-/releases/2.4.8), [2.5.4](https://gitlab.com/front-commerce/front-commerce/-/releases/2.5.4), [2.6.2](https://gitlab.com/front-commerce/front-commerce/-/releases/2.6.2), [2.7.3](https://gitlab.com/front-commerce/front-commerce/-/releases/2.7.3), [2.8.4](https://gitlab.com/front-commerce/front-commerce/-/releases/2.8.4) and [2.9.2](https://gitlab.com/front-commerce/front-commerce/-/releases/2.9.2).

<hr />
<div class="center">
  <p>
    <a class="link primary button" href="https://www.front-commerce.com/contact/">💌 Ask your questions about Front-Commerce</a>
  </p>
  <p>
    [Upgrade to Front-Commerce 2.10.0](/docs/appendices/migration-guides.html#2-9-0-gt-2-10-0) or [read the full changelog (Customers only)](https://gitlab.com/front-commerce/front-commerce/-/releases/2.10.0)
  </p>
</div>
