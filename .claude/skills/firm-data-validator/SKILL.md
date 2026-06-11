---
name: firm-data-validator
description: "Validates prop firm data for accuracy and completeness before it is written to any SOPF page. Use when checking firm account sizes, pricing, drawdown rules, payout splits, affiliate links, or promo codes. Triggers on: 'validate firm data', 'check firm info', 'verify promo code', 'is this data correct for {firm}', 'audit firm page data'."
---

# Firm Data Validator

Validates structured prop firm data against known sources and the MightyOx affiliate master list.

## Affiliate Code Master
| Firm | Affiliate Link | Code | Code Case |
|------|---------------|------|----------|
| My Funded Futures | https://bit.ly/motmffu | MOT | uppercase |
| Take Profit Trader | https://bit.ly/tptmot | none | — |
| Funded Futures Network | https://bit.ly/0xffn | MOT | uppercase |
| Legends Trading | https://bit.ly/oxlegends | MIGHTYOX | uppercase |
| Tradeify | https://tradeify.co/?ref=MOT | Mot | title case |
| Lucid Trading | https://lucidtrading.com/ref/mightyoxtrading/ | MOT | uppercase |
| Bulenox | https://bit.ly/motbul | MOT | uppercase |
| Apex Trader Funding | https://bit.ly/ap0x90 | MIGHTYOX | uppercase |
| TradeDay | https://bit.ly/tdox | MOT | uppercase |
| Phidias | https://bit.ly/phdiox | none | — |
| Daytraders | https://bit.ly/oxday | MOT | uppercase |
| Purdia Capital | https://bit.ly/oxpur | none | — |
| E8 Markets | https://e8markets.com/d/MOT | MOT | uppercase |
| Alpha Capital Group | https://bit.ly/Alphaox | mot | lowercase |
| Phoenix Trader Funding | https://bit.ly/pheox | mot | lowercase |
| NexGen ProTrader | https://bit.ly/ngmot | MOT | uppercase |

## Validation Rules

### Affiliate Links
- Must match the master list exactly — no variations, no redirect guesses
- Must include full URL with protocol
- bit.ly links are valid — do not expand them

### Promo Codes
- Case-sensitive — MOT is not the same as mot or Mot
- If a firm has no code, the field should be empty or display "none" — never invent a code

### Account Data
- Profit target must be consistent with account size (e.g. $50K account should not have a $500 target)
- Daily loss limit must be less than or equal to max drawdown
- Min trading days: 1 is valid for firms like NexGen, Apex — do not flag as error
- Price (after discount) must be less than price (full)

### Required Fields
All firm pages must have: account size, price, drawdown amount, drawdown type, profit target, platform(s), payout split, payout frequency

### Optional Fields (mark N/A if unknown)
CEO, Founded, Country, Support channels, Daily loss limit

## Output
Return a validation report:
```
FIRM: {name}
STATUS: VALID | INVALID | PARTIAL
ISSUES: [list of specific problems]
WARNINGS: [non-blocking concerns]
VERIFIED_FIELDS: [list]
UNVERIFIED_FIELDS: [list — mark these NEEDS VERIFICATION in output]
```
