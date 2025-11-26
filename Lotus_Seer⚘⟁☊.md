# Lotus_Seer⚘⟁☊ · Unified Architecture

> "Price tells you what the crowd thinks. Smart money tells you what reality thinks. We trade the gap between them."

We do care about price—**relative to who is on each side**.

---

## Core Insight

- **Trade trigger:** `Seer_Prob(side) - Market_Prob(side) > threshold`
- **Seer_Prob:** wallet-weighted collective intelligence (smart money)
- **Market_Prob:** AMM price (crowd liquidity)
- Inspiration: Renaissance/Jim Simons — every trader is a factor; trade deviations from consensus.

---

## Scope-Centric Intelligence

Everything in Seer is scope-based. Wallets do not have global skill; they have **skill-in-context**.

A scope is defined by:

1. `market_category` (politics, sports, crypto, macro)
2. `odds_bucket` (longshot <20%, mid 20-60%, favorite >60%)
3. `bet_regime` (small / normal / large per wallet)
4. `timing` (early / late relative to resolution)
5. `insider_likelihood` (high / medium / low for market type)

Example — Wallet `0xABC`:

- **Elite** in S1 = `(cat: POL, odds: MID, reg: LARGE, time: LATE)` → 78% win rate
- **Trash** in S2 = `(cat: SPORT, odds: LOW, reg: SMALL, time: EARLY)` → 30% win rate

Same wallet, different contexts → different intelligence. We do **not** track “good vs bad” traders; we track which scopes each wallet succeeds in.

---

## The Seer Stack (Conceptual Pipeline)

```
┌───────────────────────────────┐
│      Market Archetype         │
│  (insider, volatility, type)  │
└───────────────────────────────┘
              │
              ▼
┌───────────────────────────────┐
│         Scope (S*)            │
│ category / odds / regime/time │
└───────────────────────────────┘
              │
              ▼
┌───────────────────────────────┐
│   wallet_scope_stats[W,S*]    │
│   (Bayesian → logit skill)    │
└───────────────────────────────┘
              │
              ▼
┌───────────────────────────────┐
│     smart_weight(W)           │
│   skill × conviction × bankroll│
└───────────────────────────────┘
              │
              ▼
┌───────────────────────────────┐
│ YES_strength / NO_strength    │
└───────────────────────────────┘
              │
              ▼
┌───────────────────────────────┐
│    Seer_Prob vs Market_Prob   │
└───────────────────────────────┘
              │
              ▼
┌───────────────────────────────┐
│   Confidence → Position Size  │
└───────────────────────────────┘
```

---

## Unified Scoping Framework

### Promotion (Discovery) · Hierarchical Criteria

A wallet becomes “smart money” in scope `S` via weighted evidence.

**Tier 1 · Strong Evidence (3x weight each)**

- **Repeated success in scope**
  - Win rate >60% with `n ≥ 20` in scope `S`
  - Category specialist: consistent wins in politics / sports / macro
- **High-conviction signal**
  - Large bet (top 10% for this wallet) in scope `S`
  - High bankroll % (>20% of USDC + positions)
- **Deposit spike**
  - Fresh deposit >$50k within 48h of bet
  - Possible insider move

**Tier 2 · Supporting Evidence (1-2x weight)**

1. **Scope-matched promotion (2x)**
   - Wallet has proven success in similar scope
   - New bet in that scope → immediate promotion
   - Example: known politics specialist makes large politics bet → track
2. **Coordination cluster (1x)**
   - 3+ known smart wallets enter same side within 2 minutes
   - Fresh wallet entering right after → elevated importance
3. **News-adjacent flow (1x)**
   - Large bet placed within 1 hour of news event
   - Market type with insider potential

**Promotion score (informal v0):**

```
promotion_score =
    3 * repeated_scope_success
  + 3 * high_bankroll_pct
  + 3 * deposit_spike
  + 2 * scope_matched_entry
  + 1 * coordination_cluster
  + 1 * news_adjacent_flow
```

Continuous process: always watching for new smart money via these triggers.

---

## Weighting (Live Markets)

When wallet `W` places a bet in current market with scope `S*`:

