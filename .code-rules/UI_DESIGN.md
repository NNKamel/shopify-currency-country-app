## **UI Design for Product Selection and Price Input**

A user-friendly interface is crucial for merchants to easily select products and set prices per market. Here are some recommendations for the UI/UX:

* **Product Selection**: Allow the merchant to pick which products (or specific variants) they want to configure. Use an embedded app component like the Shopify App Bridge **Resource Picker** to let the user browse or search products and select some. This could be a modal that lists products and allows multi-select. Alternatively, create a Polaris page with a searchable list/table of products with checkboxes.

* **List Selected Products**: Once products are selected, display them in the app interface. For each product, show its variants (if the product has options). Each variant could be listed as a row in a table or an expandable section under the product. Include identifying info like variant title or SKU.

* **Markets and Price Inputs**: For each variant, provide input fields for each market where the merchant wants a custom price. A clean way is to create a table where columns represent markets (with currency codes in headers) and rows represent variants. Merchants can then fill in the price for the intersection of variant and market. If there are many markets, you might instead have the UI organized by market:

  * e.g. first select a Market from a dropdown, then show all selected products/variants with a field to enter that market’s price. Then repeat for another market. This avoids a very large grid if there are numerous markets.

  * Alternatively, organize by product: select a product \-\> show that product’s variants and for each variant a list of markets with input fields.

* **Use Polaris Components**: Shopify Polaris offers components that can be helpful:

  * **Forms and Form Layout**: Group price input fields in forms. Use Polaris **FormLayout** and **TextField** components for each price input for consistency (e.g., a numeric TextField with a suffix label for currency).

  * **Data Table**: If using a table layout for variants vs. markets, Polaris DataTable can format this nicely with rows and columns.

  * **Tabs or Segmented Control**: If you opt to have the merchant switch between markets or products, a tab interface could work (each tab \= one market or one product).

  * **Feedback and Save**: Use Polaris **Banner** or inline error messages to show any API errors (e.g., if a price fails to save). Use the App Bridge **Contextual Save Bar** to prompt the user to save changes if there are unsaved inputs (Shopify App Bridge has a ready-made top bar for this).

* **Saving Flow**: When the merchant clicks “Save” (or “Apply Prices”), gather all the inputs and send them to your backend (or directly to GraphQL). Show a loading state (Polaris **Spinner** or **Loading** component) while the API calls are happening. After success, show a confirmation message (Polaris **Toast** or Banner).

* **Limiting Scope**: Emphasize that this tool is for **selected products only**. The UI should not overwhelm by listing every product by default. The merchant explicitly chooses which products to configure, so the interface stays focused. This satisfies the requirement that the functionality is limited to chosen products, not a mass update of all products.

* **Preview Option** (nice-to-have): If feasible, show a preview of how the price will look on the storefront. This could be as simple as “Price in {MarketName}: {formatted price with currency}”. This confirms to the user that, for example, “Product A will show €50.00 to customers in Europe”.

By following Shopify’s design guidelines and using Polaris, the app will feel integrated with the admin. The merchant can intuitively search and pick products, then assign prices for different markets in a structured way.

