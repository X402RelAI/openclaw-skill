# RelAI — x402 Paid API Skill

Call paid APIs on the [RelAI marketplace](https://relai.fi) using x402 micropayments on Solana. This skill handles API discovery, pricing, and the x402 payment protocol. Wallet operations are delegated to [lobster.cash](https://www.lobster.cash).

## Features

- Browse and search the RelAI API marketplace
- Check endpoint pricing before calling
- Full x402 payment flow with on-chain settlement
- Automatic filtering of incompatible APIs (zAuth, non-Solana)

## Requirements

- [OpenClaw](https://openclaw.ai) with [lobster.cash](https://www.lobster.cash) plugin
- `curl` and `jq` installed
- `RELAI_API_URL` environment variable set

## Installation

```shell
npx skills add relai-fi/agent-skills --skill relai
```

Or manually copy the `relai/` directory to `~/.openclaw/skills/`.

## Configuration

Set the environment variable in your OpenClaw config or shell:

```bash
export RELAI_API_URL="https://api.relai.fi"
```

## Directory structure

```
relai/
├── SKILL.md                        # Main skill (workflow + guardrails)
├── README.md
├── LICENSE
├── agents/
│   └── openai.yaml                 # OpenAI agent config
└── reference/
    ├── marketplace-api.md          # Marketplace API endpoints
    ├── x402-flow.md                # x402 protocol details
    └── examples.md                 # Usage examples
```

## How it works

1. **Discover** — browse the marketplace, filter by Solana + no zAuth
2. **Price check** — show cost and get user confirmation
3. **Pay** — send USDC via lobster.cash, get on-chain tx hash
4. **Call** — attach payment proof (X-PAYMENT header) and get the API response

## License

MIT
