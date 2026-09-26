# What is simulated in Agentic Flow

Judges: this is a hackathon build. Everything below is simulated or seeded, and we say so openly.

| Item | What is simulated | Why | What production would use |
|---|---|---|---|
| Grocery catalogue | 40 products with invented brands, imported into a Shopify dev store | No real grocer data available | The merchant's real Shopify catalogue |
| Demo customers | 6 customers whose emails are team members' own inboxes | So offers can be received safely during the demo | Real customers with real consent |
| Purchase history | 90 days of synthetic transactions generated in Databricks | Dev store has no history | Shopify order history |
| Batches and use-by dates | `agenticflow.batches` seeded by us | Shopify does not track expiry per batch | Warehouse / WMS batch data |
| Propensity and churn scores | Rule-based scores, formulas documented in code | No time to train a model | A trained model in Databricks |
| Payments | Shopify Bogus gateway test payments | Dev stores cannot take real payments | Real payment provider |
| `AS_OF` time setting | Demo runs can set "now" to a chosen time before use-by | So the video can show urgency changing without waiting 36 hours | Real current time (the scheduled job uses real time) |
| Bloomreach sending account | Agent runs under a team member's Bloomreach API group | Hackathon sandbox | Dedicated service account |
