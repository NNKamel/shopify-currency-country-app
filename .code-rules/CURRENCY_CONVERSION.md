## **Currency Conversion and Formatting Considerations**

Working with multiple currencies and localized prices introduces a few important considerations:

* **Currency Codes and Symbols**: Always clarify which currency a price is for. For example, if the store’s default is USD and the merchant is setting a price for Europe (EUR), the UI should label it clearly (e.g. “Price (EUR)”). Use Shopify’s currency codes or symbols appropriately. The app can retrieve the currency symbol for a market’s currency via the Shopify API or use a library for formatting. For input fields, you might disable direct alphabet input and perhaps suffix the currency code in the UI for clarity.

* **Formatting**: Different currencies have different formatting (commas, decimals). Using a formatting library or Shopify’s \[Polaris `TextField` with type "number"\] combined with a format helper can ensure prices are input and displayed in a familiar format. However, you may allow simple numeric input and leave formatting minimal (Shopify will display it properly on the storefront).

* **Conversion Reference**: Since merchants may want to know what a converted price would be, consider showing a helper text. For instance, if the base price is $10 USD and they are setting a price in EUR, you might show “Current converted price: \~€9.20” as reference (fetched via an API). This can be done by querying Shopify’s Storefront API in the context of that country to get a converted price, or by using exchange rates from a currency API. This is optional but helps merchants decide on manual prices.

* **Currency Conversion Fees**: Shopify applies a conversion fee on foreign currency orders when prices are auto-converted. **Fixed prices bypass automatic conversion**, so the merchant should account for conversion fees in the price they set. Shopify’s docs note that if you set fixed prices for a market, you need to include the currency conversion fees in those prices yourself[help.shopify.com](https://help.shopify.com/en/manual/international/pricing/product-prices-by-country#:~:text=Currency%20conversion%20fees%20apply%20to,fees%20in%20your%20fixed%20prices). In other words, price high enough to cover any extra fee that Shopify would normally add on top of the exchange rate.

* **Exchange Rate Changes**: Be aware that fixed prices won’t automatically adjust if exchange rates fluctuate. This is the trade-off for manual control. You might warn merchants that they should occasionally review fixed prices, since the store’s base currency value of that price can drift over time with currency changes.

* **Tax & Duties Inclusion**: If the store uses Shopify’s international pricing features like dynamic tax-inclusive pricing or duties, note that **fixed prices are taken as-is**. Shopify will not further adjust a fixed price for taxes; the merchant should input it exactly as they want it shown to customers[help.shopify.com](https://help.shopify.com/en/manual/international/pricing/product-prices-by-country#:~:text=). This is usually fine, but it’s a consideration if, say, VAT is included in EU prices – the merchant should enter the price with VAT included if that’s desired.

By handling currencies carefully – labeling fields with currency codes, possibly offering conversion guidance, and educating the merchant on fees – the app will help manage price localization effectively.

