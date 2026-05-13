# adao-positions-data_2026

Auto-generated weekly snapshots of TLA portfolio data for named aDAO members. Written every Monday at 01:00 UTC by the [adao-positions cron](https://github.com/defipatriot/cron-scripts/tree/main/adao-positions).

## Files

### `data/members.json`

Lightweight metadata for all aDAO members (157 currently, named and unnamed). Rewritten every cron run.

Use this for:
- Member rosters and counts
- Cross-cron member lookups
- Identifying which addresses have registered names on daodao.zone

### `data/current.json`

Full portfolios for every named DAO member (~46 currently). Rewritten every cron run.

Each member's portfolio includes:

- `summary` — top-line numbers (total VP, locked LUNA equivalent, LP USD, pending rewards, potential VP gain from adjusting locks)
- `voting` — current epoch vote allocations per bucket
- `vp_per_pool[]` — user's voting power in each pool they voted for
- `locks[]` — every voting-escrow NFT with projection of VP-if-adjusted
- `lp_positions[]` — amplified and non-amplified LP stakes with USD value, pool status flags, underlying token amounts
- `pending_rewards[]` — unclaimed zluna per pool
- `pending_rebase` — gauge controller rebase
- `pending_bribes[]` — claimable bribes with USD pricing
- `wallet_balances[]` — TLA-relevant wallet tokens

Use this for:
- Per-member portfolio dashboard views
- Real-time alerts (members with at-risk LP positions, lock adjustment opportunities)
- Top-level totals across the DAO

### `data/weekly/epoch-{N}.json`

Frozen archive of the portfolio state at each epoch boundary, named by the epoch number at capture time. Accumulate over time.

Use these for:
- Epoch-over-epoch performance charts ("how did your LUNA-USDC position change this week?")
- Historical lookback ("what was my position 4 epochs ago?")
- Yield calculation in token terms vs USD terms ("token amounts up 2% / USD value down 3%")

Each file has the same structure as `current.json`.

## Key concepts

### Voting power projection

Every lock has a `projection` field that shows what its voting power **would be** if the user adjusted the lock right now using current LST ratios. Adjusting an LST lock (ampLUNA, bLUNA, arbLUNA, stLUNA) re-snapshots the underlying LUNA at current ratios — meaning users can claim "free" voting power as their LST grows from staking yield.

Example: A lock of 100 ampLUNA captured 165 underlying LUNA at lock time. The current ampLUNA ratio is 1.74, so the user's lock represents 174 LUNA underlying today. Adjusting the lock would update voting power from 1,485 (165 × 9 coefficient) to 1,566 (174 × 9) — a free gain of 81 VP.

### Pool status flags

Each LP position has a `status` field reflecting whether the pool is currently earning rewards:

- `active` — pool has ≥1.5% of bucket voting power (comfortably earning)
- `at_risk` — pool has 1.0-1.5% of bucket voting power (close to losing rewards)
- `inactive` — pool has <1.0% of bucket voting power (earning $0 — needs vote redirection)

### Amplification

Each LP position has an `is_amplified` boolean and a `source` field:

- `is_amplified: true, source: "asset_compounder"` — auto-compounding amplified position (Eris's flame icon)
- `is_amplified: false, source: "staking_contract"` — raw LP stake (only earns base TLA rewards)

## Member discovery

The cron uses three data sources, in order:

1. **`indexer.daodao.zone`** (primary, auto-updating) — fetches the current list of staked NFT owners and their voting power percentages directly from the DAO contract via DAO DAO's indexer
2. **`pfpk.daodao.zone`** (names) — resolves on-chain addresses to their registered profile names
3. **`github.com/defipatriot/adao_json_storage/main/members.csv`** (fallback) — DefiPatriot's manually maintained CSV, used if daodao.zone is unreachable

When a new DAO member registers a name on daodao.zone, the next cron run picks them up automatically.

## Schedule

Mondays at 01:00 UTC — runs ~1 hour after the TLA epoch boundary, capturing post-rewards-settlement state.

## Schema versioning

All files include a `schemaVersion` field (currently `1`). Consumers should check this before parsing to handle future migrations gracefully.

## Source code

The cron implementation lives at `github.com/defipatriot/cron-scripts/tree/main/adao-positions`. See its README for technical details on the query strategy, fallback behavior, and deployment.
