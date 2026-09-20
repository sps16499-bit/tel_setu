# Architecture

## Overview

Tel Setu is a single-page web application with three views (Vendor, Operator, Dashboard) reading and writing one shared, real-time data store. There is no separate backend server in this build — the client talks directly to a hosted document store with live subscriptions, and every connected view updates automatically when any other view writes.

```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│  Vendor view │      │ Operator view│      │Dashboard view│
│  (phone/web) │      │ (phone/web)  │      │ (phone/web)  │
└──────┬──────┘      └──────┬──────┘      └──────┬──────┘
       │  live read/write   │  live read/write     │  live read
       ▼                    ▼                      ▼
              ┌───────────────────────────┐
              │   Shared document store    │
              │  collections: vendors,     │
              │  pickups                   │
              └───────────────────────────┘
```

## Data model

Two collections, both plain JSON documents:

### `vendors/{vendorId}`

| Field | Type | Description |
|---|---|---|
| `id` | string | Client-generated unique id |
| `code` | string | Human-readable vendor code (`TS-XXXX`), used to re-open the shop on any device |
| `shop` | string | Shop / dhaba name |
| `owner` | string | Owner's name |
| `phone` | string | Contact number |
| `area` | string | One of a fixed set of Raipur localities |
| `address` | string | Free-text pickup location / landmark |
| `weekly` | number \| null | Vendor's estimated weekly UCO volume, litres |
| `createdAt` | ISO 8601 string | Registration timestamp |

### `pickups/{pickupId}`

| Field | Type | Description |
|---|---|---|
| `id` | string | Client-generated unique id |
| `vendorId` | string | Foreign key to `vendors` |
| `shop`, `area`, `address` | string | Denormalised from the vendor at request time, so the operator queue and dashboard don't need a join |
| `litresEst` | number | Vendor's estimate at request time |
| `litresActual` | number \| null | Set by the operator at collection |
| `quality` | `"good"` \| `"medium"` \| `"poor"` \| null | TPC-strip grade, set at collection |
| `rate` | number \| null | ₹/litre rate applied, derived from `quality` |
| `payout` | number \| null | `litresActual × rate`, rounded |
| `status` | `"requested"` \| `"collected"` \| `"paid"` | Pickup lifecycle state |
| `note` | string | Optional vendor note to the collector |
| `requestedAt`, `collectedAt`, `paidAt` | ISO 8601 string \| null | Lifecycle timestamps |

Denormalising `shop`/`area`/`address` onto each pickup is a deliberate trade-off: it means a vendor's later address edits don't retroactively change historical pickup records, which is the correct behaviour for an audit trail — a processor or jury should see the location a pickup actually happened at, not a shop's current address.

## Why this shape

- **Two flat collections, no nesting.** Every query the app needs (open requests, a vendor's own history, area totals, quality mix) is a filter or sort over one collection's top-level fields — no joins, no subcollections needed.
- **Status as a single field, not a flag per stage.** `requested → collected → paid` is a strict progression, so one `status` field plus timestamps captures the whole lifecycle without extra booleans to keep in sync.
- **Client-generated ids.** Vendor and pickup ids are created on the device that writes them, so a registration or a pickup request can be composed and written in one step without a round trip to mint an id first.
- **Denormalisation over joins.** With a five-thousand-document ceiling on this class of store and a target scale of hundreds of vendors, the small amount of duplicated text (shop name, area, address) costs far less than the complexity of a join layer, and keeps every list view a single-collection query.

## Real-time sync

Both the Operator queue and the Dashboard subscribe to the full `pickups` collection; the Vendor view subscribes to both `vendors` and `pickups` and filters client-side to the signed-in vendor's own records. A write from any view (a new pickup request, a logged collection, a payment marked) triggers the store's change notification, and every subscribed view re-renders within moments — this is what makes a live two-screen demo (one phone as vendor, one as operator) possible.

## Identity model

There is no traditional login. A vendor is identified by a short human-readable code (`TS-XXXX`) generated at registration and remembered in that device's local storage. Entering the same code on a different device opens the same shop. This was chosen deliberately for the target users — small vendors on shared or basic phones, where a password-based account system is friction with no real benefit, since the risk being protected against (someone else claiming your shop) is low-stakes and locally verifiable by the operator team in a real deployment.

## Payout calculation

Payout is computed client-side at the point of logging a collection: `payout = round(litresActual × rate)`, where `rate` is looked up from the graded quality (`good: ₹28`, `medium: ₹24`, `poor: ₹20` per litre — figures based on researched UCO buyback ranges in India, adjustable per local market conditions). This calculation is deterministic and auditable from the stored `litresActual` and `quality` fields alone, which is what makes the collection ledger a genuine traceability record rather than an opaque number.

## Known limitations and honest scope

- **Backend portability.** This build uses a real-time document store provided by the hosting platform (Claude Artifacts). The application logic — forms, payout calculation, route grouping, dashboard aggregation — is written to be backend-agnostic; porting to Firebase, Supabase, or a custom API means replacing the data-access calls in the single script block, not rewriting the UI.
- **Access model.** In this deployment, real-time sharing works between signed-in viewers of the same organisation opening the artifact link — it is not yet a public consumer app a vendor could open from an SMS link. A field pilot needs a different, publicly-accessible deployment.
- **Route grouping is area-level, not distance-based.** Pickups are sorted alphabetically by named locality, which approximates route efficiency without true geocoding. A production version would use real coordinates and a shortest-path calculation.
- **No offline queue on the vendor side.** A pickup request requires connectivity at the moment it's submitted. A field version for low-connectivity areas would benefit from an offline-first request queue that syncs when signal returns.
- **Quality grading is self-reported by the collector**, not sensor-verified. This matches how TPC test strips are actually used in the field (a human reads a color-based test), but a processor relying on this data at scale would want periodic third-party quality audits.

## License

MIT. See `LICENSE`.