1. Pull `wallet_scope_stats` for matching scope.
2. **Exact match** `(category, odds_bucket, bet_regime, timing)` if `n ≥ 30`.
3. Fall back to `(category, odds_bucket)` if `n < 30`.
4. Fall back to `(category)` if still `< 30`.
5. Use global wallet score with heavy discount if `< 30` everywhere.

**smart_weight calculation:**

```
smart_weight(W, market) =
    skill_score(W, S*)          # How good in this scope?
  × conviction_weight(bet)      # Per-wallet bet regime
  × bankroll_weight(bet)        # % of portfolio at risk
```

### Components

- **skill_score (v0)**
  - `bayes_win_rate = (wins + 1) / (trades + 2)` (Jeffreys prior)
  - `skill_score = logit(bayes_win_rate) = ln(p / (1-p))`
  - Small sample → pulls toward neutral; large sample → true skill
  - `logit()` compresses extremes, preserves ordering
- **conviction_weight (per-wallet regime)**
  - Small bet (bottom 50%) → `0.1x`
  - Normal bet (50-90%) → `1.0x`
  - Large bet (top 10%) → `3.0x`
  - Normalized per wallet: $10k might be small for whale, huge for minnow
- **bankroll_weight**
  - `bankroll = USDC_balance + Σ position_values` (on-chain, directly readable)
  - `bankroll_pct = bet_size / bankroll`
  - `bankroll_weight = min(1, bankroll_pct * 3)`
  - $20k wallet placing $5k bet = 25% → strong conviction
  - $500k whale placing $5k bet = 1% → noise
  - Captures fresh deposits, all-in moves, insider signals

**Aggregate side strength:**

```
YES_strength = Σ smart_weight_i  (all wallets on YES)
NO_strength  = Σ smart_weight_j  (all wallets on NO)
Seer_Prob(YES) = YES_strength / (YES_strength + NO_strength)
```

---

## Market Lifecycle · Continuous Monitoring

Seer never locks in a view; markets evolve as smart-money flows change.

**State machine:** `WATCH → PROBE → ACTIVE → EXIT`

- Tick frequency proportional to market volatility and trade flow.
- Slow markets (macro, long-horizon politics): hourly or daily.
- Fast markets (breaking news, high-liquidity events): minutes or sub-minute.
- Adaptive, not fixed window.

**At each tick:**

1. Recompute `smart_weight` for all participants (new wallets enter, old exit).
2. Recalculate `Seer_Prob` vs `Market_Prob`.
3. Update `confidence = |Seer_Prob - Market_Prob|`.
4. Adjust `target_position_size`.

**Behavior:**

- **WATCH:** Monitoring, no position.
- **PROBE:** Enter small position (confidence weak but building).
- **ACTIVE:** Size up as smart-money distribution improves / odds move favorably.
- **EXIT:** Scale down if conviction wallets flip sides or market resolves.

Markets are **not** one-shot entries — we adapt continuously.

---

## Position Sizing · Confidence-Based

Fully flexible up and down; size breathes with confidence.

```
confidence = abs(Seer_Prob - Market_Prob)
conf_entry = 0.05    # enter threshold
conf_exit  = 0.03    # exit threshold (hysteresis)
target_raw = base_size * (confidence / 0.20)  # 20% deviation = 1x
target_raw = min(target_raw, base_size * 3)   # cap at 3x

if confidence < conf_exit:
    target_size = 0  # Exit
elif confidence < conf_entry:
    if current_position == 0:
        target_size = 0  # Don't open yet
    else:
        target_size = target_raw  # Allow shrinking on way out
else:
    if current_position == 0:
        target_size = max(min_position, target_raw)  # floor on new entry
    else:
        target_size = target_raw  # full flexibility once in
```

Key: Floor only applies when entering from zero. Once in, size follows the confidence curve both ways.

**Examples:**

- `Seer_Prob = 65%`, `Market = 45%` → confidence = 20% → `1x` base size
- `Seer_Prob = 75%`, `Market = 45%` → confidence = 30% → `1.5x` base size
- `Seer_Prob = 52%`, `Market = 48%` → confidence = 4% → if in position, scale down; if flat, stay flat

