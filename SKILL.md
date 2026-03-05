---
name: relai
version: 1.0.0
description: "Browse and call paid APIs on the RelAI marketplace (relai.fi) using x402 micropayments. Delegates wallet and payment operations to lobster.cash. Use when: searching for an API, calling a paid endpoint, or checking pricing. NOT for: free/public APIs."
metadata: {"openclaw": {"emoji": "🔌", "primaryEnv": "RELAI_API_URL", "requires": {"env": ["RELAI_API_URL"], "bins": ["curl", "jq"]}}}
---

# RelAI — x402 Paid API Client

Call paid APIs on the RelAI marketplace. Handles API discovery and x402 payment protocol. Wallet operations are delegated to lobster.cash.

## Inputs needed

- `RELAI_API_URL` environment variable (e.g. `https://api.relai.fi`)
- A configured lobster.cash wallet with USDC balance on Solana

## Workflow

### Step 0 — Validate prerequisites

1) Verify `RELAI_API_URL` is set. If not, stop and inform the user.
2) Verify lobster.cash wallet is configured. If not, ask the user to set up lobster.cash first.

### Step 1 — Discover APIs

See [reference/marketplace-api.md](reference/marketplace-api.md) for full endpoint details.

```bash
curl -s "${RELAI_API_URL}/marketplace" | jq '.[] | {apiId, name, description, supportedNetworks, zAuthEnabled}'
```

Filter rules — skip and warn the user if any apply:
- `zAuthEnabled` is `true` (zAuth not supported)
- `supportedNetworks` does not include `"solana"` (lobster.cash is Solana-only)

### Step 2 — Explore endpoints and pricing

```bash
curl -s "${RELAI_API_URL}/marketplace/{apiId}" | jq '{
  apiId, name, description, network, zAuthEnabled,
  endpoints: [.endpoints[] | {path, method, summary, usdPrice, enabled}]
}'
```

Only endpoints with `enabled: true` and a `usdPrice` can be called.

### Step 3 — Wallet precheck

1) Check lobster.cash balance.
2) If balance < endpoint `usdPrice`, inform the user and stop.
3) Show the user the endpoint name, price, and ask for confirmation before proceeding.

### Step 4 — Call the API (x402 payment flow)

See [reference/x402-flow.md](reference/x402-flow.md) for protocol details and header format.

1) **Get 402 challenge**: `curl` the relay endpoint without payment. Expect HTTP 402.
2) **Parse**: decode `payment-required` header (base64 JSON). Extract `network`, `amount`, `asset`, `payTo`, `resource.url`. If `network` is not `solana`, stop.
3) **Pay**: send the required amount to `payTo` via lobster.cash. Wait for on-chain confirmation (timeout: 60s).
4) **Build X-PAYMENT**: construct header with `txId` from confirmed transaction.
5) **Retry**: `curl` the `resource.url` with `X-PAYMENT` header. Same request body as step 1.

## Output format

- **API discovered**: name, apiId, price, network
- **API response**: raw JSON from the endpoint
- **Payment summary**: amount paid, transaction hash, wallet balance after

## Guardrails

- Never call a paid endpoint without confirming the price with the user first.
- Never fabricate API responses or transaction hashes.
- Never create a new wallet if one already exists.
- Do not retry a failed payment automatically — show the error and let the user decide.
- Do not call APIs with `zAuthEnabled: true` or non-Solana networks.

## Error handling

| Situation | Action |
|---|---|
| `RELAI_API_URL` not set | Stop, tell user to configure it |
| Wallet not configured | Stop, ask user to set up lobster.cash |
| Insufficient balance | Show required amount, stop |
| `zAuthEnabled: true` | Inform user, skip this API |
| Non-Solana network | Inform user, skip (lobster.cash is Solana-only) |
| Non-402 response | Show the response and status code |
| Payment failed or timed out | Show error from lobster.cash, stop |
| API returned 4xx/5xx after payment | Show the raw error response |
