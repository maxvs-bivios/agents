# renovara-nem-connector-gotchas

> Renovara NEM Connector (Databricks MCP) — what's not loaded, permission gaps, and the price-setter trick for inferring generator bids
>
> Moved from Claude's memory (Kryten session store) on 6 Oct 2026; last updated there 2026-09-18. Keep this file current; it replaces the memory note.

Renovara NEM Connector gotchas (as of 18 Sep 2026):

- **Bid tables are not loaded** (`BIDDAYOFFER`, `BIDPEROFFER_D` etc. — all 9 BIDS tables missing). Cannot read bid stacks directly.
- **`v_duid_fuel_aer` view is not permitted** (INSUFFICIENT_PRIVILEGES). Use `silver_nem_participant_and_scheduled_loads` (dedupe with `ROW_NUMBER() OVER (PARTITION BY DUID ORDER BY FUEL_SOURCE_PRIMARY)=1`) for fuel/region lookup. Note QLD "Coal" filter also catches CSM/waste-coal-mine-gas — filter `FUEL_SOURCE_DESCRIPTOR='Black Coal'`.
- **Bid-price proxy:** `silver_pricesetter_price_setter.RRNBANDPRICE` (filter `MARKET='Energy'`, `DISPATCHEDMARKET='ENOF'`) reveals a unit's offer band price whenever it is marginal. Data lags ~1 month (ran to 2026-08-01 on 18 Sep). Only reveals bands the unit is actually marginal on — deep-negative min-load blocks are under-sampled; cross-check with `dispatch_unit_solution` TOTALCLEARED/AVAILABILITY vs RRP bins.
- Skill files live at ~/Coding/agents/skills/renovara_ai_nem_analyst/references (symlinked into ~/.claude/skills).

**How to apply:** For any "what does X bid" question, go straight to price-setter + dispatch-ratio approach; don't waste a query on bid tables or the fuel view.