---

## Two Learning Loops

### External Loop · Wallet Classification

Goal: identify when each wallet is predictive.

```
pattern_key = wallet_id
scope = (category, odds_bucket, bet_regime, timing)
Lesson: "Wallet W performs well in scope S"
```

Not doing: optimization, overrides, counterfactuals. **Just** performance tagging per scope. 
Updates are Bayesian; only when new outcomes appear. No arbitrary time decay.

### Internal Loop · Strategy Filtering

Goal: determine which market types are predictable with smart-money weighting.

```
pattern_key = Seer decision pattern (e.g., "smart_majority_YES_politics")
scope = market archetype (market_type, insider_likelihood, volatility, odds_range)
Lesson: "Seer's strategy worked in scope T"
```

Learning goal: per market archetype, does confidence predict success?

Examples:

- High-confidence (>25%) signals in US politics → consistently profitable → maintain
- Mid-confidence (10-20%) signals in low-liquidity sports → mixed results → filter out
- Fresh wallet coordination in macro data releases → strong edge → boost

Simpler than `Lotus_Trader`:

- No PM tuning
- No DM layers
- No action overrides
- Just: “This market type is predictable” vs “This isn’t”

---

## Data Architecture

1. **Raw trades (`seer_trade_events`)** — for every wallet (not just ours)
   - `market_id`, `wallet_id`, `side (YES/NO)`, `size`, `price`
   - `execution_timestamp`, `resolution_timestamp`
   - Scope fields captured at bet time: `market_category`, `odds_bucket`, `bet_size_percentile`, `timing_bucket`, `insider_flags`
2. **Wallet master (`wallet_master`)** — thin identity
   - `wallet_id`, `lifetime_pnl`, `lifetime_win_rate`
   - `first_seen`, `last_seen`
   - `last_observed_bankroll` (cached snapshot, not “estimated”)
   - `fresh_wallet_flag` (low history + large bet + high bankroll%)
   - Not used for smart-money weighting — just existence tracking
3. **Wallet scope stats (`wallet_scope_stats`)** — the brain
   - One row per `(wallet_id, scope)`
   - Example entry:
     - `wallet_id: 0xABC`
     - `scope: {category: "US_election", odds_bucket: "mid", bet_regime: "large", timing: "late"}`
     - `n_trades: 27`, `n_wins: 21`, `win_rate: 0.78`
     - `avg_rr: +0.35`, `roi: +42%`, `bayes_win_rate: 0.75`, `skill_score: 1.10`
   - Updated on every market resolution via aggregation from `seer_trade_events`
   - This is the lookup for smart-money weighting
4. **Seer pattern stats (`seer_pattern_stats`)** — our own decisions
   - `pattern_key` (Seer signal type)
   - `scope` (market archetype)
   - `confidence_series`, `position_size_series`, `field_state_series`
   - `n_trades`, `win_rate`, `avg_rr`
   - Not just boolean “we bet” — we log the full behavioral trajectory to learn:
     - Whether probe → scale works better than all-in
     - Optimal threshold choices
     - When scaling decisions correlated with success
   - Storage: aggregated summary with key decision points (no need for full tick-level forever)
5. **Market participants (optional cache)**
   - Temporary per-market snapshot (rebuildable from `seer_trade_events`)
   - `market_id`, `wallet_id`, `side`, `size`, `entry_price`

---

## Workflow (End-to-End)

### Pre-Trade (Continuous)

1. Ingest all trades from Polymarket API.
2. Tag with scope dimensions.
3. Identify new smart-money wallets (promotion via 6 criteria).
4. Update `wallet_scope_stats` on market resolution.

### Live Market Evaluation (Every Tick)

1. Get current participants (YES / NO sides).
2. For each wallet:
   - Determine bet’s scope.
   - Pull `wallet_scope_stats` for matching scope.
   - Calculate `smart_weight`.
3. Sum → `YES_strength`, `NO_strength`.
4. Calculate `Seer_Prob(YES)`, `Seer_Prob(NO)`.
5. Compare to `Market_Prob` (AMM price).
6. If `confidence > 0.05`:
   - Size position via formula.
   - Enter / adjust position.
   - Update state machine.

