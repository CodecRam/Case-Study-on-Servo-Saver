# Servo Saver — Product Analysis

A Business Analyst's UML case study of **Servo Saver**, the fuel-price feature inside the Service Victoria app. It deconstructs the motorist's experience, evaluates the interface, and proposes one well-evidenced improvement: **Price Drop Alerts**.

*Individual project by Ramkrushna Bhadekar.*

> Servo Saver lets Victorian motorists compare government-reported fuel prices at nearby stations, highlights the cheapest options, and shows each retailer's locked maximum price for the next day.

📊 **[Interactive dashboard](dashboard.html)** · 📄 **[Full report (PDF)](Servo%20Saver%20Individual%20Project%20Report.pdf)** ([.docx](Servo%20Saver%20Individual%20Project%20Report.docx))

---

## The app

| | | | |
|---|---|---|---|
| ![Map view](Screenshots/Screenshot1_servo%20saver.PNG) | ![List view](Screenshots/Screenshot2_servo%20saver.PNG) | ![Fuel type](Screenshots/Screenshot3_servo%20saver.PNG) | ![Filters](Screenshots/Screenshot4_servo%20saver.PNG) |

*Scope: analysed from the **motorist's perspective only**. Retailer, admin and regulator roles appear in the models for context.*

## Why it matters

Australian fuel retailing is a ~$58.7B industry (IBISWorld, 2025) where sharp daily price cycles mean a Melbourne motorist buying at the cycle low could save ~$333/year (ACCC, 2024). Victoria's Fair Fuel Plan mandates real-time price reporting and a daily price cap — submitted 2pm, published 4pm, locked from 6am for 24h (Premier of Victoria, 2025). Servo Saver (launched Oct 2025) is the consumer face of that reform.

For the motorist it answers two questions: *"where's the cheapest fuel near me now?"* and *"fill up today or wait?"* The remaining gaps are about **reach and trust**, not data quality — the service is entirely pull-based and shows a single app-wide freshness timestamp.

## What's modelled

| Artifact | What it shows | Files |
|---|---|---|
| **Activity diagram** | The search journey (UC1) across swimlanes — GPS-vs-manual fallback, parallel price/cap retrieval, empty-state handling. | [PDF](Activity_Diagram.pdf) |
| **Use case diagram** | Every service by actor. Notably, *no* use case has the system contact the motorist first — the gap the improvement fills. | [Legacy](Use_case_Diagram.pdf) · [Improved](Use_case_Improvised.pdf) |
| **Expanded use cases** | Fully-dressed UC1 (search) and the new *Set price drop alert*, with exception flows. | [Legacy](Expanded%20Use%20Cases.pdf) · [Improved](Expanded%20Use%20Cases%20Improvised.pdf) |
| **Class diagram + CRC** | `FuelStation` composing `FuelPrice`/`DailyPriceCap`; `User → RegisteredAccountHolder`. Improvement adds `PriceAlert` + `AlertNotification`, purely additively. | [Class](Class%20Diagram.pdf) · [Improved](Class%20Diagram_improvised.pdf) · [CRC](CRC%20Card.pdf) · [Improved](CRC_Cards_Improvised.pdf) |
| **State machine** | The full life of one `FuelPrice`: validate → activate → stale → archive → destroy, with a legal-cap check before going live. | [PDF](State%20Machine%20Diagram.pdf) |

## Interface evaluation

- **Good** — map-first colour-coded pins with a "Lowest" badge; a "tomorrow's price cap" panel for the fill-now-or-wait call; strong filters; inherits Service Victoria's trust and sign-in.
- **Bad** — entirely pull-based (no way to be told when a price drops); no watchlist; one app-wide timestamp instead of per-station freshness; non-combinable sort options.
- **Ugly** — a prominent "Opening hours not available" line that reads as broken; inconsistent brand logos falling back to a grey placeholder.

## Proposed improvement — Price Drop Alerts

A registered account holder sets a **fuel type, target price, radius and optional quiet hours**; the system watches newly activated prices and notifies them when a nearby station hits the target — turning a tool you must remember to check into a proactive assistant.

It's an **integration, not a bolt-on**: reuses the existing *Detect user location* service, validates targets against the same locked `DailyPriceCap`, and listens for the same `FuelPrice → Active` event the state machine already raises.

| Value driver | Mechanism | Evidence |
|---|---|---|
| Engagement & retention | Opt-in push at the moment value is highest (~3× retention) | Airship (2024) |
| Motorist savings | Surfaces the cycle low as it happens | ACCC (2024), ~$333/yr |
| Policy alignment | Steers demand toward competitively priced stations | Premier of Victoria (2025) |
| Low build cost/risk | Reuses location, price, cap and account components | This report |

**Trade-offs:** notification fatigue (opt-in, user thresholds, quiet hours, 10-alert cap); privacy (location/preference storage flagged for a security review before build).

## Repository contents

- **[dashboard.html](dashboard.html)** — interactive summary dashboard
- **[Servo Saver Individual Project Report.pdf](Servo%20Saver%20Individual%20Project%20Report.pdf)** / [.docx](Servo%20Saver%20Individual%20Project%20Report.docx) — the full report this README summarises
- **`Screenshots/`** — live-app captures
- **UML PDFs** — activity, use case, expanded use cases, class, CRC and state machine diagrams (legacy + improved variants), linked in the table above

## References

Airship (2024) · ACCC (2024) · Consumer Affairs Victoria (2025) · IBISWorld (2025) · Premier of Victoria (2025) · Service Victoria (2025) · VAGO (2020). Full citations in the [report](Servo%20Saver%20Individual%20Project%20Report.pdf).
