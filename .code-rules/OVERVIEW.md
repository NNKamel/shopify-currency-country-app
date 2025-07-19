Overview

# **Building a Shopify Admin App for Per‑Market Product Pricing**

This guide walks through creating an **embedded Shopify Admin app** that lets merchants manually set different prices for selected products in different markets (countries), leveraging Shopify’s Markets feature. We will cover the app architecture, required APIs, how to manage markets and pricing data, currency considerations, UI design, and installation steps.

## **App Architecture and Tech Stack**

Develop the app as an **embedded Admin app** so it runs inside the Shopify admin interface. Key architectural considerations include:

* **Backend**: Use a server (e.g. Node.js/Express or Ruby on Rails) to handle OAuth, Shopify API calls, and business logic. Shopify’s Admin API (GraphQL recommended) will be used for data operations.

* **Frontend**: Build the UI with **Shopify Polaris** React components for a familiar look and feel, embedded via **Shopify App Bridge** in an iframe[shopify.dev](https://shopify.dev/docs/api/app-bridge#:~:text=URL%3A%20https%3A%2F%2Fshopify.dev%2Fdocs%2Fapi%2Fapp,Shopify%20mobile)[shopify.dev](https://shopify.dev/docs/api/app-bridge#:~:text=environment,react.shopify.com%2Ffoundations%29%20to%20create%20familiar). Polaris provides ready-made components (forms, tables, pickers) that match Shopify’s admin UI.

* **Session Auth**: Implement OAuth 2.0 with Shopify to obtain an API access token. Since the app is embedded, use Shopify App Bridge’s session tokens or cookies to maintain authenticated sessions within the Shopify admin iframe.

* **Data Flow**: The frontend will call backend endpoints (or directly use the Admin GraphQL API via the access token) to fetch markets, products, and update prices. Use Shopify’s official API libraries (e.g. `@shopify/shopify-api` for Node) for convenience.

**Summary**: A typical stack is **Node \+ React/Polaris**, with the app served in an iframe inside Shopify admin. This provides a seamless, secure experience where the app can interact with Shopify data while matching the admin UI.

## **Required Shopify APIs and Access Scopes**

Building this app requires using Shopify’s Admin APIs (primarily GraphQL, with REST as needed) and appropriate permissions:

* **Admin GraphQL API** – Highly recommended for this use-case because Shopify’s **PriceList API** (for market-specific pricing) is available via GraphQL. You will use GraphQL queries/mutations for markets and pricing overrides. (The REST API’s product variant endpoint exposes `presentment_prices` for reading but does not allow writing fixed international prices).

* **Admin REST API** – Optionally use REST for simpler data pulls (e.g. listing products or variants), but all per-market price adjustments will use GraphQL.

**Access Scopes (Permissions)**: In the app’s configuration, request the following OAuth scopes:

* `read_products` and `write_products` – to read product details and update product variant prices (required for PriceList GraphQL mutations[shopify.dev](https://shopify.dev/docs/api/admin-graphql/2024-04/mutations/priceListFixedPricesUpdate#:~:text=Requires%20,to%20create%20and%20edit%20catalogs)).

* `read_markets` (and possibly `write_markets`) – to retrieve market configurations. The Shopify Markets data requires the `read_markets` scope for queries (and `write_markets` if making market config changes)[shopify.dev](https://shopify.dev/docs/api/admin-graphql/latest/objects/Market#:~:text=Requires%20,for%20mutations).

* (Optional) `read_inventory` if you need inventory info, or other scopes as needed, but not required for pricing.

Ensure the app’s access token has these scopes when making API calls. For GraphQL, the Admin endpoint is `/admin/api/<version>/graphql.json`. All queries and mutations will need to include the token in the header.

## **Fetching and Managing Market Data**

To integrate with Shopify Markets, your app must fetch the merchant’s market settings and relevant context:

* **Retrieve Markets List**: Use the GraphQL Admin API’s `markets` query to get all markets configured for the shop[shopify.dev](https://shopify.dev/docs/api/admin-graphql/latest/objects/Market#:~:text=Anchor%20to%20markets%20%2081). For each market, retrieve details like the market ID, name, and base currency. For example, a GraphQL query might request each market’s `name`, `id`, and `currencySettings.baseCurrency.currencyCode`. This tells the app which countries/regions are grouped into which markets and the currency for each market.

* **Market Context**: Each Market has a base currency and possibly multiple regions. Note that if a market contains multiple countries, it uses a single base currency for all those countries. Your app should be aware that **pricing overrides can only be set in the base currency of a market** – other countries in that market will automatically use the converted price[help.shopify.com](https://help.shopify.com/en/manual/international/pricing/product-prices-by-country#:~:text=You%20can%20set%20individual%20product,market%20that%20you%20have%20activated). (If truly distinct pricing per country is needed, merchants must set up separate markets per country).

* **Price Lists**: Shopify Markets uses **Price Lists** behind the scenes to manage international pricing. Each market’s catalog can have an associated Price List that defines fixed product prices for that market[shopify.dev](https://shopify.dev/docs/apps/build/markets/build-catalog#:~:text=the%20catalog%20After%20you%20query,For)[shopify.dev](https://shopify.dev/docs/apps/build/markets/build-catalog#:~:text=prices%20converted%20to%20the%20market,price%20list%20currency%20to%20USD). Use GraphQL to query existing price lists:

  * You can call the `priceLists` query (GraphQL) to find price lists by name or currency. For example, find the PriceList for a market by matching its currency code or market name.

  * Alternatively, query a specific market’s catalog via GraphQL: e.g., use `market(id: …) { catalogs { priceList { id, currency } } }` to get the PriceList ID for that market if it exists.

* **Creating/Updating Price Lists**: If a market has no price list yet (meaning the merchant has not set custom prices or adjustments for that market), your app should create one. Use the GraphQL `priceListCreate` mutation to create a new price list for that market’s catalog, specifying the market’s base currency and linking it to the market’s publication/catalog[shopify.dev](https://shopify.dev/docs/apps/build/markets/build-catalog#:~:text=the%20catalog%20After%20you%20query,For). (Ensure the PriceList currency matches the market currency – a price list’s currency *must* match the market’s currency[shopify.dev](https://shopify.dev/docs/apps/build/markets/build-catalog#:~:text=prices%20converted%20to%20the%20market,price%20list%20currency%20to%20USD).)

  * If a PriceList already exists (e.g., the merchant may have set a percentage price adjustment in Markets settings), you can reuse it. You might update it to switch from percentage adjustment to manual pricing, or simply add fixed prices to it.

* **Fetching Current Prices**: For displaying current prices per market in the UI (if desired), you can use:

  * GraphQL Admin API: query each product variant’s `presentmentPrices` or use the new `@inContext(country: XX)` directive on Storefront API/Product queries to get the price in a specific country context. This lets you show the merchant what the price is currently (either converted or previously fixed) in each market.

  * Alternatively, use the REST Admin API to get `presentment_prices` for variants in various currencies (though this may require enabling those currencies in the store).

By managing market data through these APIs, the app can dynamically list all markets and ensure any pricing changes align with the correct market and currency.

## **Setting Product Prices Per Market**

The core feature is allowing merchants to set **fixed prices per product per market** (overriding Shopify’s automatic currency conversion for those products). Key steps to implement this:

1. **Product Selection**: First, determine which products and variants the merchant wants to override. The app can allow selecting one or multiple products (see the UI section). After selection, fetch the product’s variants (each variant has its own price). Gather each variant’s Shopify *variant ID* as you will need these for price updates.

2. **Prepare Price Inputs**: For each selected product variant and each target market, collect the desired price. The merchant will input a price in the market’s currency (the app should ensure the input is in the correct currency format, e.g. numeric field plus a currency label).

3. **Ensure Price List**: Determine the PriceList ID for the market:

   * If you retrieved an existing PriceList ID for the market in the previous step, use it.

   * If not, call GraphQL `priceListCreate` to make a new one (with the market’s base currency and link it to the market’s catalog/publication).

   * You may also need to attach the PriceList to the market’s catalog via `priceListUpdate` if not done at creation.

4. **Set Fixed Prices via API**: Use the **GraphQL Admin API** mutation to add or update fixed prices for variants on the price list. Shopify provides a mutation named `priceListFixedPricesUpdate` (or the similar `priceListFixedPricesAdd`) to do this. This mutation accepts:

   * The `priceListId` (for the market’s price list),

   * A list of `PriceListPriceInput` items for each variant you want to set, including the variant ID, the price (amount and currency), and optional compare-at price,

   * (Optionally, a list of variant IDs to remove from the price list if you want to delete previously set overrides).

For example, your mutation might add a price for variant `gid://shopify/ProductVariant/12345` on the EUR price list for “Europe” market. The mutation updates fixed prices on a price list – you use it to “set a fixed price for specific product variants” on that market’s price list[shopify.dev](https://shopify.dev/docs/api/admin-graphql/2024-04/mutations/priceListFixedPricesUpdate#:~:text=Updates%20fixed%20prices%20on%20a,associated%20with%20the%20price%20list). Shopify caps this at 250 variant price updates per call, which is plenty for most use cases.

 graphql  
Copy  
`mutation priceListFixedPricesUpdate($priceListId: ID!, $pricesToAdd: [PriceListPriceInput!]!) {`  
  `priceListFixedPricesUpdate(priceListId: $priceListId, pricesToAdd: $pricesToAdd, variantIdsToDelete: []) {`  
    `pricesAdded {`  
      `price { amount currencyCode }`  
      `variant { id }`  
    `}`  
    `userErrors { field message }`  
  `}`  
`}`

5.  In the variables, supply the PriceList ID for the market and an array of price inputs. Each `PriceListPriceInput` includes the variant ID, the new price (amount and currencyCode matching the market’s currency), and optionally a compare-at price. This will **override the variant’s price in that market**.

6. **Verify and Save**: After the mutation, check for `userErrors`. If none, the prices have been successfully set. The variant’s price for that market is now fixed at the given amount instead of using the automatic exchange-rate conversion. (If a fixed price already existed, it will be updated to the new value[shopify.dev](https://shopify.dev/docs/apps/build/markets/build-catalog#:~:text=Step%203%3A%20Set%20fixed%20prices,mutation%20accepts%20a%20maximum%20of).) Save any mappings as needed in your app’s database if you are tracking overrides internally (optional, since Shopify stores the fixed prices).

7. **Reverting Prices** (Optional): Provide a way to remove a fixed price if the merchant wants to go back to dynamic pricing. This can be done via the same mutation by providing the variant ID in the `variantIdsToDelete` list. If a fixed price is deleted, the market will revert to using the automatic converted price for that variant (based on the store’s base currency and any percentage adjustments) by default.

By following these steps, the app uses Shopify’s Markets API to set per-market variant pricing. Under the hood, this leverages Shopify’s PriceList API – a **PriceList** associated with the market’s catalog contains the custom prices. After setting these, customers from that market will see the specified prices on the storefront for those products (instead of converted prices).

