# TN10 grok — 18 Sep 2026 stress journal

**Writer: Grok Build (Windows desk), 18 Sep 2026. Not tn10 bot. Not kaspa bot.**

Private. tKAS has no value. Not Kaspa core. 20 Sep Grok Build pushback: [BUILD-REVIEW-2026-09-20.md](BUILD-REVIEW-2026-09-20.md). Sandbox farm notes are the bot REVIEW / RECOVERY files, not this one.

Operator asked to analyse testnet-10, use tn10.kaspa.stream plus other sources, reboot the desk tKAS node and miner if needed, and mine to a designated operator address (not stored in this repo).

Grok bot is already mining tKAS and looks unsuccessful. This pass is the desk Windows site plus public REST.

Wallet addresses are not stored in this repo.

## Sources that actually answered

| Source | What we got |
| --- | --- |
| [tn10.kaspa.stream](https://tn10.kaspa.stream/) | Live SPA explorer. HTML fetch is a shell; numbers come from REST. |
| [api-tn10.kaspa.org](https://api-tn10.kaspa.org/) | BlockDAG, hashrate history (`hashrate_kh`), health, kaspad info, address balances, sink block |
| explorer-tn10.kaspa.org | **Paused** (`Deployment Paused` / 402) |
| Desk rusty-kaspa.log | IBD + last healthy run 17 Sep 17:18 Europe/Brussels |
| Desk miner v0.2.7 | 17.79 MH/s then `route is full` |
| [STP-KAS/groks-wallet](https://github.com/STP-KAS/groks-wallet) | Locked faucet pay-from, 150-miner farm on **random** addresses |

kaspa.stream resources page points REST at api.kaspa.org / api-tn10.kaspa.org. That is the stream backend for this pass.

## Network snapshot (~12:50 UTC)

- `networkName`: kaspa-testnet-10
- rusty-kaspa on the public indexer: **2.0.1**, synced, utxoindex on
- `virtualDaaScore`: **573,864,255**
- `blockCount` / `headerCount`: 1,268,158
- `difficulty`: 602,067 → ~**12.04 MH/s** (`difficulty * 20` at 10 BPS). REST `/info/hashrate` returns `1.20e-5` which is the same rate in TH/s. History field `hashrate_kh` is the readable unit.
- Hashrate history: now 12.3 MH/s; last 48 points mean 15.7 MH/s (min 9.9, max 20.8); last 7d mean 20.4 MH/s (max 31.7). All-time max in that series 366 MH/s (31 May 2026, `hashrate` max endpoint 7.27e-5 TH/s = 72.7 MH/s on that particular header).
- **tipHashes: 3,612**. Virtual parents: 4. This is a stressed DAG, not a quiet 10-parent chain.
- Mempool: **7,804** (later fee buckets sat at 100 sompi/gram; an earlier sample had priority 226).
- Last desk-at-tip log (17 Sep 17:18): 7–9 blocks/s accepted via relay, 50–80 u-tps, mergeset ~7–11. Today’s header IBD shows **16 parents** and mergesets **20–128**. Wider than yesterday.
- Coin supply: 26.95B tKAS circulating / 29B max. Block reward **2.060 tKAS**. Next reduction 30 Sep 2026 00:55 UTC to 1.945 tKAS.
- Health blueScore ~562,686,000 vs DAA 573,864,000 — expected DAA vs blue gap, indexer `isSynced: true`.

Sink coinbase sampled from REST (competitor miners, not stored here):

- minerInfo `2.0.1/0.2.6`
- Nearby: `2.0.0/0.2.6`
- Nearby: `2.0.1/0.2.6`, 51-tx block

Competition is other CPU `kaspa-miner` 0.2.6 processes, not ASICs.

## Addresses

Wallet addresses are not stored in this repo. Roles only:

| Role | REST |
| --- | --- |
| This pass (mine-to) | 0 sompi, 0 txs |
| groks-wallet faucet pay-from (locked, do not change) | **4,127,126.87 tKAS**, 1,184,744 txs |

Faucet already has >1M tKAS, so the grok-bot farm rule (sweep random miners into pay-from when under 1M) does **not** fire. Random-address mining will not show up on the locked address.

No seed in this repo.

## Desk stack this pass

Found **down** at session start. Last healthy TN10 kaspad: 17 Sep 17:18 (force-stopped later). Last miner log: 15 Sep, then `h2 protocol error` / connection reset. Scheduled tasks `KaspaPublicNode` and `GrokCyberstpWatch` still Disabled. Mainnet 16111 left alone.

Restarted:

```
kaspad 2.0.1 --testnet --netsuffix=10 --utxoindex
  --listen [ip]
  --rpclisten [ip]
  --rpclisten-borsh [ip]
  --rpclisten-json [ip]
  --outpeers 16 --maxinpeers 64 --ram-scale 0.7 --rpcmaxclients 32
  --enable-unsynced-mining --externalip <desk-public-ip>:16211
```

CPU: 13th Gen i7-13700K (8P+8E / 24 threads), 32 GB RAM. Datadir ~59 GB under `%LOCALAPPDATA%\rusty-kaspa\kaspa-testnet-10`.

`--externalip` is required: without it kaspad advertised a Hamachi/Radmin `26.x` address as public.

IBD: connected 12–14 outbound peers (protocol 10), inbound from two public peers. Header catchup ~150–1000 headers / 10s, **0 bodies** yet. One outbound peer has a history of `pruning points are violating finality` in the desk warn log.

## Miner experiment (then pause)

Started:

```
kaspa-miner v0.2.7 --testnet -t 24 --mine-when-not-synced
  --mining-address <not stored in this repo>
  -s [ip] -p 16210 --user-agent-suffix grokstress
```

Result in ~30s:

- Hashrate **17.79 Mhash/s** (above live network 12.3 MH/s)
- Hundreds of `Found a block`
- Almost every submit: `RpcError { message: "Block was not submitted: route is full" }`
- kaspad still at `Processed 0 blocks and N headers`

Miner was **killed** so IBD can use the CPU and the submit channel. Mining on a stale IBD template cannot land coinbase on the real DAG.

This is the same failure mode as “grok bot is mining but unsuccessful”:

1. Difficulty is so low that a CPU farm *finds* blocks constantly.
2. `Found a block` ≠ selected-parent coinbase. Submit must succeed on a **synced** node.
3. During IBD (or with 150 one-thread miners) kaspad’s submit route fills. Work is thrown away.
4. Farm policy mines **random** wallet addresses, so the address you watch stays at 0 even when some blocks do land.
5. Desk node was simply **off** since 17 Sep. Nothing to submit to.

## Plan to beat it

1. Let desk kaspad finish header IBD, then body IBD, until `Accepted N blocks via relay` (tip following). Watcher is running against pid 88436.
2. Restart miner to the target address, **no** `--mine-when-not-synced`. Start at 16 threads; raise to 24 if submits stay `Block submitted successfully`.
3. If `route is full` returns after sync, drop threads (do not add a 150-process farm on this box).
4. Confirm coinbase on REST + tn10.kaspa.stream for the target address.
5. Do **not** retask kaspa bot (mainnet). Do **not** change groks-wallet faucet pay-from. Optional later: tell tn10 bot to park hashrate on this same address for the stress, then restore random+sweep.

## Not done yet

- Desk not at tip.
- Target address still 0.
- No coinbase txid to journal.
- Grok bot farm not retasked from this Windows session (sandbox loopback is not this PC).
