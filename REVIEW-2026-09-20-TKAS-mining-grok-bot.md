# TKAS mining Grok bot review

**Writer: tn10 bot (Grok Bot Linux sandbox), 20 Sep 2026.**  
**Not Grok Build. Not kaspa bot. Not Kaspa core.**

Independent Grok Build pushback (same GitHub): [BUILD-REVIEW-2026-09-20.md](BUILD-REVIEW-2026-09-20.md).

**Date:** 2026-09-20 (Europe/Brussels)  
**Repo journal:** STP-KAS/tn10-storm-2026-09-18-mining-public (private)  
**Scope:** **Sandbox** TN10 stress farm (100–150×1-thread) vs Kaspa testnet-10. Not the Windows desk miner. tKAS has no value.  
**Status at review:** Sandbox mining **STOPPED**. TN10 kaspad and mainnet kaspad left running on the sandbox. Then user ordered a wipe — see RECOVERY.

Wallet addresses are **not** stored in this file (indices only).

**Correction (same day):** local submit accepts are **~714,409** total (3,122 singular `Accepted block … via submit` + **711,287** from batch `Accepted N blocks … via submit` lines). First draft only counted singular lines.

---

## Executive summary

We found a huge number of blocks because **TN10 network hashpower is tiny** relative to a ~100–150× one-thread CPU farm. Finding work is easy.

Most of that work never became live DAG coinbase, because:

1. **Submit path saturation** — miners reported ~95–99% `route is full` in the last window (found ≈28k, route_full ≈28k, submit_ok ≈250). Lifetime miner logs: Found ≈894k, route_full ≈811k, submit_ok ≈58k, “not synced” fails ≈21k.
2. **Local node overload / tip lag** — kaspad often logged `Processed 0 blocks and N headers` while still accepting submits (often in batches of ~100); ~90 peer flows disconnected on 120s timeouts (load, not an explicit ban).
3. **Address rotation** — 29 of 46 list slots used (idx 0–28). Public REST shows **~171.7k tKAS** across **10** of those 29 indices; the other 19 (including the last ones, 27–28) sit at **0**.

**Were we cut out?** **Partial.** Not a peer ban (0 `banned` / misbehavior hits). We *were* effectively sidelined for useful (blue/paid) mining late by **self-congestion + tip-chase + route full**. Earlier tip-following windows still funded explorer balances.

