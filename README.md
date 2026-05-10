# Enterprise ad fraud detection 2026: the four-stage funnel framework

A working framework for enterprise ad fraud detection across pre-bid, click, post-click, and CAPI-payload stages. Plus reference patterns for closing the post-click + CAPI feedback loop the legacy verification stack doesn't cover.

## Why this exists

Most "enterprise ad fraud detection" pages rank pre-bid vendors (DV, IAS, HUMAN, MOAT). They miss the bigger picture: ad fraud has four funnel stages, and the legacy verification stack only covers one.

The Adalytics 2025 reports flagged accuracy gaps even at pre-bid (IAS missed obscured bots 77% of the time). The CAPI feedback loop where bots train Meta and Google optimization models is the silent killer of CAC.

This README is the framework plus reference implementation.

## The four stages

```
[ad served]
    |
    v
Stage 1: pre-bid impression -- DV, IAS, HUMAN, MOAT
    |
    v
Stage 2: click -- Lunio, ClickCease, CHEQ, TrafficGuard
    |
    v
Stage 3: post-click -- DataCops (the gap)
    |
    v
Stage 4: CAPI-payload -- DataCops (the bigger gap)
```

The legacy verification tier covers stage 1. The click vendors cover stage 2. Stages 3 and 4 are where modern AI agent traffic and bot conversion poisoning lives.

## Stage 4: the CAPI feedback loop

When a bot's conversion event fires through your pixel and flows to Meta or Google CAPI, the optimization model treats it as a successful conversion. Lookalike audiences get trained on the bot. CAC creeps up.

The fix: tag the CAPI event payload with the fraud verdict.

```json
{
  "event_name": "Lead",
  "event_time": 1715299200,
  "event_id": "evt_abc123",
  "action_source": "website",
  "user_data": {
    "em": ["hashed_email"],
    "client_ip_address": "203.0.113.42"
  },
  "custom_data": {
    "fraud_verdict": "human",
    "fraud_score": 0.04,
    "fraud_reasons": []
  }
}
```

When `fraud_verdict` is `bot` or `risky`, also set `data_processing_options: ["LDU"]` so Meta excludes the event from optimization.

## Honest vendor map

| Stage | Vendor | Notes |
|---|---|---|
| 1 (pre-bid) | DoubleVerify | ~68% market share. Adalytics flagged gaps. $50K to $500K+ ACV. |
| 1 (pre-bid) | IAS | 77% miss on obscured bots (Adalytics 2025). Senator letters. |
| 1 (pre-bid) | HUMAN | Strong API/account-layer. Custom enterprise. |
| 1 (pre-bid) | MOAT | Post-Oracle uncertainty. |
| 2 (click) | Lunio | EU-strong. From ~$99/mo. |
| 2 (click) | ClickCease | SMB-friendly published pricing. |
| 2 (click) | CHEQ | Cybersecurity pedigree. Custom enterprise. |
| 2 (click) | TrafficGuard | Multi-channel. From ~$300/mo. |
| 3 (post-click) | DataCops | The post-click bot filter layer. |
| 4 (CAPI-payload) | DataCops | The CAPI feedback layer. |

## Reference setup: closing stages 3 and 4

```bash
# 1. CNAME on your subdomain
# CNAME ads-trust -> cdn.datacops.com

# 2. Drop the script
```

```html
<script async src="https://ads-trust.yourdomain.com/dc.js" data-site="YOUR_SITE_ID"></script>
```

```bash
# 3. Configure CAPI destinations
# Meta, Google Ads, TikTok, LinkedIn

# 4. Verify
# - Real-time bot percentage on dashboard
# - CAPI events show fraud_verdict in payload
# - Meta Events Manager shows EMQ above 8.0
```

## Verifying the loop is closed

After 14 days of CAPI events tagged with fraud verdict, you should see:

- Lookalike audience CPM trending down
- Meta CPA decreasing on consistent budget
- Bot share of "Lead" or "Purchase" events trending down in your dashboard
- Higher EMQ on Meta Events Manager

If those don't move, you have a stage-1 or stage-2 problem (your pre-bid or click filter is letting bots through). Layer in DV/IAS at pre-bid or Lunio at click.

## When DataCops isn't the right tool

- You only need pre-bid impression coverage (DV, IAS, HUMAN, MOAT).
- You only need click-layer filtering on Google Ads (ClickCease, Lunio at SMB tier).
- You're a Roblox-tier adversarial gaming platform with SMS toll fraud as a top risk (Arkose has the $1M warranty).
- Mobile attribution fraud is your specific pain (AppsFlyer Protect360 or Branch).

## Disclaimer

DataCops is in the SOC 2 Type II in-progress phase, not certified. Enterprise tier offers a single-tenant runtime, dedicated IP DB (146.4B datacenter, 202B residential, 11.9B VPN, 620M proxy, 160K fraud email domains), custom DPA, EU/US data residency, HubSpot integration, migration engineer, and 99.9% uptime SLA today.

## Links

- Fraud Traffic Validation: joindatacops.com/fraud-traffic-validation
- Conversion API: joindatacops.com/conversion-api
- SignUp Cops: joindatacops.com/signup-cops
- Enterprise: joindatacops.com/enterprise
- Pricing: joindatacops.com/pricing

---

Research by [DataCops](https://www.joindatacops.com) · First-party tracking, consent infrastructure & fraud prevention.
