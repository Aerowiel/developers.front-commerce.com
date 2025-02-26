---
id: add-new-shipping-data-in-graphql
title: Add a shipping method with pickup points
---

In this guide we'll go through the steps needed to retreive pickup points from GraphQL and display them in a Map using Front-Commerce's component. This method can also be useful if you are planning to setup a Store Locator in your shop.

If you are willing to display these pickups in the checkout this guide will help you. However if you are willing to save the needed information in the user's quote, please have a look at [source/docs/advanced/shipping/custom-shipping-information.md](Custom Shipping Information) instead.

<blockquote class="feature--new">
_Since version 2.1.0_
</blockquote>

> **Prerequisite:** To go through this guide, you'll need to have a created a [new GraphQL module](/docs/essentials/extend-the-graphql-schema.html) and to know how to [fetch data in a component](/docs/essentials/create-a-business-component.html#Fetching-our-data).

## Add pickups to your GraphQL Schema

The goal here is to be able to fetch a list of pickups from GraphQL. You could create your own types and your custom implementation. However, in Front-Commerce, we've created a GraphQL interface named `FcPickup` that ensures that any pickup point can be displayed in the PostalAddressSelector component of Front-Commerce.

### 1. Declare your custom pickup points in your GraphQL module

```graphql
type CustomPickup implements FcPickup {
  id: ID!
  name: String!
  address: FcPostalAddress!
  coordinates: GeoCoordinates!
  schedule: FcSchedule
}

extend type Query {
  customPickupList: [CustomPickup]
}
```

Feel free to add any parameters to your query. It can for instance be useful to use an address parameter to filter based on the location of the user. You can use the `FcPostalAddressInput` for this usecase. It would look like this:

```graphql
# src/server/modules/shipping/schema.gql
extend type Query {
  pickupList(
    "The shipping address"
    address: FcPostalAddressInput!
  ): [CustomPickup]
}
```

Moreover, the fields listed in the `CustomPickup` type are compulsory and come from `FcPickup`. But if you need more, feel free to add them to your `CustomPickup` type.

Once you've created your schema, you can import it in your GraphQL module.

```js
// src/server/modules/shipping/index.js
import typeDefs from "./schema.gql";

export default {
  namespace: "CustomShipping/Magento1",
  dependencies: ["Front-Commerce/Shipping"],
  typeDefs,
};
```

