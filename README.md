# Tel Setu (तेल सेतु) — Used Cooking Oil Collection Network

A last-mile collection and traceability platform that connects small food vendors with biodiesel processors, turning used cooking oil (UCO) into feedstock instead of waste.

Built for **Hack for a Social Cause — VBYLD 2027**, Theme 5: Environment & Sustainability.

**Live demo:** https://claude.ai/artifact/YS9g5SZ4RyKUU8ZHLKFL4r

---

## The problem

Small food vendors and dhabas in tier-2/3 towns like Raipur, Chhattisgarh routinely reuse cooking oil past FSSAI's safe limit (25% Total Polar Compounds), because discarding it is a pure loss with no buyer. India's existing UCO collection network (FSSAI's RUCO initiative) serves large hotel chains in metro cities — a 6-litre-a-week dhaba is not worth a dedicated collection run under that model. The result: a public health risk on one side, and biodiesel feedstock worth real money going to waste on the other.

## The solution

Tel Setu is a three-role platform:

- **Vendors** register their shop once, then request pickups whenever they have oil ready. They see their own pickup history, quality grade, and payout status in real time.
- **Collectors / operators** work a pickup queue grouped by area (so several small stops become one viable route), log actual litres and an oil-quality grade at collection, and mark payments as completed.
- **Everyone** sees a live dashboard: total litres collected, quality mix, area breakdown, amount paid to vendors, diesel-equivalent offset, and an indicative biodiesel value — the traceability record a processor needs to trust the supply.

This closes a gap between two of India's largest import bills: the country imports 55–60% of its edible oil and separately imports roughly 89% of its crude oil. UCO that has already been paid for once (as imported edible oil) does not have to become pure waste — converted to biodiesel, it displaces diesel that would otherwise require importing more crude.

## Who this is for

- Small vendors and households in Raipur (and, as the network grows, Durg, Bhilai, Bilaspur) who currently have no formal, paid channel to dispose of used oil safely.
- Local collectors, prioritising women and self-help-group members, earning a per-litre commission on a route they can actually run profitably.
- FSSAI-registered biodiesel processors (e.g. Chhattisgarh Biofuels, Dharma Energy) who need steady, quality-verified feedstock but cannot economically source it below metro scale today.

## Features

- Vendor self-registration with shop name, owner, phone, area, and precise pickup location
- Vendor code system so a shop can be reopened from any device without a login system
- Pickup request flow with estimated litres and a note field for the collector
- Operator queue grouped by area for basic route optimisation
- Quality grading (good / medium / poor) with automatic payout calculation (₹20–28/litre)
- Real-time sync between vendor and operator views — a request made on one device appears instantly on another
- Live dashboard with litres, payouts, quality mix, area breakdown, and a full collection ledger
- Diesel-equivalent offset and indicative biodiesel value, computed from collected volume

## Tech stack

- Single-page HTML5 / CSS / vanilla JavaScript client — no build step, no framework dependency, runs on any browser including low-end Android devices
- Real-time shared data store (Claude Artifact `db` capability in this deployment) for cross-device sync between vendor, operator, and dashboard views
- LocalStorage used only for a single per-viewer convenience (remembering the signed-in vendor's code on that device) — never for shared data

See [`ARCHITECTURE.md`](./ARCHITECTURE.md) for the full data model and design decisions.

## Repository contents

| File | Purpose |
|---|---|
| `index.html` | Complete application — vendor registration, pickup requests, operator queue, dashboard. Single self-contained file. |
| `README.md` | This file |
| `ARCHITECTURE.md` | Data model, sync design, and known limitations |
| `sample-data.json` | Example `vendors` and `pickups` records showing the exact document shape, for testing or reviewing without live registration |
| `LICENSE` | MIT |

## Running / trying it

This build is deployed as a Claude Artifact and works as-is at the demo link above. To adapt the front-end code for a different real-time backend (Firebase, Supabase, a custom API), replace the data-layer calls in `index.html` — the UI, forms, and business logic (payout calculation, route grouping, dashboard aggregation) are backend-agnostic and can be pointed at any document-store API that supports live subscriptions. `sample-data.json` shows the exact shape each record must have.

```bash
# open directly in a browser for a static/offline preview of the UI
open index.html
```

Note: the offline/static preview will not sync data between devices — that requires the real-time backend described in ARCHITECTURE.md.

## Roadmap

| Phase | Timeline | Milestone |
|---|---|---|
| Pilot | 0–3 months | 15–20 vendors onboarded in Pandri, Tatibandh, GE Road; validate buyback pricing |
| Formalise | 3–6 months | FSSAI RUCO registration; signed offtake agreement with a regional processor |
| Expand | 6–12 months | Extend to Durg, Bhilai, Bilaspur using the same collector network model |

## AI tool disclosure

AI assistance (Claude) was used for: background research on India's crude oil and edible oil import data and FSSAI RUCO policy; drafting and editing the text of the presentation deck and this documentation; and generating portions of the front-end application code, which the team reviewed, tested, and modified. The problem framing, choice of solution, Raipur field context, and factual verification of cited figures are the team's own work. See the presentation deck for the team's full disclosure statement and estimated proportion of AI involvement.

## License

MIT — see [`LICENSE`](./LICENSE).

## Team

TEL SETU — Hack for a Social Cause, VBYLD 2027, State: Maharashtra
Piloted in Chhattisgarh (Raipur) — the Team Lead's home state, where the gap was identified firsthand. The collection-network model is directly replicable in Maharashtra's own tier-2/3 towns, which face the identical FSSAI RUCO coverage gap.

Team Lead: Shrish Pratap Singh — National Insurance Academy, Pune
Registration ID: HSC|MH|00047
