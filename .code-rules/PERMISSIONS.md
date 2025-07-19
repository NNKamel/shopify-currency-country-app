## **Installation and Permissions Configuration**

Setting up the app for use involves proper installation flows and configuration in Shopify:

* **App Registration**: Create the app in the Shopify Partners dashboard (or use Shopify CLI to scaffold a new app). Ensure you mark the app as **Embedded** (so that it can appear in the Shopify Admin Apps section and use App Bridge). In the app setup, list the required API scopes (as identified above: products, markets, etc.). After setting the app URL and redirect URLs, install the app on a development store.

* **OAuth & Permission Prompt**: When the merchant (store owner) installs the app, Shopify will prompt them to approve the requested permissions. They should see that the app is asking to “View and edit products, view markets” etc. This must be accepted for the app to function. Make sure your OAuth implementation includes those scopes and exchanges the temporary code for a token successfully.

* **API Access Configuration**: Once installed, the app’s backend will receive an OAuth access token. Store this token securely (mapped to the store). This token will be used for all Admin API calls (GraphQL and REST). No further login is needed by the user beyond the initial install approval.

* **Embedded App Setup**: Use Shopify App Bridge in your front-end code to ensure the app is rendered inside Shopify admin. For example, initialize App Bridge with the API key and shop domain, and wrap your React app in the Polaris **AppProvider** with App Bridge support. This enables features like the top bar, modal, and resource picker integrations to work[shopify.dev](https://shopify.dev/docs/api/app-bridge#:~:text=URL%3A%20https%3A%2F%2Fshopify.dev%2Fdocs%2Fapi%2Fapp,Shopify%20mobile)[shopify.dev](https://shopify.dev/docs/apps/build/admin#:~:text=The%20screenshot%20highlights%20an%20app,menus%2C%20modals%20that%20cover%20the).

* **Necessary Webhooks** (Optional): You might consider subscribing to relevant webhooks:

  1. If markets or currency settings are changed (Shopify might have a webhook for markets changes or shop settings update) – to know if you should refresh cached market data.

  2. If products are deleted or variants removed – to clean up any stored references.

  3. App uninstallation webhook – to clean up data if needed when the app is removed.

* **Testing**: After installation, test the app on the store:

  1. Add a market in Shopify admin with a different currency (if not already set up).

  2. Create a product or pick an existing one, use your app to set a fixed price for that market.

  3. Verify on the storefront (by changing the country or using a VPN/GeoIP or adding `?country=XX` to the URL) that the price shows as the one you set.

  4. Test that not setting a price for a product in a given market results in the default converted price (ensuring your app only affects selected products).

* **Shopify Markets Feature Integration**: No additional installation is needed for Markets itself (it’s a Shopify core feature), but the store should have Shopify Markets enabled and at least one market besides the primary. You may want to mention in your app documentation that the merchant should configure Markets (Settings \> Markets in admin) before using the app. The app could even detect if Markets is not enabled and prompt the user to enable it.

* **App Configuration**: In the Partners dashboard or the app’s code, ensure the app has a proper **redirect URL** after OAuth and that the embedded settings are correct. Also, configure the **App URL** which is the landing page in admin (the app home). This is where your React/Polaris app loads and from where the merchant will interact with the features described.

Finally, once everything is configured, the app will appear in the merchant’s **Apps** section in Shopify admin. Clicking it will load your embedded app. From there, the merchant can select products and set market-specific prices as designed.

