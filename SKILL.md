---
name: florida-material-risk
description: Read-only material planning for PMGT - Inventory Planning V6.0: Florida or other site forecast inventory shortfalls and safety stock, affected finished goods from the HPE BOM PDF, largest-excess donor locations, and committed purchase orders to pull forward. Use for full reports or narrow follow-ups; default to Scenario 1 unless the user explicitly selects another scenario. Load only the relevant workflow.
---

# Fast materials planning router

Use only the named Anaplan model and supplied HPE BOM PDF; read-only. Never write, import, run processes, or imply an order or transfer was executed. If a needed source cannot be read, stop and state the limitation. No outside facts about inventory, lead times, suppliers or substitutes.

## Route once (do not load unrelated references)
- Forecast inventory, below zero/safety stock, material shortfalls -> `references/shortfalls.md`.
- Impacted finished goods/BOM/buildability -> `references/finished-goods.md`.
- Best donor/largest excess/transfer -> `references/transfers.md`.
- Committed/open PO pull-forward -> `references/purchase-orders.md`.
- Full planning report -> read all four, reuse findings from each stage rather than repeat the same reads.

## Defaults and economical reads
1. **Scenario 1 is the default whenever the user has not specified a scenario. Do not ask them to choose among scenarios.** An explicit scenario overrides the default; never combine scenarios. Scenario may be a list rather than native Versions: identify its actual axis. If module has native Versions, pin exactly one version per query. If no location/horizon is provided, use all stock-holding locations and the next model year from the model current period, and say so. Carry the user's chosen location, scenario and horizon across follow-ups unless changed.
2. Check `get_model_context` once per conversation/session; bind the named model only if needed. Reuse previously verified module, dimension, line-item, and BOM code mappings in this conversation. Discover only missing schema, not the whole model on every follow-up. Verify model clock for relative time using `list_time_anchors`, resolve any conflicting candidates with user and call `set_time_anchor`; for week spans, use model-calendar membership, not arithmetic on week labels. Do not separately query `SYS003` if it is already covered by anchor discovery. On follow-up, use an existing valid anchor for the same bound model when available; refresh it if current period may have moved or the user asks for a fresh horizon. The model's FY labels need not match today's date.
3. Answer the **question actually asked**: a PO follow-up should not re-read every site's forecast; a transfer follow-up should not search PO lines; a BOM follow-up can use the previously established shortlist **explicitly labelled as prior-read**, unless the user asks for refreshed inventory. Re-read only cells needed for an up-to-date claim. For a new independent inventory assessment, fully read its stated location/material/period scope. Favor `read_cells` with complete dimension slice and one or two leaf axes, pagination and a practical limit; if broad reads time out once, split into bounded slices. Use `sql_schema`/`sql_query` only when aggregation/selection demands them. Do not substitute sampled rows for complete coverage.
4. `Shortfall (U)` means unmet demand, not below-zero inventory; `Excess Available for Push and Pull (U)` is modeled excess, not verified transferable stock. Check line-item meaning/sign/units before calculations. Exact Anaplan code (or documented exact crosswalk) for BOM matching; no guess from name. Missing inventory, BOM, build demand or lead time stays unknown. Use severity Critical within 4 model weeks or demonstrably unbuildable; High weeks 5–8; Medium later or safety-stock-only. A material shortfall alone does not prove unbuildable FG units.
5. State locations, Scenario 1 (or selected scenario), model-period horizon and **actual data-read date**. Distinguish current reads from prior conversation results and disclose incomplete coverage. Rank action checks by likely speed, but don't assert feasibility without stock release, transit/lead time or live PO confirmation.
