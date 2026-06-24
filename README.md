# Servo Saver — Product Analysis

A Business Analyst's report on the **Servo Saver** fuel-price feature of the Service Victoria app, modelled in UML. An individual project by Ramkrushna Bhadekar.

> Servo Saver lets Victorian motorists compare government-reported fuel prices at nearby service stations on a map or ranked list, highlights the cheapest options, and shows each retailer's locked maximum price for the following day. This report deconstructs the system from the motorist's perspective using UML, evaluates its interface, and proposes a single, well-evidenced improvement: **Price Drop Alerts**.

Full report: [Servo Saver Individual Project Report.pdf](Servo%20Saver%20Individual%20Project%20Report.pdf) ([.docx](Servo%20Saver%20Individual%20Project%20Report.docx))

## Contents

1. [Business Case](#1-business-case)
2. [Activity Diagram - Searching for Nearby Fuel](#2-activity-diagram---searching-for-nearby-fuel)
3. [Overall Use Case Diagram](#3-overall-use-case-diagram)
4. [Expanded Use Case - Search Nearby Fuel Stations](#4-expanded-use-case---search-nearby-fuel-stations)
5. [Class Diagram & CRC Cards](#5-class-diagram--crc-cards)
6. [State Machine Diagram - the FuelPrice Object](#6-state-machine-diagram---the-fuelprice-object)
7. [User Interface Design Evaluation](#7-user-interface-design-evaluation)
8. [Proposed Improvement - Price Drop Alerts](#8-proposed-improvement---price-drop-alerts)
9. [Improvement Incorporated into the Models](#9-improvement-incorporated-into-the-models)
10. [Limitations & Future Work](#10-limitations--future-work)
11. [Conclusion](#11-conclusion)
12. [References](#references)

Scope: this report analyses Servo Saver from the **customer (motorist) perspective only**. The retailer, administrator and regulator roles appear in the models for context only.

---

## 1. Business Case

Fuel retailing is one of Australia's largest consumer-facing industries (~$58.7B revenue, ~4.4% annualised growth to 2024-25, IBISWorld 2025). Prices move through sharp daily "price cycles" - the ACCC (2024) estimates a Melbourne motorist buying consistently at the cycle low could save ~$333/year. Victoria addressed this with mandatory real-time price reporting and a daily price-cap rule under its Fair Fuel Plan: retailers submit by 2pm, prices publish by 4pm, and the cap locks in from 6am for 24 hours (Premier of Victoria, 2025).

**Servo Saver** (launched October 2025) is the consumer face of this reform, delivered inside the Service Victoria "super-app" (100+ hosted services), so motorists inherit the platform's identity, security and trust.

| Figure | Value | Source |
|---|---|---|
| Australian fuel-retailing revenue (2024-25) | ≈ $58.7 billion | IBISWorld (2025) |
| 5-year annualised industry growth | ≈ 4.4% | IBISWorld (2025) |
| Potential annual saving (buying at cycle low) | ≈ $333 | ACCC (2024) |
| Daily price cap | Submitted 2pm → published 4pm → locks 6am for 24h | Premier of Victoria (2025) |
| Penalty per reporting breach | $3,000+ (serious offences $24,000+) | Consumer Affairs Victoria (2025) |

For the motorist, Servo Saver answers two questions: *"where is the cheapest fuel near me right now?"* and *"should I fill up today or wait?"* (via tomorrow's locked cap). The remaining gaps are problems of **reach and trust**, not data quality - the service is entirely pull-based, and a single app-wide "updated" timestamp obscures per-station freshness. These gaps motivate the proposed improvement (Section 8).

## 2. Activity Diagram - Searching for Nearby Fuel

📄 [Activity_Diagram.pdf](Activity_Diagram.pdf)

Models **UC1 - Search nearby fuel stations** step by step, across swimlanes for the Motorist, the Service Victoria host app, the Servo Saver module, and Location Services.

- **Decision 1 - GPS permission:** `[granted]` requests coordinates from Location Services; `[no permission]` falls back to manual suburb/postcode entry, which is geocoded. Both paths converge on a usable location.
- **Fork/join:** price retrieval and daily-price-cap retrieval run **in parallel**, since neither depends on the other.
- **Decision 2 - results found:** `[stations found]` renders the map; `[none found]` shows an empty state with a re-search option.
- Ends with the motorist viewing prices and selecting a station.

**Business meaning:** the core journey is short, resilient and fast - it never leaves the user stranded, and it exposes "view prices / select station" as the natural point where the proactive alerts feature later extends the experience.

## 3. Overall Use Case Diagram

📄 [Use_case_Diagram.pdf](Use_case_Diagram.pdf)

Bird's-eye view of every service Servo Saver offers, grouped by actor lane: **motorist**, **fuel retailer**, **scheduled-system**, **administrator**, **regulator**.

- Hub use case: **Search nearby fuel stations**, which `«includes»` *Detect user location*, *Retrieve real-time fuel prices*, *Display fuel price results* (mandatory sub-steps).
- Optional `«extend»` services: *Enter location manually*, *Filter (type/brand)*, *Sort (price/distance)*, *View daily price cap (tomorrow)*.
- Also: *Select fuel station* → *View station details*, *Get directions*, *Save favourite stations*; and *Report incorrect price/issue* (optionally extended by *Attach photo evidence*, *Provide contact details*).
- **Registered SV Account Holder** is a generalisation of **User (Motorist)** - inherits all motorist behaviour and adds account-specific actions (e.g. saving favourites). This generalisation is the integration point the improvement later attaches to.
- Context actors: *Location Services* «GPS Provider», *Mapping Service* «Routing Provider», *Fuel Retailer*, time-triggered *Scheduler*, *Service Victoria Admin*, *Consumer Affairs Victoria*.

**Business meaning:** Servo Saver is fundamentally a search-and-compare service for motorists wrapped in governance/data-supply machinery. Notably, **no use case has the system reach out to the motorist first** - every motorist flow begins with the motorist acting. That absence is exactly what the improvement addresses.

## 4. Expanded Use Case - Search Nearby Fuel Stations

📄 [Expanded Use Cases.pdf](Expanded%20Use%20Cases.pdf)

Fully-dressed description of **UC1**, primary actor User (Motorist), secondary actor Location Services «GPS Provider».

**Flow of activities** (actor → system):
1. Motorist opens app, taps "Servo Saver" → system loads feature, checks GPS permission.
2. Motorist confirms location access → system requests/receives coordinates, queries fuel-price DB and locked daily caps **in parallel**, renders map.
3. Motorist filters/sorts → system re-displays results.
4. Motorist selects a station → system displays station details.

**Exception conditions** (one per step): GPS denied → manual entry; Location Services failure → manual entry fallback; price DB unreachable → service-unavailable + retry; no stations found → empty state with widen/re-enter option; station details fail to load → cached details with "last updated" warning.

**Business meaning:** demonstrates the central service is implementable, not aspirational - and establishes the pattern (including the reused location-detection step) that the new *Set price drop alert* use case follows in Section 9.

## 5. Class Diagram & CRC Cards

📄 [Class Diagram.pdf](Class%20Diagram.pdf) · [CRC Card.pdf](CRC%20Card.pdf)

Core classes: **User** → (generalisation) **RegisteredAccountHolder**; **FuelRetailer**, **ServiceVictoriaAdmin**, **ConsumerAffairsVictoria**; **SearchRequest**, **Location**; and the heart of the model - **FuelStation**, **FuelPrice**, **FuelType**, **DailyPriceCap** - plus an abstract **Report** hierarchy.

Key relationships:
- `FuelStation` **composes** `FuelPrice` and `DailyPriceCap` (1 → 0..\*) - prices/caps cannot exist without their station.
- `FuelStation` is an **aggregation** part of `FuelRetailer` (a station is meaningful on its own, but owned by a retailer).
- `User` → `RegisteredAccountHolder` is a **generalisation**, mirroring the use case diagram.

**CRC card highlights:**
- **FuelStation** - knows identity/brand/opening hours; provides current prices (with FuelPrice, FuelType, SearchRequest); maintains price history/caps; holds at most **one active FuelPrice per FuelType at a time**.
- **FuelPrice** - knows amount/submission time/status; validates against the locked `DailyPriceCap`; activates/expires/supersedes itself; a price not refreshed is flagged stale by the Scheduler. This lifecycle is exactly what the state machine (Section 6) models.

## 6. State Machine Diagram - the FuelPrice Object

📄 [State Machine Diagram.pdf](State%20Machine%20Diagram.pdf)

Tracks the full life of a **single `FuelPrice` object** through nine states:

1. **Submitted** → automatically enters **Validating** (composite, own start/end).
2. Inside Validating, a **fork** runs `CheckingFormat` and `CheckingCap` concurrently; a **join** waits for both, then a **choice**:
   - fail → `[< 3 attempts]` back to Submitted, else → **Rejected** (archived).
   - pass → **Live** (composite: Active ⇄ Stale, with a **history marker** to resume the correct substate after interruption).
3. Out of Live: superseded by a newer price, withdrawn by the retailer, or expired after 7 days stale → **Archived** → retained 7 years → **final state** (destroyed).
4. A **terminate** marker purges the price immediately if its station is deregistered (consequence of the composition relationship in the class diagram).

**Business meaning:** guarantees published prices are trustworthy - every price is validated against the legal cap before going live, auto-flagged stale, and retained for compliance. The moment a price becomes **Active** is precisely the event the Price Drop Alerts feature listens for.

## 7. User Interface Design Evaluation

Based on a hands-on walkthrough of the live app (map view, list view, fuel-type selector, brand filter).

**The good:** map-first, colour-coded price pins with a "Lowest" badge; the "Tomorrow's price cap" panel supports a fill-now-or-wait decision; comprehensive filters/sort; inherits Service Victoria's trust and single sign-in.

**The bad:** entirely pull-based - no way to be told when a price falls; no save/watchlist capability; a single app-wide "updated" timestamp instead of per-station freshness; sort options aren't combinable (e.g. cheapest-within-distance); ambiguous search field.

**The ugly:** a prominent "Opening hours not available" line that reads as broken rather than absent; inconsistent brand-filter logos (many fall back to an identical grey placeholder), undermining a polished feature.

## 8. Proposed Improvement - Price Drop Alerts

A registered account holder sets a **fuel type, target price, search radius and optional quiet hours**; the system watches newly activated prices and notifies them when a station within range falls to or below the target - converting Servo Saver from a tool the motorist must remember to check into a proactive assistant.

**Integration, not a bolt-on:** reuses the existing *Detect user location* service (same GPS/manual fallback as UC1), validates targets against the same locked `DailyPriceCap`, and listens for the same `FuelPrice → Active` event the state machine already raises.

**Why it adds value:**
| Value driver | Mechanism | Evidence |
|---|---|---|
| Engagement & retention | Opt-in push alerts bring users back at the moment value is highest (~3x retention vs. non-notified users) | Airship (2024) |
| Motorist savings | Surfaces the cycle low / lower locked cap as it happens | ACCC (2024), ~$333/yr potential |
| Policy alignment | Steers demand toward competitively priced stations | Premier of Victoria (2025) |
| Low build cost/risk | Reuses existing location, price, cap and account components | This report |

**Trade-offs considered:** notification fatigue (mitigated by opt-in, user-set thresholds, quiet hours, a 10-alert cap); retailer perspective (rewards competitive pricing rather than penalising it); privacy (storing location/preferences flagged for a security review prior to implementation).

**Supporting interface fixes:** per-station freshness indicator, accessible price-trend icon (not colour-only), fixing the "opening hours" defect, and a favourites/watchlist that feeds naturally into alerts.

## 9. Improvement Incorporated into the Models

📄 [Use_case_Improvised.pdf](Use_case_Improvised.pdf) · [Expanded Use Cases Improvised.pdf](Expanded%20Use%20Cases%20Improvised.pdf) · [Class Diagram_improvised.pdf](Class%20Diagram_improvised.pdf) · [CRC_Cards_Improvised.pdf](CRC_Cards_Improvised.pdf)

**Improved use case diagram:** adds *Set price drop alert*, *Set quiet hours* (`«extend»`), *Manage saved alerts*, *Send price drop notification* (`«includes»` *Match alerts against new prices*), and a new **Notification Service** «Push Provider» actor. All new use cases attach to the existing **Registered SV Account Holder** specialisation and reuse the existing **Scheduler**.

**Expanded new use case - Set price drop alert:** primary actor Registered SV Account Holder; `«includes»` the reused *Detect user location*; `«extended by»` *Set quiet hours*. Flow: holder picks fuel type/threshold → system validates against the cap; holder sets radius → system determines location; holder optionally sets quiet hours; holder saves → system creates a `PriceAlert` (status = active) and registers it with the Scheduler. Five exception conditions cover not-signed-in, implausible threshold, GPS fallback, duplicate alerts, and the 10-alert limit.

**Improved class diagram - two new classes, purely additive (no legacy class altered):**
- **PriceAlert** - `alertId`, `fuelCode`, `thresholdPrice`, `radiusKm`, `quietHours`, `status`. Associates with `RegisteredAccountHolder` (0..\* alerts → 1 holder) and `FuelPrice` (0..\* matched ↔ 0..\* triggering); **composes** `AlertNotification`.
- **AlertNotification** - `notificationId`, `channel`, `sentAt`, `deliveryStatus`. Reaches `FuelPrice`/the holder only via its owning `PriceAlert`; failed deliveries retry up to 3 times.

**Business rules:** only registered holders can create alerts; an alert fires only when an Active `FuelPrice` for its fuel code, within radius, falls to or below threshold; no notifications during quiet hours (deferred, sent once they end).

## 10. Limitations & Future Work

Single-author, motorist-perspective-only analysis; models are a faithful reconstruction from observed behaviour, not internal documentation. Price Drop Alerts is specified at the analysis level and would need detailed design, a **security review** (location/preference storage), and user testing before implementation. Future extensions: favourites/watchlist feeding alerts, per-station freshness indicators app-wide, and eventual EV-charging price coverage.

## 11. Conclusion

Servo Saver is a well-conceived public service that makes a volatile, opaque market transparent. The central search journey is fast and resilient, services are clearly defined per actor, the structure cleanly separates stations/prices/types/caps, and published prices are trustworthy under a disciplined validate → activate → stale → archive → destroy lifecycle. Its defining weakness - working only when the motorist remembers to check - is addressed directly by **Price Drop Alerts**, which integrates cleanly with the existing system by reusing the account, location, price and cap components already in place.

## References

- Airship. (2024). *The 2024 mobile app retention benchmarks: How push notifications impact retention* [Benchmark report]. Airship.
- Australian Competition and Consumer Commission. (2024). *There are tools available to save money on fuel* [Media release]. ACCC.
- Consumer Affairs Victoria. (2025). *Mandatory fuel price reporting: Obligations for retailers*. State Government of Victoria.
- IBISWorld. (2025). *Fuel retailing in Australia* [Industry report]. IBISWorld.
- Premier of Victoria. (2025). *Daily fuel price cap now in place to stop price gouging* [Media release]. State Government of Victoria.
- Service Victoria. (2025). *Servo Saver: How the fuel price cap works*. State Government of Victoria.
- Victorian Auditor-General's Office. (2020). *Service Victoria - Digital delivery of government services*. VAGO.

---

## Repository Contents

| File | Description |
|---|---|
| [Servo Saver Individual Project Report.pdf](Servo%20Saver%20Individual%20Project%20Report.pdf) / [.docx](Servo%20Saver%20Individual%20Project%20Report.docx) | Full report (source of this README) |
| [Activity_Diagram.pdf](Activity_Diagram.pdf) | UC1 activity diagram |
| [Use_case_Diagram.pdf](Use_case_Diagram.pdf) | Overall (legacy) use case diagram |
| [Use_case_Improvised.pdf](Use_case_Improvised.pdf) | Improved use case diagram with Price Drop Alerts |
| [Expanded Use Cases.pdf](Expanded%20Use%20Cases.pdf) | Expanded UC1 description |
| [Expanded Use Cases Improvised.pdf](Expanded%20Use%20Cases%20Improvised.pdf) | Expanded "Set price drop alert" description |
| [Class Diagram.pdf](Class%20Diagram.pdf) | Legacy class diagram |
| [Class Diagram_improvised.pdf](Class%20Diagram_improvised.pdf) | Improved class diagram (PriceAlert, AlertNotification) |
| [CRC Card.pdf](CRC%20Card.pdf) | Legacy CRC cards (FuelStation, FuelPrice) |
| [CRC_Cards_Improvised.pdf](CRC_Cards_Improvised.pdf) | New CRC cards (PriceAlert, AlertNotification) |
| [State Machine Diagram.pdf](State%20Machine%20Diagram.pdf) | FuelPrice state machine |
