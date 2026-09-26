# Agentic Flow — instructions for Claude

Read this before writing any code for this repo.

## What we are building
Agentic Flow is an autonomous agent for an online grocer (Bloomreach Composable AI Hackathon 2026, track T6).
Every few hours it finds stock batches that won't sell before their use-by date, decides who to offer them to
and at what discount, sends the offer, and learns from the results.

Loop: Shopify stock + Databricks sales history -> risk check -> Databricks customer scores ->
Gemini decision -> Shopify discount code -> Bloomreach `surplus_offer` event -> live Bloomreach
scenario emails the customer -> Shopify order -> results written back to Databricks.

## Hard rules
- Python only. Code runs as Databricks notebooks (serverless). No TypeScript, no web server.
- Never put secrets in code. Read them with `dbutils.secrets.get("agenticflow", "<key>")`.
- Anything faked, simulated or hard-coded for the demo must be marked with a `# MOCK:` comment
  AND listed in MOCKS.md. Never silently stub an API call to make something "work".
- Every notebook cell must include its own imports and setup (notebooks restart and lose state).
- Gemini must make the live decisions. Do not replace it with rules or with another model.
- Guardrails are enforced in code, not only in the prompt:
  - discount may never take the price below unit cost x (1 + product min margin)
  - never offer a batch that will be past its use-by date
  - any discount over 40% goes to `agenticflow.pending_approvals` instead of being sent
- Only customers with `demo_contact = true` may ever be emailed.

## Secrets (scope: agenticflow)
shopify_client_id, shopify_client_secret, gemini_api_key,
bloomreach_api_key_id, bloomreach_api_secret, bloomreach_project_token

## Shopify
- Store: <STORE_NAME>.myshopify.com (dev store, password-protected, Bogus gateway for test payments)
- Auth: Dev Dashboard app "Agentic Flow". Exchange client ID + secret for a 24-hour token at the start
  of every run: POST https://<STORE_NAME>.myshopify.com/admin/oauth/access_token
  with grant_type=client_credentials. Then call the GraphQL Admin API (version 2025-10)
  with header X-Shopify-Access-Token.
- Products: 40 grocery SKUs AF-0001 to AF-0040. The SKU is the product ID everywhere.
  Hero product: AF-0001 Scottish Salmon Fillets 2-pack 240g.
- All stock is at the location named "Shop location".
- Collection "Rescue deals" exists for final-hours public markdowns.

## Gemini
- Python SDK: `google-genai`. Model: `gemini-3.8-flash` (older names such as gemini-2.5-flash are retired).
- Disable automatic function calling. Ask for structured JSON output validated with Pydantic.

## Bloomreach Engagement
- Project: sleepy-badger (project token = bloomreach_project_token secret)
- API base URL: https://api-engagement.bloomreach.com (confirm against the dashboard)
- Tracking API with Basic auth (api key id + secret):
  - customers: POST {base}/track/v2/projects/{token}/customers
  - events:    POST {base}/track/v2/projects/{token}/customers/events
- CUSTOMER ID TYPES: this project uses `email_id` and `shopify_id`. There is NO `registered` ID.
  Always send: "customer_ids": {"email_id": <email>, "shopify_id": <shopify customer id>}
- Email consent category: "other". Demo customers have accepted it.
- Live scenario "Agentic Flow – Surplus offer": on event `surplus_offer` -> demo_contact = true ->
  80% email / 20% control group -> email.
- `surplus_offer` event properties (all required):
  subject_line, message, product_name, product_sku, discount_pct, discount_code, offer_ends, product_url
- Loomi Connect (MCP) cannot create customers, segments, or send campaigns; it creates drafts only.
  Use it for reading results, not for sending.

## Demo customers (Shopify customer ID -> email)
| Name | Email | Shopify ID | Persona |
|---|---|---|---|
| Amara Okafor | dolapo96+af1@gmail.com | 26631601127756 | active-fish-lover |
| James Whitfield | dolapo96+af2@gmail.com | 26631601226060 | lapsing-fish-buyer |
| Priya Shah | hello.akintoye+af3@gmail.com | 26631601258828 | high-value |
| Tom Barker | hello.akintoye+af4@gmail.com | 26631601324364 | at-risk-of-leaving |
| Grace Adeyemi | gabbibada+af5@gmail.com | 26631601357132 | bakery-lover |
| Oliver Grant | gabbibada+af6@gmail.com | 26631601389900 | ready-meals |

## Databricks tables (schema: agenticflow)
products, customers, transactions, batches, customer_features, customer_scores,
product_settings, interventions, pending_approvals
