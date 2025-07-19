# Project Tasks for Currency-Country Conversion App

This document breaks down the work needed to build and complete the app based on the project guidelines and requirements. The tasks are organized to reflect their dependencies.

---

### **Phase 1: App Setup and Configuration** ✅

This phase covers the foundational steps to get the app registered and authenticated with Shopify.

- **1.1. App Registration**:
    - Create the app in the Shopify Partners dashboard or use Shopify CLI.
    - Mark the app as **Embedded**.
    - Configure the **App URL** and **Redirect URLs**.

- **1.2. Permissions and Scopes**:
    - Request the following OAuth scopes: `read_products`, `write_products`, `read_markets`, and optionally `write_markets`.

- **1.3. Authentication**:
    - Implement the OAuth 2.0 flow to get an access token.
    - Securely store the access token, mapped to the store.
    - Use Shopify App Bridge for session management within the embedded app.

- **1.4. Initial App Structure**:
    - Set up a basic Node.js/Express or Ruby on Rails backend.
    - Create a minimal React frontend with Shopify Polaris and App Bridge.

---

### **Phase 2: Backend Development & API Integration**

This phase focuses on building the server-side logic to interact with Shopify's APIs.

- **2.1. Market Data**:
    - Implement a service to fetch the list of markets using the GraphQL `markets` query.
    - Retrieve market details: `id`, `name`, and `currencySettings.baseCurrency.currencyCode`.

- **2.2. Price List Management**:
    - Create logic to query for existing **Price Lists** associated with a market.
    - Implement the `priceListCreate` GraphQL mutation to create a new price list if one doesn't exist for a market.
    - Handle reusing or updating existing price lists.

- **2.3. Price Update Endpoint**:
    - Create a backend endpoint that accepts product variant IDs and new prices.
    - This endpoint will call the `priceListFixedPricesUpdate` GraphQL mutation to set the fixed prices.
    - Implement error handling for the API calls.

- **2.4. (Optional) Webhooks**:
    - Subscribe to webhooks for market changes, product deletions, and app uninstallation to keep data synchronized.

---

### **Phase 3: Frontend Development (UI/UX)**

This phase covers building the user interface for the merchant.

- **3.1. Product Selection**:
    - Implement a product selection interface using the Shopify App Bridge **Resource Picker** or a custom searchable list.
    - Allow merchants to select multiple products and their variants.

- **3.2. Main App Interface**:
    - Display the selected products and their variants in a clear layout (e.g., a table or list).
    - Design the price input section. Options:
        - A table with variants as rows and markets as columns.
        - A view where the merchant first selects a market, then sees all products to price.
    - Use Shopify Polaris components (`FormLayout`, `TextField`, `DataTable`, `Tabs`) for a native feel.

- **3.3. User Feedback**:
    - Implement a **Contextual Save Bar** (from App Bridge) to show when there are unsaved changes.
    - Use Polaris `Banner`, `Toast`, or inline messages to display success or error messages from the API.
    - Show loading indicators (`Spinner`, `Loading`) during API calls.

---

### **Phase 4: Core Feature - Setting Prices**

This phase connects the UI to the backend to implement the main feature.

- **4.1. Data Flow for Pricing**:
    - When the merchant enters prices, store the changes in the frontend state.
    - On "Save", send the new price data to the backend endpoint.

- **4.2. Currency Handling in UI**:
    - Clearly label all price input fields with the correct currency code (e.g., "Price (EUR)").
    - Use numeric `TextField` components and consider a currency formatting library.
    - (Optional) Display a "current converted price" as a reference for the merchant.

- **4.3. Reverting Prices**:
    - Implement a UI element (e.g., a "revert to automatic" button) to allow merchants to remove a fixed price.
    - This will call the backend, which will use the `variantIdsToDelete` argument in the `priceListFixedPricesUpdate` mutation.

---

### **Phase 5: Finalization and Testing**

This phase ensures the app is robust and user-friendly.

- **5.1. Merchant Guidance**:
    - In the UI, include text that explains:
        - The need to include currency conversion fees in fixed prices.
        - That fixed prices don't update automatically with exchange rates.
        - That fixed prices are "as-is" regarding taxes and duties.
    - Add a check to see if Shopify Markets is enabled and prompt the user if it's not.

- **5.2. End-to-End Testing**:
    - Test the entire flow on a development store.
    - Add a market with a different currency.
    - Set a fixed price for a product and verify it on the storefront.
    - Test that reverting a price works as expected.
    - Ensure that products without fixed prices still show the default converted price.

- **5.3. (Optional) Price Preview**:
    - Implement a feature to show a preview of what the price will look like on the storefront.