# Log new Shopify orders to Google Sheets (free n8n workflow)

One n8n workflow: each new Shopify order becomes one row in a Google Sheet. A webhook delivered twice still gives you one row, because the workflow matches on `order_key`.

**Columns:** `order_key, order_number, order_date, customer_name, customer_email, currency, total, items_summary, financial_status, is_test, logged_at` (see `orders-template.csv` / `orders-template.xlsx`).

## How it works
1. **On new Shopify order** fires for each new order (`orders/create`).
2. **Normalize Shopify order** (Code node) turns the order into one tidy row: order key and number, date (UTC), customer name and email, currency, total, a one-line items summary, financial status and a test-order flag.
3. **Upsert row in Orders** writes the row to Google Sheets with "Append or update", matched on `order_key`. A repeat delivery updates the row instead of adding a new one.

## Setup
1. Create a Google Sheet with a tab named **Orders** and paste in the header row from `orders-template.csv`.
2. In n8n: *Import from file* → `log-new-shopify-orders-to-google-sheets.json`.
3. **On new Shopify order**: connect a *Shopify OAuth2 API* credential for an app you create in the Shopify Dev Dashboard with the `read_orders` scope. If you have an older admin-created custom app, switch Authentication to *Access Token* instead.
4. **Required: Protected customer data access.** Shopify treats customer name and email as protected customer data, and until your app has this access, activating the workflow fails with `Forbidden` ("This topic contains protected customer data") when n8n registers the `orders/create` webhook. In the Shopify Partner Dashboard go to **Apps → your app → API access → Protected customer data access**, select **Name** and **Email**, give a reason such as *Store management*, and save. For a custom app or a development-store app, Step 1 on that page is all you need. Merchants using their own custom app need the same setting.
5. **Upsert row in Orders**: connect Google Sheets and choose your sheet.
6. Your n8n must be reachable from the internet to receive webhooks (for self-hosted, a public HTTPS URL). Click *Execute workflow*, place a Shopify test order, and check the row. Then activate the workflow.

The sticky notes on the canvas repeat these steps. The workflow file contains no credentials or credential IDs; after import you pick your own Shopify and Google Sheets credentials on those two nodes.

## What was tested
- **End-to-end on n8n Cloud 2.44.5 with a Shopify development store (Oct 2026):** imported this workflow (same nodes and settings; only sticky-note text changed since), connected real Shopify and Google Sheets credentials, activated it, and placed 2 test orders marked as paid. Both landed as rows in the `Orders` sheet. Re-running one order updated its existing row (matched on `order_key`) without adding a duplicate.
- Also built and run earlier on self-hosted n8n 2.43.8 with sample Shopify order payloads.
- Not yet run on a live store with real customer orders.

## Customize
To add a column, return the extra field from **Normalize Shopify order** and add a header with the same name to the sheet.

## Full version
The **Holiday Rush Automation Pack for n8n** adds Stripe and Gumroad orders, a per-product Items tab, thank-you emails, low-stock alerts and a daily sales digest: https://quietenginecompany.com/products/

## Support
Questions or problems? Email **quietenginecompany@gmail.com** with what you tried. We usually reply within a few business days.

---
Made with AI assistance. Independent project; not affiliated with or endorsed by n8n or Shopify. MIT licensed (see `LICENSE`).

Made by Quiet Engine Co. · https://quietenginecompany.com