> **Important:** Since we're using the `FcPickup` interface and other types coming from Front-Commerce's core. Make sure to add the `Front-Commerce/Shipping` [dependency in your GraphQL's module declaration](/docs/reference/graphql-module-definition.html#dependencies-optional).

### 2. Add a loader

You will now have to create a loader that returns the list of pickups. Let's say it's named `CustomShippingLoader` and has a function named `loadPickupList`. Then this list needs to be formatted following the GraphQL types used previously. You would have something like this:

```js
// src/server/modules/shipping/loaders.js

// Please keep in mind that this formatting function is an example
// and needs to be adapted to your usecase. The goal is to have the
// same guys, but with the correct values.
const formatPickupPoint = (pickupPoint) => {
  return {
    id: pickupPoint.id,
    name: pickupPoint.name,
    address: {
      country: pickupPoint.countryCode,
      locality: pickupPoint.city,
      postalCode: pickupPoint.zipcode,
      streetAddress: pickupPoint.street,
      // If your address doesn't need any additional information,
      // you can use the DefaultPostalAddress type. This is the
      // default implementation of the interface FcPostalAddress.
      // If you're unsure, use it, it's a safe bet.
      __typename: "DefaultPostalAddress",
    },
    coordinates: {
      latitude: pickupPoint.latitude,
      longitude: pickupPoint.longitude,
    },
    schedule: {
      monday: pickupPoint.schedule.monday,
      tuesday: pickupPoint.schedule.tuesday,
      wednesday: pickupPoint.schedule.wednesday,
      thursday: pickupPoint.schedule.thursday,
      friday: pickupPoint.schedule.friday,
      saturday: pickupPoint.schedule.saturday,
      sunday: pickupPoint.schedule.sunday,
    },
  };
};

const CustomShippingLoader = () => {
  return {
    loadPickupList: async () => {
      const response = await axios.get("/your-remote-api");
      return response.data.map(formatPickupPoint);
    },
  };
};
```

The API can live anywhere. It can come from your backend, a shipping service, or a CMS, as long as it returns pickup points.

Once you've created your loader, you can register it in your GraphQL module:

```diff
// src/server/modules/shipping/index.js
import typeDefs from "./schema.gql";
+import CustomShippingLoader from "./loaders";

export default {
  namespace: "CustomShipping/Magento1",
  dependencies: [
    "Front-Commerce/Shipping",
  ],
  typeDefs,
+  contextEnhancer: ({ req, loaders }) => {
+    const CustomShipping = CustomShippingLoader(axiosInstance);
+
+    return {
+      CustomShipping,
+    };
+  },
};
```

### 3. Use your loader in a resolver

```js
// src/server/modules/shipping/resolvers.js
export default {
  Query: {
    pickupList: (parent, args, { loaders }) =>
      loaders.CustomShipping.loadPickupList(address),
  },
};
```

```diff
// src/server/modules/shipping/index.js
import typeDefs from "./schema.gql";
import CustomShippingLoader from "./loaders";
import resolvers from "./reslovers";

export default {
  namespace: "CustomShipping/Magento1",
  dependencies: [
    "Front-Commerce/Shipping",
  ],
  typeDefs,
+  resolvers
};
```

### 4. Test your API in your playground

By now, if you've registered your module in your `.front-commerce.js` and restarted the server, you should be able to retreive the pickup list from GraphQL.

```graphql
query PickupListQuery($address: FcPostalAddressInput!) {
  pickupList(address: $address) {
    id
    name
  }
}
```

## Display the list of pickup points

Now that you have the list of pickup points available in GraphQL, it is now possible to display them in your application. This can be done by using the component `theme/components/organisms/PostalAddressSelector`.

To do so, you need to first retreive the list of pickup from GraphQL. This can be done by using this query:

```graphql
#import "theme/components/organisms/PostalAddressSelector/PostalAddressSelectorFragment.gql"

query PickupListQuery {
  pickupList {
    ...PostalAddressSelectorFragment
  }
}
```

The fetched data, then needed to be passed to your component by using the `graphql` HOC.

```js
import { graphql } from "react-apollo";
import PickupListQuery from "./PickupListQuery.gql";
import PostalAddressSelector from "theme/components/organisms/PostalAddressSelector";

const PickupList = ({ loading, error, locations }) => {
  const [activeLocation, setActiveLocation] = useState(null);
  const [activeLocationIntention, setActiveLocationIntention] = useState(null);

  if (loading) {
    return <div>Loading...</div>;
  }
  if (error) {
    return <div>Error...</div>;
  }

  return (
    <PostalAddressSelector
      activeLocation={activeLocation}
      activeLocationIntention={activeLocationIntention}
      setActiveLocation={setActiveLocation}
      setActiveLocationIntention={setActiveLocationIntention}
      locations={locations}
    />
  );
};

export default graphql(PickupListQuery, {
  props: ({ data }) => {
    return {
      loading: data.loading,
      error: !data.loading && data.error,
      locations: data.pickupList ?? [],
    };
  },
})(PickupList);
```

Please note here that we're using the `activeLocation` and the `activeLocationIntention`. The difference between those two is that the `activeLocation` is only set if the user explicitly clicked on the button that allows them to choose a pickup point. The intention is triggered if they've clicked on the map's marker or on the address of the pickup. This allows to fine tune the UX depending on your pickup selector needs.

`activeLocation` though is entirely optional and you can rely on `activeLocationIntention` instead.

## Final words

Please keep in mind that this feature is built for shipping purpose in priority. This means that you may need additional work to make it work with a custom store locator page for instance. But it shouldn't be too much work and would allow you to have the same behaviors in both your store locator and your checkout.

Moreover, by using this method, this also means that you are only a step away from adding a click & collect shipping method (provided you have the existing logistics in your business).

If you have any questions, please contact us. We are eager to support more use cases.
