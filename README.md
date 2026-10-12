> **Experimental. We are just trying this.**
>
> We test, we write down what we think we saw, and that is the whole product. A number here is not the truth. A chart is not the truth. Any other sentence that sounds sure of itself is not the truth either. Do not count any of it as a claim.
>
> [Disclaimer](DISCLAIMER.md)
>
> **Part of [tn-master](https://github.com/STP-KAS/tn-master)**, the map of every TN10 storm. Renamed on 11 Oct 2026 from `tn10-grok`; old links redirect here.
>
> ← first storm | next storm: [tn10-storm-2026-09-25](https://github.com/STP-KAS/tn10-storm-2026-09-25) →
>
> **Public clean copy** of `STP-KAS/tn10-storm-2026-09-18-mining` (private, archived), made on 11 Oct 2026 from its current files only. IPs, tunnel hosts, wallet addresses, 64-hex hashes and local paths are replaced by `[ip]`, `[tunnel]`, `[address]`, `[hash]` and `[local path]`. The full history stays private.

> **Experimental only. Not a product.**
>
> Do not use wallet integrations on this GitHub. [DISCLAIMER.md](DISCLAIMER.md)

# tn10 grok

Private TN10 stress journal. **tKAS is worthless.** Not Kaspa core. Not a product. Not mainnet.

Wallet addresses are not stored in this repo.

Live: [tn10.kaspa.stream](https://tn10.kaspa.stream/) · REST: [api-tn10.kaspa.org](https://api-tn10.kaspa.org/)

Public sister catalog (Grok Build, 14 Sep freeze): [STP-KAS/tn10-hard-test](https://github.com/STP-KAS/tn10-hard-test).

---

## Bot vs Build (do not flatten)

| Voice | Machine | `[ip]` means | Files in this repo |
| --- | --- | --- | --- |
| **Grok Build** | Windows desk | **this PC** when desk kaspad is up | [REPORT.md](REPORT.md) (18 Sep), this README, [BUILD-REVIEW-2026-09-20.md](BUILD-REVIEW-2026-09-20.md), desk rows in [MINING-LOG.md](MINING-LOG.md) |
| **tn10 bot** | Grok Bot Linux sandbox | **sandbox** loopback, not this PC | [REVIEW-2026-09-20-TKAS-mining-grok-bot.md](REVIEW-2026-09-20-TKAS-mining-grok-bot.md), [RECOVERY-2026-09-20-wipe-light-mine.md](RECOVERY-2026-09-20-wipe-light-mine.md), farm/wipe rows in the log |
| **kaspa bot** | Grok Bot Linux sandbox | never TN10 | not this journal |

Grok Build pushback (20 Sep): **[BUILD-REVIEW-2026-09-20.md](BUILD-REVIEW-2026-09-20.md)**.

---

## State at last write (20 Sep 2026 evening)

| Site | Last journaled state | Who |
| --- | --- | --- |
| **Desk (Windows)** | kaspad **tip-following** (relay + submit, bodies). Miner **8 threads**, `grokstress`, no `--mine-when-not-synced`. Empty operator mine-to (not stored). | Grok Build |
| **Sandbox (tn10 bot)** | TN10 datadir **wiped** ~15:32 UTC. kaspad in fresh IBD. Miners **0**. Planned: tip-follow, then 10 miners @ rotation idx 29. **Unverified from this Windows session.** | tn10 bot |
| **Public REST** | DAA ~575.98M · ~21.1 MH/s · mempool **3** · tips ~2,039 · locked pay-from ~4.07M tKAS / 1.18M txs (sweep idle) | Grok Build REST |

18 Sep IBD pause (pid 88436, miner killed after 10 stale local accepts / REST 404) is **history**. Do not quote it as live.

---

## 18 Sep desk experiment (history)

| When (UTC) | Who | Target | Result |
| --- | --- | --- | --- |
| 12:55:21–12:56:14 | Grok Build desk 24-thread miner | previous desk target | 17.79 MH/s. `route is full`. 10 local accepts; **public REST 404**. Still **0**. |
| then | Desk miner | **paused** | kaspad pid 88436 header IBD. |

That failure mode is the same family as the later sandbox farm (`Found a block` ≠ selected-parent). Details: [REPORT.md](REPORT.md).

Mining log: [MINING-LOG.md](./MINING-LOG.md)

---

> **Standard disclaimer.** This GitHub, not the topic above.
>
> Si vis pacem, para bellum.
>
> Intern at https://sixpack.wtf/  
> X: https://x.com/StppStp · GitHub: https://github.com/STP-KAS
