# pharmQ data schema

All tables are plain CSV. The bundled copies live in `pharmq/data/<region>/` (Python) and `r/pharmqR/inst/extdata/<region>/` (R), with regions `usa` and `africa`. Keep the two in sync with `make sync-data`.

In the Africa pack, the `state` column holds the **ISO 3166 alpha-3 country code** (NGA, KEN, ZAF...), `dea_vault` means a licensed controlled-substance vault, and `plants.csv` also lists **import gateways** (`GW-*`: seaports and air-cargo hubs), since most medicines in Africa are imported.

## states.csv: one row per state / country
| column | type | meaning |
|---|---|---|
| state | text | US 2-letter state code, or ISO3 country code (key) |
| name | text | full name |
| region | text | sub-region (US Census regions; Africa: North, West, Central, East, Southern) |
| population | int | residents (US 2020 Census; Africa 2024 UN estimate) |
| lat, lon | float | population centroid |
| island | 0/1 | 1 = reachable only by air/sea (AK, HI, PR; MDG, MUS, CPV, COM, STP, SYC) |

## zones.csv (optional): your own demand zones; replaces the zones built from states
| column | type | meaning |
|---|---|---|
| zone_id | text | unique id (e.g. a county FIPS or 3-digit ZIP) |
| state, region | text | used for air rules and filtering |
| population | int | residents served |
| lat, lon | float | centroid |
| island | 0/1 | as above |
| pharmacies | int | number of dispensing sites (informational) |

## dc_sites.csv: candidate distribution centers
| column | type | meaning |
|---|---|---|
| site_id | text | unique id |
| city, state | text | location |
| lat, lon | float | coordinates |
| fixed_cost_musd | float | annual fixed cost if open, $ millions |
| capacity_mpacks | float | annual throughput capacity, millions of packs |
| cold_chain | 0/1 | can store 2–8 °C products |
| dea_vault | 0/1 | licensed vault for Schedule II controlled substances |

## plants.csv: manufacturing / import sources
| column | type | meaning |
|---|---|---|
| plant_id | text | unique id |
| name, state | text | description |
| lat, lon | float | coordinates |

## products.csv: SKU catalog
| column | type | meaning |
|---|---|---|
| sku | text | unique id |
| name, therapeutic_class | text | description |
| handling | ambient / cold / controlled | storage & transport class |
| unit_cost | float | $ per pack |
| annual_packs_per_1k | float | packs per 1,000 residents per year |
| demand_cv | float | coefficient of variation of weekly demand |
| season_amp | float | seasonality strength (0 = flat, ~1 = strong) |
| season_peak_month | 1–12 | month of peak demand |
| trend_pct | float | annual growth % |
| shelf_life_days | int | shelf life at manufacture |
| lead_time_days, lead_time_sd | float | replenishment lead time from plant (mean, sd) |
| primary_plant | text | `plant_id` of main source |
| backup_plant | text or blank | `plant_id` of qualified second source |

## Outputs (`results/`)
| file | content |
|---|---|
| open_dcs.csv | selected DCs with throughput, utilization, zones served |
| lanes.csv | zone × handling → DC assignment, miles, km, mode, cross_border, transit days (incl. customs), annual cost |
| inventory_policy.csv | DC × SKU safety stock, reorder point (avg & peak), order qty, days of supply, value |
| simulation.csv | DC × SKU simulated demand, fill rate, stockout days, expired packs |
| shortage_risk.csv | SKU risk score, tier, main driver, component scores |
| forecast.csv / forecast_accuracy.csv | 26-week forecast with 95% band; back-test accuracy |
| zones.csv / demand.csv | inputs actually used |
| summary.json / config.json | headline KPIs and the exact configuration |
