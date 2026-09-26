# Agentic Flow

An autonomous agent that stops online grocers throwing away stock and margin.

Built for the Bloomreach Composable AI Hackathon 2026, track T6, using Databricks, Google Gemini,
Bloomreach Engagement and Shopify.

## How it works
1. **Signal (Shopify + Databricks):** every few hours, live Shopify stock is compared with each batch's
   use-by date and recent sales. Batches that won't sell in time are flagged.
2. **Target (Databricks):** customer scores (purchase propensity, churn risk, favourite category)
   pick the customers most likely to want the product.
3. **Decide (Gemini):** Gemini chooses who to target, what discount each group gets, the channel
   and the message, escalating as expiry approaches. Code enforces margin and use-by guardrails.
4. **Act (Shopify + Bloomreach):** the agent creates discount codes in Shopify and sends
   `surplus_offer` events to Bloomreach, where a live scenario emails each customer
   (with a 20% control group).
5. **Learn (Databricks):** Shopify orders and Bloomreach engagement are written back, so the next
   decision starts from what actually worked.

## Repo layout
- `notebooks/00_access_checks` — connection tests for all four platforms
- `notebooks/01_generate_data` — creates the demo data (see MOCKS.md)
- `notebooks/02_agent` — the agent itself, run as a scheduled Databricks Job
- `MOCKS.md` — everything simulated, disclosed

## Setup
_To be completed: secrets, Shopify app, Bloomreach scenario, Databricks job._

## Team
Agentic Flow — _names and roles to be added_
