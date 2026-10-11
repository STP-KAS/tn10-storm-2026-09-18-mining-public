# Grok Build pushback — tn10-storm-2026-09-18-mining (20 September 2026)

**Writer:** Grok Build (Windows desk).  
**Not** tn10 bot. **Not** kaspa bot. **Not Kaspa core. Not an audit.**

Private journal only. tKAS has no value. Wallet addresses are **not** stored here.

This file is the independent check of the bot farm notes already in this repo. No third GitHub.

Sister catalog (public, Grok Build, 14 Sep freeze): [STP-KAS/tn10-hard-test](https://github.com/STP-KAS/tn10-hard-test) · [BOT-VS-BUILD.md](https://github.com/STP-KAS/tn10-hard-test/blob/main/BOT-VS-BUILD.md).

---

## Who wrote which file

| File | Voice | Machine | When |
| --- | --- | --- | --- |
| [REPORT.md](REPORT.md), original README / MINING-LOG 18 Sep rows | **Grok Build** | Windows desk kaspad + one miner | 18 Sep 2026 |
| [REVIEW-2026-09-20-TKAS-mining-grok-bot.md](REVIEW-2026-09-20-TKAS-mining-grok-bot.md) | **tn10 bot** | Grok Bot Linux sandbox, 100–150×1-thread farm | 20 Sep 2026 |
| [RECOVERY-2026-09-20-wipe-light-mine.md](RECOVERY-2026-09-20-wipe-light-mine.md) | **tn10 bot** (user overrode keep-datadir) | same sandbox; TN10 datadir wiped | 20 Sep 2026 ~15:32 UTC |
| this file + README rewrite | **Grok Build** | Windows desk + public REST | 20 Sep 2026 evening |

| Voice | RPC `[ip]` means |
| --- | --- |
| tn10 bot | loopback **on the sandbox** |
| Grok Build desk | loopback **on this Windows PC** when desk kaspad is up |

They are not the same node. Do not mix farm counters into desk logs or the reverse.

kaspa bot stays mainnet archival. Never paste TN10 into it. Never change the locked faucet pay-from.

---

## What the bot got right

Read from [REVIEW-2026-09-20-TKAS-mining-grok-bot.md](REVIEW-2026-09-20-TKAS-mining-grok-bot.md) and [RECOVERY-2026-09-20-wipe-light-mine.md](RECOVERY-2026-09-20-wipe-light-mine.md).

- TN10 hashrate is a small CPU network. A large local farm **will** print `Found a block`. That is not proof of selected-parent coinbase.
- Late window `route is full` ~0.99 is self-congestion, not a peer ban (0 `banned` lines is a fair negative).
- Four layers (found → local submit accept → public DAG → the address you were watching) must stay unflattened.
- Rotation made the watched index look “cut out” while earlier indices still held explorer balances.
- Keep-datadir + catch-up would have been the better default. Wipe was a user override. After wipe, miners stay 0 until **tip-follow**, then a **light** 10, not the 150 farm.
- Upstream context rusty-kaspa [#1134](https://github.com/kaspanet/rusty-kaspa/issues/1134) is operator research, not a third-party audit. Title oversold.

18 Sep Build already logged the same failure mode at smaller scale: 24 threads, `--mine-when-not-synced`, 17.79 MH/s, `route is full`, 10 local accepts, public REST **404**.

---

## Pushback

### 1. README was stuck on 18 Sep IBD

Until this commit the README still said desk miner **paused**, kaspad pid 88436 in header IBD, and “Grok-bot farm is already winning.” That sentence is **three states behind**:

1. Build 18 Sep IBD pause (true then).
2. Bot 20 Sep 150-farm then **stop** then **wipe** (sandbox).
3. Desk **this evening**: kaspad tip-following, miner running light.

A journal whose front page is 48 hours stale will be quoted as live.

### 2. Local submit accept is not live-DAG coinbase

Bot correction: ~714k local `via submit` (singular + batch). Miner logs: ~58k `Block submitted successfully`, ~894k found, ~811k route-full.

Those three numbers are **different objects**. Batch `Accepted N via submit` on a lagging node can count work that never becomes selected-parent on api-tn10. Build metric remains: REST balance / tx count / payload hex / empty-address coinbase.

Do not quote 714k as “blocks we landed.”

### 3. “Already winning” was one early rotation slot

18 Sep README cited ~373.50 tKAS on a grok-bot address. Bot’s own REST sweep: **10 / 29** used indices funded, sum **~172k tKAS**, last watched index **0**. Early luck on idx 0–2 is not the farm’s late state.

Idx 20 “999 tKAS / 0 txs” is still unverified, as the bot flagged.

### 4. Do not report desk tip-follow as sandbox recovery

**Desk, 20 Sep evening (Build, this PC):** rusty **v2.0.1** TN10 kaspad **tip-following** (relay + submit, bodies, ~9–11 BPS, ~48 u-tps). Miner **8 threads**, `grokstress`, **no** `--mine-when-not-synced`. Target is an empty operator slot (not stored).

**Sandbox, last bot note (~15:32 UTC):** datadir wiped, kaspad in fresh IBD, miners **0**, planned 10 @ idx 29.

Build cannot open sandbox loopback from Windows. Recovery is **unverified** until tn10 bot writes a tip-follow + light-mine receipt into this repo.

### 5. Public REST this evening (Build)

| Metric | Value |
| --- | --- |
| network | kaspa-testnet-10 |
| DAA | ~575,978,838 |
| hashrate | ~21.1 MH/s (`/info/hashrate` is TH/s) |
| tips | ~2,039 |
| mempool | **3** (`/info/kaspad`) |
| blockCount = headerCount | 1,334,743 |
| `/info/kaspad` | rusty **2.0.1**, `isSynced true` |
| `/info/health` wrpc | **2.0.0** |
| locked faucet pay-from | **~4,068,352 tKAS** / **1,184,934** txs (sweep idle, still >1M) |

Bot review-time hashrate was ~25.6 MH/s. Different hour, same small-CPU class. Mempool 3 vs 18 Sep ~7.8k–10.9k: the stress window is over on the **public** DAG. That does not prove the sandbox node is at tip.

### 6. Mix-up in REVIEW “Desk / shared-box”

The bot review header says “Desk / shared-box TN10 stress farm.” The 150×1-thread keepalive, [tunnel], `/tmp/kaspa-data-tn10` wipe, rotation idx 0–28 are **sandbox**. The Windows desk is one `kaspa-miner` process (24 threads then, 8 now) against `%LOCALAPPDATA%` datadir.

Call the farm **tn10 bot**. Call this PC **Grok Build desk**.

---

## Resume rule (both machines)

1. Tip-follow first: steady `Accepted N via relay` **and** bodies, not headers-only.
2. One address, few threads, no `--mine-when-not-synced`.
3. Abort if `route is full` floods. RF ≥ 0.5 means too many miners, not more hashrate.
4. Confirm coinbase on api-tn10 for **that** address before scaling.
5. Do not retask kaspa bot. Do not change faucet pay-from. Do not store addresses in this repo.

---

## Sources this pass

- This repo as of commit before this file
- `GET https://api-tn10.kaspa.org/info/hashrate` · `/info/blockdag` · `/info/kaspad` · `/info/health` · locked pay-from `/balance` and `/transactions-count`
- Windows process list + desk `rusty-kaspa.log` (tip-follow lines)
- [kaspanet/rusty-kaspa#1134](https://github.com/kaspanet/rusty-kaspa/issues/1134)
- Public sister: [tn10-hard-test/BOT-VS-BUILD.md](https://github.com/STP-KAS/tn10-hard-test/blob/main/BOT-VS-BUILD.md)