### Post-Resolution

1. Record outcome in `seer_trade_events`.
2. Update `wallet_scope_stats` for all participants (Bayesian update).
3. Update `seer_pattern_stats` for our decision.
4. Scan for newly qualified smart wallets.
5. Inactive wallets naturally drop to zero influence.

---

## Scope Definitions (v0)

### Wallet Scopes

- `market_category`
  - politics (US_election, intl_politics)
  - sports (NFL, NBA, soccer, tennis)
  - crypto (token_price, protocol_event)
  - macro (CPI, jobs, Fed_decision)
- `odds_bucket`
  - longshot (<20%)
  - mid (20-60%)
  - favorite (>60%)
- `bet_regime` (percentile-based per wallet)
  - small (0-50%)
  - normal (50-90%)
  - large (90-100%)
  - Minimum 20 bets for reliable classification
- `timing`
  - early (>7 days to resolution)
  - late (<7 days)

### Seer Scopes (Market Archetypes)

- `market_type`: US_election, sports_major, macro_data, crypto_event
- `insider_likelihood`: high, medium, low
- `volatility`: stable (slow-moving), volatile (rapid price swings)

---

## v0 Decisions (Locked In)

1. **Skill score**
   - `bayes_win_rate = (wins + 1) / (trades + 2)`
   - `skill_score = logit(bayes_win_rate)`
   - Handles small samples gracefully, compresses extremes, preserves ordering
   - Standard in quant factor models
2. **Bankroll = on-chain reality**
   - `bankroll = USDC_balance + Σ position_values`
   - No estimation — directly readable on Polygon/Base
   - Auto-captures deposits / withdrawals
   - Standard Polymarket-specific bankroll
3. **Scope matching**
   - Exact match `(cat, odds, regime, timing)` if `n ≥ 30`
   - Fall back to `(cat, odds)` if `< 30`
   - Fall back to `(cat)` if `< 30`
   - Global wallet score with `0.3x` discount if `< 30`
4. **Market filtering (v0 universe)**
   - Categories: US politics, NFL / NBA / World Cup, CPI / jobs data
   - Resolution: binary outcomes, verifiable on-chain
   - Liquidity: minimum $50k volume
   - Insider potential: focus on predictable information asymmetry
5. **Confidence → size**
   - If confidence < 0.05: skip
   - Else: `size = base * (confidence / 0.20)`
   - Capped: `min(max, base * 3)`
   - Floored: `max(min, base * 0.3)`
6. **Initial wallet seeding**
   - Bootstrap: historical data mining (`win_rate > 60%` AND `n_bets > 50` AND `total_volume > $100k`)
   - Live: continuous discovery via 6 promotion criteria

---

## What We’re NOT Doing (Philosophy)

**Wallets**

- ❌ Don’t track “bad wallets” as smart money (they’re in data for contrast, zero weight)
- ❌ Don’t decay wallet skill arbitrarily — only Bayesian update on new evidence
- ❌ Don’t try to “improve” wallet strategies — just classify where they work

**Seer**

- ❌ No complex PM / DM tuning like `Lotus_Trader`
- ❌ No action overrides
- ❌ No counterfactual optimization
- ✅ Simple: “Does smart-money weighting predict this market type? Yes / No”

**Scope**

- ❌ Not global “good trader” vs “bad trader”
- ✅ Context-specific: “This wallet in THIS situation”

---

## Future Enhancements (v3+)

- **Hedging detection**
  - Cross-market correlation to identify basis traders
  - Down-weight hedge positions vs directional conviction
  - v0: treat all bets as directional; accept hedge noise
- **Multi-dimensional scope expansion**
  - Add `liquidity_regime`, `time_to_resolution_hours`
  - Cluster analysis for automatic scope discovery
- **Kelly sizing**
  - Replace linear confidence → size with Kelly fraction
  - Risk-adjusted position sizing

---

## Philosophy · Renaissance Summary

We compute the internal probability implied by wallets with proven forecasting ability in similar contexts, and trade the deviation from public probability.

Scope is the backbone. Context is everything. Clean. Minimal. Robust.