Upstream context: [rusty-kaspa#1134](https://github.com/kaspanet/rusty-kaspa/issues/1134) (SubmitBlock backpressure / route full).

---

## Setup

| Piece | What ran |
| --- | --- |
| Node | `kaspad` TN10 (`--testnet --netsuffix=10 --utxoindex`), listen `[ip]`, RPC `[ip]`, `--ram-scale=0.15`, `--rpcmaxclients=400`, `externalip` via bore |
| Public reach | `[tunnel]` → local 16211 |
| VPN | ProtonVPN ON near stop (egress ~`[ip]`) |
| Farm | up to ~150 `kaspa-miner` processes, **`-t 1` each** (~8–10 MH/s class), keepalive + red-watch / rotate |
| Goal / hold | goal 150; late hold 100 under `HOLD100_RF_HIGH` |
| Mainnet | untouched (still archival on 16111) |

Stop artifacts: `/tmp/kaspa-tn10-miners/STOP_MINING`, `MINING_SUSPENDED`, `MINING_STOPPED`, `hold-miner-count=0`.

---

## What happened (highlights)

- **17 Sep+:** TN10 node logging accepts via submit from early stress (first singular accepts ~17:30 UTC 17 Sep).
- **18 Sep desk journal** (`REPORT.md`): single 24-thread miner already ~17.8 MH/s vs network ~12 MH/s; immediate `route is full` during IBD — same failure mode, smaller scale.
- **Through 20 Sep:** farm scaled to ~100–150 one-thread miners; rotated idx 0→28; red-watch forced explorer red/blue checks; RF repeatedly high → hold 100.
- **20 Sep ~05:08Z:** peer ping/relay 120s timeouts; header catch-up bursts.
- **Afternoon:** sustained `Processed 0 blocks and N headers/10s`; HOLD100_RF_HIGH.
- **20 Sep ~17:17 CEST:** last quiet metrics — RF≈0.9924, alive≈104/100, explorer reds=blues=0 on current idx.
- **20 Sep ~17:17–17:19 CEST:** user ended stress test; miners + keepalive killed; nodes kept.

---

## Why so many blocks found?

**Yes — lack of competing hashpower on TN10, plus a large local farm.**

Public REST at review time:

| Metric | Value |
| --- | --- |
| Network hashrate | ~**25.6 MH/s** (`hashrate` 2.56e-5 TH/s) |
| Difficulty | ~1.28e6 |
| tipHashes | **871** (wide DAG; Sep 18 journal had ~3.6k tips) |
| blockCount ≈ headerCount | ~1.72M |

A single synced CPU miner at ~18 MH/s was already a large fraction of the whole testnet. Scaling to ~100–150 threads of `kaspa-miner` makes `Found a block` spam the expected outcome. **High find count is not proof of high DAG acceptance.**

---

## Submit path bottleneck (route full)

Lifetime miner-*.log totals (deeper pass):

| Signal | Count |
| --- | --- |
| Found a block | 893,909 |
| route is full | 811,149 |
| Failed submitting (all) | 831,972 |
| node is not synced | 20,823 |
| Block submitted successfully | 57,758 |

Last keepalive metrics window:

| Signal | Count |
| --- | --- |
| found_5m | 28,332 |
| route_full_5m | 28,117 |
| submit_ok_recent | 251 |
| RF | **0.9924** |

Local kaspad log (`/tmp/kaspa-logs-tn10/rusty-kaspa.log`):

| Signal | Count |
| --- | --- |
| Singular `Accepted block … via submit` | 3,122 |
| Batch `Accepted N blocks … via submit` (sum of N) | **711,287** (27,408 events) |
| **Total local submit accepts** | **~714,409** |
| Accepted … via **relay** | 82 |
| `banned` / misbehaving | **0** |
| Peer disconnect `timeout expired after 120s` | **90–92** |

Interpretation: the node accepted a large volume of submits over the campaign (often batched), but late windows were dominated by discarded finds. Peer timeouts match overload, not a ban list. Many local accepts still do not imply paid explorer blues on the *current* address while tip-chasing.

---

## Were we cut out?

| Hypothesis | Evidence | Verdict |
| --- | --- | --- |
| Explicit peer ban | 0 ban/misbehavior lines | **No** |
| Network operators blacklisting us | Still connected to public peers; bore + VPN; accepts continued until stop | **Unlikely** |
| VPN / egress blocking REST | Intermittent DNS timeouts; later API 200 OK; balances readable | **Transient tool pain, not a ban** |
| Node tip lag / IBD-like header chase | Long stretches of `0 blocks / N headers`; large mergesets; after miner kill, bursts of body processing | **Yes — operational cut-out** |
| RPC submit starvation (`route is full`) | 95–99% of finds discarded in late window; lifetime RF ~0.91 of finds | **Yes — self cut-out** |
| Explorer “0 reds / 0 blues” on current addr | Matches idx 27–28 REST balance **0 / 0 txs**; earlier idxs still funded | **Looked like cut-out on the watched address; farm-wide not zero** |

**Bottom line (one-liner):** Partial — no ban; peers up; tip-chase + `route is full` sidelined useful mining on idx28 despite ~714k lifetime local submit accepts and ~172k tKAS on earlier tip-following addresses.

---

## Hashpower vs network difficulty

- TN10 difficulty tracks a small CPU testnet (~10–30 MH/s class during this campaign).
- Farm hashrate ≫ typical network hashrate ⇒ enormous local find rate.
- Without a **synced tip** and a **non-full submit route**, finds do not become selected-parent coinbase on the public DAG.
- Scaling process count past what kaspad can accept **hurts** more than it helps (more RF, more peer timeouts, more tip lag).

---

## Outcomes (layers)

Distinguish four layers:

1. **Found locally** — ~894k in retained miner logs; tens of thousands per few minutes in the late window.
2. **Accepted by our kaspad via submit** — **~714,409** (singular + batch).
3. **Accepted on the public DAG / explorer** — uneven by rotation index (see table).
4. **Visible on the address you were watching at stop (idx 28)** — **0** balance / **0** txs.

### Public REST balances for used indices 0–28 (api-tn10.kaspa.org)

| Idx | Balance (tKAS) | Tx count | Notes |
| --- | --- | --- | --- |
| 0 | 435.23 | 37 | early |
| 1 | 373.50 | 37 | early |
| 2 | 17,227.58 | 40 | strong early |
| 3–4 | 0 | 0 | empty |
| 5 | 8,524.57 | 633 | |
| 6–13 | 0 | 0 | empty stretch |
| 14 | 12,796.97 | 1,389 | |
| 15 | **88,121.58** | **17,705** | best slot in this pass |
| 16 | 0 | 0 | |
| 17 | 6,214.04 | 918 | |
| 18 | 7,004.51 | 1,025 | |
| 19 | 0 | 0 | |
| 20 | 999.00 | 0 | balance without tx count — verify manually |
| 21–23 | 0 | 0 | |
| 24 | 30,000.00 | 25 | |
| 25–28 | 0 | 0 | **late / stop address empty** |
| **Sum (nonzero)** | **~171,697 tKAS** | — | **10 / 29** slots funded |

tKAS is worthless; numbers are for stress analysis only.

---

## Recommendations

1. **Keep mining stopped** until a deliberate resume (already done).
2. On resume: **sync first** (tip-follow / relay accepts), mine **one** process (or few threads) to **one** address; raise threads only while submits stay `Block submitted successfully`.
3. Cap farm size to what RPC can accept; treat RF ≥ 0.5 as “too many miners,” not “need more hashrate.”
4. Do not use explorer red/blue on a single rotating address as the sole health signal — sweep all used indices (as above).
5. Leave mainnet alone (already policy).
6. Optional: consolidate funded indices into the operator/sweep address once mining is intentionally resumed and tip is healthy.
7. Track upstream SubmitBlock backpressure: rusty-kaspa#1134.

---

## Raw numbers (quick table)

| Item | Value |
| --- | --- |
| Miners at stop | 0 |
| Keepalive | dead |
| TN10 / mainnet nodes | up / up |
| Rotation index at stop | 28 (29 used of 46) |
| Local submit accepts (singular + batch) | **~714,409** |
| Local relay accepts | 82 |
| Lifetime miner Found / RF / submit_ok | ~894k / ~811k / ~58k |
| Ban hits | 0 |
| Peer timeout disconnects | ~90 |
| Late RF | ~0.99 |
| Network HR (review) | ~25.6 MH/s |
| Funded rotation slots | 10 / 29 |
| Sum tKAS (those slots) | ~171.7k |

---

## Sources

- `/tmp/kaspa-logs-tn10/rusty-kaspa.log`
- `/tmp/kaspa-tn10-miners/` (miner logs, hold/stop flags, rotation-state)
- `[local path]`
- `https://api-tn10.kaspa.org/info/hashrate`, `/info/blockdag`, `/addresses/{addr}/balance`
- Prior journal: `REPORT.md` / `MINING-LOG.md` in this repo (18 Sep desk pass)
- https://github.com/kaspanet/rusty-kaspa/issues/1134
