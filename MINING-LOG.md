# TN10 mining log (desk Build + sandbox bot)

tKAS only. Target changes are written here first. Wallet addresses are not stored in this repo.

Testnet-10. Zero value. Best practice: **sync, then mine one address**. Unsynced submits do not land on the live DAG.

**Two nodes.** Desk `[ip]` is Windows Grok Build. Sandbox `[ip]` is tn10 bot. Label the row.

| When (UTC) | Action | Target | Notes |
| --- | --- | --- | --- |
| 2026-09-18 12:55:21 | START 24 threads `--mine-when-not-synced` | previous desk target | **Grok Build desk.** 17.79 MH/s. `route is full`. 10 local accepts; public REST 404. Still 0. |
| 2026-09-18 12:56:14 | STOP miner | same | **Grok Build desk.** Let header IBD run. |
| 2026-09-18 13:02 | NEXT (queued) | first empty operator address | **Grok Build desk.** |
| 2026-09-18 14:25 | HOLD | same queued | **Grok Build desk.** kaspad pid 88436 **header IBD ~85%** (last header 18 Sep 10:28 +02). 0 bodies. Public mempool 10,933. Network ~11.4 MH/s. |
| 2026-09-20 15:17–15:19 | STOP all miners + keepalive (Grok box farm) | idx28 | **tn10 bot sandbox.** Stress test ended. See REVIEW. |
| 2026-09-20 15:32 | WIPE TN10 datadir + restart kaspad | none (miners 0) | **tn10 bot sandbox.** User ordered wipe. Data erased. Fresh IBD. Mainnet untouched. Next: tip-follow then 10 miners @ idx29. See RECOVERY. |
| 2026-09-20 ~18:56 | Build recheck (Windows desk) | desk empty mine-to (not stored) | **Grok Build desk**, not sandbox. kaspad **tip-following** (relay+submit, ~9–11 BPS, ~48 u-tps). Miner 8 threads, grokstress, no `--mine-when-not-synced`. Sandbox recovery **unverified** from this PC. See BUILD-REVIEW-2026-09-20.md. |
