# ⚡ VoltRelay Energy — Network Performance, Service Failures & Retention Diagnostic

**Data Analytics Hackathon · Gradient Learnings · 26 Sept 2026**

> Business Analytics · Operations · Customer Experience · Battery Analytics · Station Performance

---

## 📌 Table of Contents

- [Project Overview](#-project-overview)
- [Business Objective](#-business-objective)
- [The Six Business Questions](#-the-six-business-questions)
- [How the Questions Connect](#-how-the-questions-connect)
- [Dataset Overview](#-dataset-overview)
- [Data Relationships](#-data-relationships)
- [Dataset Details](#-dataset-details)
- [Data Cleaning](#-data-cleaning)
- [Methodology by Question](#-methodology-by-question)
- [Budget Decision Framework](#-budget-decision-framework)
- [Known Limitations](#-known-limitations)
- [How to Run](#-how-to-run)
- [Repository Structure](#-repository-structure)

---

## 📌 Project Overview

**VoltRelay Energy** is a battery-swapping network serving electric two-wheelers (2W) and three-wheelers (3W) across six Indian cities:

- Bengaluru
- Delhi NCR
- Hyderabad
- Pune
- Mumbai
- Jaipur

The network has seen strong growth in swaps and revenue, but leadership has flagged several concerning signals:

- Service failures are rising
- New-rider retention is declining
- Contribution margin per swap is shrinking
- Operational performance varies sharply across stations and cities
- Battery health differs across suppliers and manufacturing lots
- Customer complaints point to real service friction
- Fleet-partner contracts may carry very different economics

Leadership needs an **evidence-based answer** to one question:

> **Where is the network actually breaking, what factors are associated with those problems, and where should the next operating budget go?**

### The four budget options on the table
1. Build more stations
2. Buy more batteries
3. Roll out network-wide pricing changes
4. Sign a long-term exclusive with the largest fleet partner

This project analyzes VoltRelay's operational data to weigh the evidence behind each option.

---

## 🎯 Business Objective

Diagnose network performance and identify the operational and customer-experience factors associated with:

- Service failures, long queues, and abandoned swaps
- Station-level and geographic performance gaps
- Battery degradation and equipment/cohort differences
- Customer complaints and support friction
- Pricing and fleet-partner economics
- New-rider retention

---

## 🧭 The Six Business Questions

| # | Theme | Question |
|---|--------|----------|
| **Q1** | 📈 Growth | Is the network growing in a healthy way — swaps, revenue, failure rate, margin per swap over time? |
| **Q2** | 🚨 Failures | When, where, and for whom are service failures and customer friction concentrated? |
| **Q3** | 🗺️ Stations & Geography | Which cities, station types, charger generations, or expansion cohorts are associated with poor performance? |
| **Q4** | 🔋 Batteries | Are specific suppliers, manufacturing lots, or equipment cohorts associated with weaker battery health? |
| **Q5** | 💰 Pricing & Partners | Do pricing structures and fleet-partner contracts produce sustainable revenue and margin? |
| **Q6** | 👥 Retention | Which first-experience factors are associated with whether a new rider comes back? |

**Important framing:** this is observational data. Every finding below is described as an *association*, never a proven cause.

---

## 🔥 How the Questions Connect

```
                    NETWORK HEALTH
                          │
                          ▼
             Q1 — Is growth healthy?
                          │
                          ▼
               CUSTOMER EXPERIENCE
                          │
                          ▼
         Q2 — Where/when do failures happen?
                          │
                          ▼
              STATION / GEOGRAPHY
                          │
                          ▼
       Q3 — Which locations/equipment cluster?
                          │
                          ▼
                BATTERY HEALTH
                          │
                          ▼
      Q4 — Which battery cohorts underperform?
                          │
                          ▼
                    ECONOMICS
                          │
                          ▼
      Q5 — Are pricing & partner terms sound?
                          │
                          ▼
                    RETENTION
                          │
                          ▼
     Q6 — Are poor experiences tied to churn?
                          │
                          ▼
              ┌────────────────────┐
              │   BUDGET DECISION  │
              └────────────────────┘
```

This is not six disconnected analyses — it's one story: **understand network health → find where it breaks → find why → connect it to money and retention → recommend where to spend.**

---

## 📊 Dataset Overview

| Dataset | Approx. Rows | Purpose |
|---|---|---|
| `swap_events` | ~3.88M | Swap attempts, outcomes, pricing, wait time, battery IDs |
| `station_hourly_status` | ~1.49M | Hourly station & equipment telemetry |
| `riders` | 20,000 | Rider profile & signup information |
| `batteries` | 6,500 | Battery health, supplier, manufacturing lot |
| `support_tickets` | 44,000 | Complaints and support interactions |
| `stations` | 152 | Station configuration and geography |
| `city_daily_context` | 3,282 | Weather, events, grid, competitor context |
| `fleet_partners` | 12 | Fleet contract and pricing terms |

> ⚠️ **`swap_events` note:** at ~3.88M rows, this file is too large to upload directly and is analyzed **in Colab** (read from `/content/swap_events.csv.gz`), not in this repo. The notebook checks for its presence and marks any dependent section `PENDING — swap_events not found` rather than fabricating numbers when it isn't available in a given run.

---

## 🔗 Data Relationships

```
                  fleet_partners
                        │
                        ▼
                     riders
                        │
                        ▼
                  swap_events
                  /         \
                 ▼           ▼
            batteries     stations
                              │
                              ▼
                  station_hourly_status

riders ─────────────► support_tickets
stations + date ────► city_daily_context
```

This lets analysis connect: rider → swap behavior → station → battery → city → external context, and rider → support experience → fleet partner.

---

## 📁 Dataset Details

### 1. `swap_events` — one row per swap attempt
Key fields: `event_id`, `rider_id`, `station_id`, `event_ts`, `event_type`, `queue_wait_sec`, `battery_in_id`, `battery_out_id`, `soc_in_pct`, `soh_in_pct`, `soc_out_pct`, `soh_out_pct`, `km_since_last_swap`, `tariff_code`, `list_price_inr`, `discount_inr`, `amount_charged_inr`, `payment_mode`, `energy_to_recharge_kwh`, `station_firmware`, `sync_mode`.

Outcomes include: `swap_completed`, `failed_no_charged_battery`, `abandoned_queue`, `cancelled_by_rider`, `failed_system_error`.

Powers: Q1, Q2 (detailed), Q5, Q6.

### 2. `station_hourly_status` — hourly station telemetry
Key fields: `charged_2w_avg`, `charged_2w_min`, `charged_3w_min`, `packs_charging`, `packs_quarantined`, `chargers_online`, `ambient_temp_c`, `cabinet_temp_c`, `avg_charge_minutes`, `outage_minutes`, `grid_kwh`, `telemetry_status`.

Powers: station health, equipment availability, outages, battery quarantine, city/charger-generation comparisons.

### 3. `riders` — 20,000 records
Key fields: `rider_id`, `partner_id`, `vehicle_class`, `vehicle_model`, `home_city`, `home_zone`, `signup_date`, `signup_channel`, `plan_type`, `declared_shift`, `age_band`, `kyc_verified`.

Powers: segmentation, fleet vs. independent comparisons, retention analysis.

### 4. `batteries` — 6,500 records
Key fields: `battery_id`, `pack_type`, `supplier`, `manufacturing_lot`, `manufacture_date`, `commission_date`, `rated_capacity_kwh`, `purchase_cost_inr`, `initial_soh_pct`, `bms_firmware`, `current_soh_pct`, `retired_date`, `retirement_reason`.

Powers: Q4 (battery/equipment analysis).

### 5. `support_tickets` — 44,000 records
Key fields: `ticket_id`, `rider_id`, `station_id`, `battery_id`, `created_ts`, `channel`, `category`, `rider_comment`, `priority`, `resolution_status`, `resolution_hours`, `csat_score`.

Powers: customer friction, complaint concentration, support workload.

### 6. `stations` — 152 records
Key fields: `station_id`, `city`, `zone`, `latitude`, `longitude`, `location_type`, `host_type`, `commissioned_date`, `decommissioned_date`, `expansion_wave`, `charger_generation`, `slots_2w`, `slots_3w`, `inventory_target_2w`, `inventory_target_3w`, `monthly_rent_inr`, `monthly_maintenance_inr`, `grid_tariff_inr_kwh`, `connectivity_tier`, `firmware_version`, `firmware_updated_date`, `competitor_within_1_5km_since`.

### 7. `city_daily_context` — 3,282 records
Daily external context per city: temperature, rainfall, heat alerts, flood disruption, festivals/events, public holidays, grid outages, competitor promotions, e-commerce events.

### 8. `fleet_partners` — 12 records
Key fields: `partner_id`, `partner_name`, `partner_segment`, `vehicle_class`, `contract_type`, `contract_start_date`, `discount_pct`, `amendment_date`, `discount_pct_after_amendment`, `peak_surcharge_billable`, `payment_terms_days`, `cities_active`.

---

## 🧹 Data Cleaning

1. **Test stations excluded** — any station ID prefixed `STN-TST` is dropped from operational analysis.
2. **City name standardization** — variants like *Bangalore / BLR → Bengaluru*, *Delhi / NCR / New Delhi → Delhi NCR*, *Bombay → Mumbai*, *HYD → Hyderabad* are unified so city-level analysis isn't split across spellings.
3. **Timestamp correction** — stations on firmware `v3.2.0` between **10 Mar – 14 Apr 2025** had a known 5.5-hour timestamp shift, corrected before any hour-of-day analysis.
4. **Offline duplicate handling** — poor-connectivity stations can log near-duplicate events from offline sync; records with the same rider + station + battery combination within a short time window are flagged rather than double-counted.
5. **Numerical outlier handling** — negative/implausible `km_since_last_swap` values are treated as missing; `soc_pct`/`soh_pct` values above 100% are capped at 100%.
6. **Missing telemetry ≠ zero activity** — missing hourly telemetry is left as missing, not silently converted to zero, to avoid manufacturing false operational signals.

---

## 🔬 Methodology by Question

### Q1 — Network Performance Over Time
Tracks four series over the ~18-month period: completed swaps, revenue, failure rate, and contribution margin per swap.

```
Failure Rate      = Unsuccessful Attempts / Total Swap Attempts
Contribution Margin = Amount Charged − Energy Cost
Energy Cost        = Energy Used (kWh) × Grid Tariff (₹/kWh)
```

**Signal to watch for:** swaps ↑, revenue ↑, but failure rate ↑ and margin ↓ → *growth is happening, but its quality and economics are deteriorating.*

### Q2 — Service Failures & Customer Experience
Breaks failure rate down by **station**, **hour of day**, **season**, and **vehicle class (2W vs 3W)**, and checks whether a small share of stations accounts for a disproportionate share of failures. Cross-references support ticket categories (queue length, battery availability, billing, low range, technical issues).

### Q3 — Station & Geographic Patterns
Compares outage minutes, quarantine rates, charging time, and complaints across **city**, **charger generation**, **location type** (transit hub, commercial, roadside), **expansion wave**, and **connectivity tier**. Uses complaints-per-1,000-swaps to normalize for station volume.

### Q4 — Battery & Equipment Performance
Compares **State of Health (SOH)** — current battery condition relative to new — across **supplier** and **manufacturing lot**, and links battery IDs in support tickets back to battery health. Explicitly framed as association, not causation.

### Q5 — Pricing & Partner Economics
Compares volume, revenue, and margin across **tariff codes** (`STD`, `PEAK`, `OFFPEAK`, `PREPAID`, `PARTNER`, `PROMO_FREE`) and across **fleet partners**, factoring in discount %, contract type, and payment terms — since high volume doesn't automatically mean high profitability.

### Q6 — Rider Retention
Defines a retained rider as one who completes another swap within **30 days** of their first completed swap, and compares retention across first-swap outcome, queue wait bucket, signup channel, fleet vs. independent status, and KYC verification.

---

## 🧩 Budget Decision Framework

| Option | Key evidence to weigh |
|---|---|
| **More stations** | Station-level demand, failure concentration, queue friction, city-level gaps, capacity/outage patterns |
| **More batteries** | Supplier/lot-level SOH gaps, quarantine rates, complaint linkage |
| **Network-wide pricing rollout** | Whether pricing shifts usage meaningfully, or whether effects are segment-specific |
| **Fleet exclusive (largest partner)** | Partner volume **and** margin, discount terms, payment terms, operational dependency |

**Framing for the final recommendation:** don't spend evenly across the network — target the specific supplier, the specific high-failure station clusters, and the first-swap experience, because those are what's actually driving failure rate, churn, and margin pressure.

---

## ⚠️ Known Limitations

- All findings are **associations**, not proven causes — the data is observational.
- `swap_events` (~3.88M rows) is analyzed in Colab and not included in this repo; any section depending on it is marked `PENDING` when the file isn't present in a given run.
- Manufacturing-lot and supplier signals should be treated as leads for further investigation, not confirmed root causes.
- Seasonal and external-context comparisons (weather, festivals) are directional, not statistically adjusted for confounders.

---

## ▶️ How to Run

1. Open `VoltRelay_Hackathon_Analysis.ipynb` in Google Colab.
2. Upload the seven smaller CSVs (`batteries.csv`, `city_daily_context.csv`, `fleet_partners.csv`, `riders.csv`, `station_hourly_status.csv`, `stations.csv`, `support_tickets.csv`) to the Colab session, or mount Drive.
3. Place `swap_events.csv.gz` at `/content/swap_events.csv.gz` in the Colab session (not in this repo).
4. Run all cells top to bottom — the notebook detects whether `swap_events` is present and skips/labels dependent sections accordingly.
5. Review the Q1–Q6 sections and the Budget Decision summary at the end.

---

## 🗂️ Repository Structure

```
.
├── README.md
└── VoltRelay_Hackathon_Analysis.ipynb
   (all CSVs + swap_events.csv.gz loaded directly into the Colab session, not committed to this repo)
```

---

*Prepared for the Gradient Learnings Data Analytics Hackathon — 26 Sept 2026.*
