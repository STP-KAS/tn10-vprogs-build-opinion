# Public reads behind the opinion

26 Sep 2026. Testnet-10 only. No wallet address is copied here.

Times: `acceptedTxBlockTime` 1790365505207 ms is 2026-09-25 19:45:05.207 UTC, which is 21:45:05.207 CEST. The blockDAG `pastMedianTime` on this read was 1790429486982 ms, 63,981,775 ms later, about 15:31 CEST on 26 Sep 2026. Past-median time lags wall clock by a little.

## api-tn10.kaspa.org

`GET /info/health`

- `serverVersion`: 2.0.1
- `isSynced`: true
- kaspad `blueScore`: 568825293
- database `blueScore`: 568825255
- `blueScoreDiff`: 25
- `acceptedTxBlockTime`: 1790365505207

`GET /info/virtual-chain-blue-score`

- `blueScore`: 569462361
- ahead of the health document's kaspad score by 637068 (about 17 h 42 min at 10 blue per second)
- ahead of the stress note's indexer cutoff 568828507 by 633854

`GET /info/blockdag`

- `networkName`: kaspa-testnet-10
- `virtualDaaScore`: 580980207
- `pastMedianTime`: 1790429486982

## vprogs-tt.izio.fr

`GET /api/state`

- `l2_tip`: 873933
- `settled.daa_score`: 580940363
- `settled.txid`: `9bcba5323515fcbece8a5d4f02ba1dfabe521077ccc9feda899e6e19658ba852`
- gap to virtual DAA 580980207: 39844

Movement since the stress note's evening sample (settled DAA 580229488, `l2_tip` 566858):

- settled DAA +710875
- L2 tip +307075

The desk public snapshot's settled DAA was 580137056. That is a third value, earlier than the evening sample.

## kaspanet/vprogs pull 165

- state: open, draft, unmerged
- author: biryukovmaxim
- created: 2026-09-26T11:22:48Z
- base: `fix/reorg-boundary-duplicate-bundles`
- head: `081af9b9de0689b61e1a456cf296a4ad9e562b73`
- commits: `36a6d38cc476355844d8a9ddd7fbfb442fab498f`, `bf3509cf1018ed357bab5ed5979ac55ca39a46e8`, `494aacc6d140c0e556597293a9f60b0b9eba6455`, `081af9b9de0689b61e1a456cf296a4ad9e562b73`
- files include `l1/wallet/src/build/carrier.rs`, `l1/wallet/src/build/payout.rs`, `l1/wallet/src/lib.rs`, `zk/backend/risc0/settler/src/worker.rs`, `runner/src/exit_index.rs`, `runner/tests/two_provers_contend.rs`

## Pins at this read

| Ref | SHA | Commit time |
|---|---|---|
| kaspanet/vprogs `release-candidate` | `081af9b9de0689b61e1a456cf296a4ad9e562b73` | 2026-09-26T11:21:41Z |
| kaspanet/vprogs `master` | `f9b84a863a7c7c20586a9cf947550475e894f72e` | 2026-07-28T11:24:41Z |
| biryukovmaxim/vprog-tictactoe `f128efd` | `f128efd65f80fa396c2df8aced5d17c8420c40de` | 2026-09-26T11:40:16Z |
| biryukovmaxim/vprog-tictactoe `master` | `803a120ba1449c1ecbadef46dd68586b8ce803f6` | 2026-09-26T11:50:10Z |

The stress runs name vprogs `3a61c0b` as the code they executed. `release-candidate` has since moved to the pull-request head above.

## Processed-transaction counter

On kaspanet/rusty-kaspa v2.1.0, `consensus/src/pipeline/body_processor/processor.rs` does `txs_counts.fetch_add(block.transactions.len())` for each processed block body. `consensus/src/pipeline/monitor.rs` prints that delta in the 10-second "Processed N blocks…" line. Grok Bot's `scripts/nettps.sh` divides the transaction count in that line by the window length and calls the result network TPS.

Default mempool transaction count in `mining/src/mempool/config.rs` is `1_000_000`. `apply_ram_scale` multiplies that count and the byte limit by `ram_scale`, and only scales down. Their crash log is the scaled cap of 100,000.
