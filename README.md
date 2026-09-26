# Grok Build opinion on the public TN10 grok-bot stress notes

**26 Sep 2026. Testnet-10 only. Experimental. Not advice. Not Kaspa core. Not an audit.**

This is an independent reading of the public write-ups Grok Bot published about one TN10 stress run on 25–26 Sep 2026. STP-KAS hosts the repository. The reading uses those public files, public HTTP endpoints, and the public rusty-kaspa v2.1.0 tree. It does not use the private repositories, and it does not repeat wallet addresses, seeds, or keys.

The front door of the set is [STP-KAS/tn10-vprogs-stress-findings](https://github.com/STP-KAS/tn10-vprogs-stress-findings). The round repositories are the lab notebooks behind it. Where a headline and the synthesis disagree, the synthesis is the more careful text, and even the synthesis still needs the metric definitions below.

Live checks in this note were taken when the TN10 blockDAG's past-median time was about **15:31 CEST on 26 Sep 2026**. The raw reads are in [CHECKS.md](CHECKS.md).

## How to read the set

Three different measurements are sitting under the words "TPS", "games", and "fees".

| Word in the notes | What the underlying counter actually is |
|---|---|
| Network TPS, including the 12,175 peak and the 5.7k median | Transactions inside block bodies the node processed, divided by the log window. kaspad adds `block.transactions.len()` to `txs_counts` for every processed body, and the 10-second consensus line prints that delta. The same transaction can sit in more than one block. This counter also includes coinbase. |
| Accepted tx/s in round 4 | Their own storm workers' submits that the node accepted. Other TN10 traffic is outside this number. |
| Finished games in rounds 5 and 6 (63,614 and 600,055) | Chains of ordinary L1 transactions with game state in the payload. Each chain remembers the output it just created. There is no vprogs runtime and no proof. |
| Finished games in rounds 3 and 4 (5, then 8, under the full storm) | The upstream tic-tac-toe runner, exec mode, proofs off, on a node that had `--utxoindex` at the time. |
| Gross TKAS fees | A fee counter. Their miners were finding a large share of TN10 blocks, so a large part of those fees came back as coinbase. The notes say about 57% of coinbase value, fees included, returned to those miners. That percentage is a separate estimate from the one 41-second window where the miners found about 65% of blocks. |

A 10-block-per-second selected chain, full of their low-mass unsigned P2SH payments (they measured mass 571), holds about **8,760 transactions per second if each transaction is counted once** (`10 × 500,000 / 571`). The published 10-second peak of **12,175** is above that. A short window can contain more than 10 blocks per second, and the body counter can count one transaction more than once. Both can be true. The peak is a processing-log peak. It is the wrong number to quote as selected-chain throughput.

The TPS lane that produced the high figures is mostly unsigned `P2SH(OP_TRUE)`, chosen because the mass is 571 instead of 1,624 for a signed 1-in-1-out payment. Signed minimum payments pack at about **3,079 per second**. A 0.5 TKAS payment plus change packs at about **250 per second**, because storage mass is about 20,000 and the block storage limit is 500,000. That last figure is the one both write-ups measured, and it matches the arithmetic.

## What I rechecked

**The public health document is still frozen, and it still says the indexer is synced.**

`GET https://api-tn10.kaspa.org/info/health` returned `isSynced: true`, `blueScoreDiff: 25`, kaspad blue score **568,825,293**, database blue score **568,825,255**, `acceptedTxBlockTime` **1790365505207** (25 Sep 2026 19:45:05 UTC, 21:45:05 CEST), and `serverVersion` **2.0.1**.

At the same time `GET /info/virtual-chain-blue-score` returned **569,462,361**. That is **637,068** blue score ahead of the health document's kaspad score, about 17 hours 42 minutes at 10 blue per second. `GET /info/blockdag` returned virtual DAA **580,980,207** and a past-median time about 15:31 CEST. The chain tip moved. The health document did not.

Grok Bot's morning note is still true on this afternoon read, and the lag is larger than the ~328k blue score they measured at 06:48 CEST. Their overload is a neighbor in time (the freeze sits a few minutes after the full-throttle phase began). The cause is still unknown. `/info/health` is a stale document while it reports that blue score and `isSynced: true`. The version string inside it belongs to that stale document. Grok Build's own public note already recorded the resolver handing out both kaspad 2.0.1 and 2.1.0.

**The hosted tic-tac-toe settlement they called frozen has moved.**

`GET https://vprogs-tt.izio.fr/api/state` returned `l2_tip` **873,933** and settled DAA **580,940,363**, settlement transaction `9bcba5323515fcbece8a5d4f02ba1dfabe521077ccc9feda899e6e19658ba852`. Virtual DAA at the same reading was **580,980,207**, a gap of **39,844** DAA. At 10 per second that is about 66 minutes. At the 12.5 per second implied by their "85.4 minutes" sentence, it is about 53 minutes.

Their evening sample was settled DAA 580,229,488 and `l2_tip` 566,858. Since that sample the settled score has risen by **710,875** and the L2 tip by **307,075**. The desk public note's earlier snapshot, settled DAA 580,137,056, is a third number. Settlement moved between the afternoon desk read and the evening stress read, and it has moved again since. The specific freeze is historical. A gap remains on this read. Draft PR 165's settler and exit-index changes are one candidate explanation for the earlier stall. This read does not show that those changes caused the later movement.

Their "64,057 DAA ≈ 85.4 min" uses about **12.5 DAA per second**. At the 10-block-per-second target the same gap is about **107 minutes**. The DAA gap is the measurement. The minute figure needs the rate sample next to it.

**PR 165 is the draft they describe, and it is not a retest.**

[kaspanet/vprogs#165](https://github.com/kaspanet/vprogs/pull/165) is open, draft, unmerged. Author `biryukovmaxim`. Created 26 Sep 2026 11:22 UTC (13:22 CEST). Base `fix/reorg-boundary-duplicate-bundles`. Head `081af9b9de0689b61e1a456cf296a4ad9e562b73`. The four commits are `36a6d38`, `bf3509c`, `494aacc`, `081af9b`. The files include `l1/wallet` carrier, payout, and lib, the risc0 settler worker, `runner/src/exit_index.rs`, and `runner/tests/two_provers_contend.rs`.

`release-candidate` currently points at that same head `081af9b`. The runs themselves were on `3a61c0b`. vprogs `master` is still `f9b84a863a7c7c20586a9cf947550475e894f72e` (28 Jul 2026). `biryukovmaxim/vprog-tictactoe` `master` is `803a120ba1449c1ecbadef46dd68586b8ce803f6` (26 Sep 2026 11:50 UTC). `f128efd` exists and is the parent-era commit they named (11:40 UTC).

I confirmed the pull request's identity and file list. I did not review the diff, and I did not run their suggested 30–60 minute retest. "Probably addressed by PR 165" stays a mapping until that retest exists. No comment was posted on the pull request.

## What holds up

These are the claims I would hand to someone working on the node or the vprogs client.

1. **Plain payments waited on fee during their overload, and 100× bought nothing over 10×.** The round-1 table is 117 probes per tier, none rejected, none evicted. 1× minimum: p50 7.0 s, max 105 s. 10×: max 3.7 s. 100×: the same max 3.7 s. The CSV in the synthesis matches the README. This is under a flood this setup created, on TN10, with `--ram-scale=0.1`. Round 1's leap from that table to a mainnet DEX, bridge, or liquidation is a different claim and does not follow from the table.

2. **A 0.5 TKAS output is the packing limit.** Grok Bot measured storage mass 20,001 and about 25 such payments per block. Grok Build's public note measured storage mass 20,000, compute mass 2,036, and a packing ceiling of 250 per second. The fee stays on compute mass. The block fills on storage mass. The round-5 150× fee on ~0.25 TKAS coins is the same mechanism in the other direction: the fee shrank the change, storage mass rose, the effective feerate fell, and throughput dropped. I did not recompute the ~69,000 gram figure from a raw transaction. The shape of the result matches KIP-9.

3. **The mempool count check is an assert, and their log shows it firing once at the cap they configured.** The message `Transactions in mempool: 100001, max: 100000` matches the assert that prints the pool length plus one against `maximum_transaction_count`. In the v2.1.0 sources the default count is **1,000,000**. `--ram-scale=0.1` scales it down, and their flags set that scale, so the cap in the crash is **100,000**. The byte side of the same panic was about 51 MB against a 100 MB scaled limit, so the count check is the one that fired. Later in their own logs the pool sat near 99,967 with evictions and did not panic again. The published evidence is one crash at the lowered cap, during their own flood. The assert is the wrong way to refuse one more transaction. A default 1,000,000 cap is a different run.

4. **Restart drops the mempool.** They recorded 58,035 transactions gone on a planned restart, and the crashed node's pool gone with the process. That matches a mempool that lives in memory. It matters to a client that assumes a submitted transaction is still there after the node restarts. It is operator-visible behavior of this kaspad, recorded here with counts.

5. **Upstream vprogs on that node required `--utxoindex`, and building the index was expensive.** 1,115 seconds of resync, RPC and P2P down, no progress line, about 13 GB, free disk from about 40 GB to about 11 GB, total downtime 1,153 seconds. During the later disk emergency they deleted that index to get the node back. After that, the upstream runners could not run there. Rounds 5 and 6 exist because of that choice.

6. **The upstream tic-tac-toe client lost to its own fee and to its own UTXO handling.** Under the full 10× storm, local `ttloop` finished **5** games and failed **1,322**, with the failure text `already spent by transaction ... in the mempool`. In the next full-storm window it finished **8** and failed **1,223**, and those 8 landed in the first seconds. Carriers paid the relay floor, so a proxy that raised `getFeeEstimate` did nothing for them. Workers then died in `carrier.rs` when the first UTXO was too small for the deposit (`98955500` too small for `100000000`, later `14913800` too small for `40000000`). The desk note had already recorded that `signed_carrier_transaction` asserts when the funding output cannot pay the extra outputs and the fee. The storm supplies the runtime counts. I did not re-read `carrier.rs` line by line for this note.

7. **The small guest rejected the impossible debits it was given, in exec mode.** 0 of 3,518 in round 3 and 0 of 4,291 in round 4. Proofs were off (`RISC0_DEV_MODE`, `TT_PROVE=0`). That is a guest rule check on a dev lane. It is the right kind of result, and it is small on purpose.

8. **Disk, not the mempool assert, ended the long run.** Round 2's own text is clear: free disk drove the taper from about 9.7 GB to about 8.5 GB, the offered rate fell because the supervisor cut it, and the recovery watch fired at mempool 7. There was no drain to measure. The 26 Sep 07:03 pruning move, storm already off, grew consensus data from about 75 GB to about 89 GB and filled the disk. That is an operator log. The useful part is the size of the transient spike on a node that had just stored a high-throughput day, plus a utxoindex beside it.

9. **Grok Build's public wallet-load note already limits its own peak.** 1,162 tx/s is 340 sweeps in 0.29 seconds, all included. The sustained rate was bounded by confirmed inputs and public wRPC, and the note says so. Signing at about 13,200 tx/s is a local CPU measurement on that host. The synthesis line "552 and 1,162 tx/s offered, all included" is fair while the 0.29 seconds stays attached.

## What the headlines oversell

**Rounds 5 and 6 are an L1 chaining test.** 1.04 million and 10.1 million runner transactions, sub-second p50 at a high feerate on large inputs, mempool highs of 85,827 and 78,319: those describe payload chains that track their own outputs and subscribe to `virtual-chain-changed`. The synthesis says this in prose. The round 5 and round 6 metric tables still label the rows "tic-tac-toe games" and "vprog programs". The illegal-move counts (207,770 and 2,026,964, zero executed) were refused in the client before a transaction was built. The guest result is the few thousand impossible debits in rounds 3 and 4.

**Round 2's 7 hour 44 minute median of 1,548 TPS mixes three regimes.** Their own regime table is the one to use. The comparable full-throttle hour at 2× fee is median **7,687**, peak **10,846**, over about 62 minutes, and that number is still the processing log. The 12,175 peak sits in a later 10× slice that also contains a silent stretch. Quoting the whole-window median or the single peak as "TN10 did X for 8 hours" flattens the log.

**Round 3 is inside round 2's clock.** Round 2 runs 22:47 on 25 Sep through 06:31 on 26 Sep. Round 3 runs 23:48 through 01:13, on the same node and the same storm. Adding the rows counts that interval twice. The wall clock of the whole exercise is about 17 hours, from 20:12 on 25 Sep to 13:16 on 26 Sep, with restarts and the index rebuild inside it.

**The fee totals are gross.** Round 2 storm fees about 197.8k TKAS are a sum across supervisor restarts, which is the right way to handle a counter that resets, and the note shows the three pieces. Round 5 and round 6 runner counters, about 954k and 685k TKAS, plus about 134k TKAS of KNS prices, are gross. Net cost to the operator is smaller, and no single reconciled balance sheet is in the public set. One KNS restart stranded an estimated 26k TKAS in keys that were only in memory. That loss is disclosed. It is an accounting hole, separate from the fee counter.

**"One desktop" is the operator count.** The run was one person and one model in two sessions. The machines were a separate 8-core / 16 GB node, miners that took a large share of TN10 blocks, on the order of 800 throwaway wallets, and a second spender (the Grok Build wallet load) whose traffic sits inside the processing log and outside the storm fee counter. Round 2 says that plainly. Round 5 also records an external flood that was not theirs, with the mempool already high before their runners carried the rate. TN10 load in these logs is theirs, plus that other flood, plus whoever else was sending.

**KNS "1,883 names created" is a client counter against a lagging indexer.** They recorded the public indexer about 235k DAA behind, and ownership checks deferred (`owner_unk`). A created-event is what their script counted. It is not a caught-up uniqueness result. Short random names on a public testnet indexer are a load generator. [kns-spec](https://github.com/STP-KAS/kns-spec) and [kns-tn10-testing](https://github.com/STP-KAS/kns-tn10-testing) are different repositories and are outside this reading.

**The explorer reward check is a good short method, on short windows.** Balance change matched the coinbase sum to the sompi over 20 seconds and over 90 seconds, after an earlier 120-second window missed one reorged-out block. Twelve sampled blocks were blue. The transaction list and `/info/health` were already stale, which this afternoon's read still shows. The note prints a full mining address. This opinion leaves addresses out. The windows are too short to be a lifetime reconciliation, and the note says that.

## Repository by repository

| Public repo | Reading |
|---|---|
| [tn10-vprogs-stress-findings](https://github.com/STP-KAS/tn10-vprogs-stress-findings) | Best of the set. Caveats are in the prose. The round table and `rounds-summary.json` still invite a skimmer to add overlapping rounds and to treat round 5–6 games as vprogs. |
| [grok-bot-vprogs-round1-public](https://github.com/STP-KAS/grok-bot-vprogs-round1-public) | Strongest primary log: panic excerpt, fee-tier probes, storage-mass cap, utxoindex requirement. The "why" section carries the result onto mainnet. History is squashed, so the commit chain is not the original lab history. |
| [grok-bot-vprogs-round2](https://github.com/STP-KAS/grok-bot-vprogs-round2) | Best throughput write-up, because it splits regimes and says the recovery measurement is empty. The related-link line still calls the explorer note private. The explorer note is public. |
| [grok-bot-vprogs-round3](https://github.com/STP-KAS/grok-bot-vprogs-round3) | The real vprogs client evidence: fee starvation, carrier panic, mempool UTXO reuse, exec-mode guest. Shares its clock with round 2. |
| [grok-bot-vprogs-round4](https://github.com/STP-KAS/grok-bot-vprogs-round4) | Separates worker-accepted rate from processed TPS, and records the pruning disk emergency and the 96,546 mempool near-miss. The opening still calls round 5 private. |
| [grok-bot-vprogs-round5](https://github.com/STP-KAS/grok-bot-vprogs-round5) | Useful for the 150× fee backfire, the external flood, and `GET /addresses/.../utxos` being stale while `POST /addresses/utxos` was live. The headline game counts are the index-free runner. |
| [grok-bot-vprogs-round6](https://github.com/STP-KAS/grok-bot-vprogs-round6) | The long index-free run, the 2×-normal fee rule, and the KNS client counter. Network TPS here is explicitly from snapshots because `nettps.jsonl` went stale after 07:46. That is the right disclosure, and it means the round-6 "network" figures are a different instrument from rounds 1–4. |
| [grok-bot-explorer-rewards-check](https://github.com/STP-KAS/grok-bot-explorer-rewards-check) | Read-only, and the health-document claim survived a same-day re-read. Address is in the file. Cause of the stall is still timing, which they say. |
| [vprogs-tn-desk-public](https://github.com/STP-KAS/vprogs-tn-desk-public) | Earlier desk note. Pins in it are the 25 Sep pins (`3a61c0b`, tictactoe `ba05d924`). Those pins moved on 26 Sep, as above. Settlement snapshot is the earlier DAA, not the evening one. The SilverScript point (release tag v1.0.0, language constant still 0.1.0) is a separate fact. This opinion does not recompile it. The public copy includes receive addresses and UTXO counts. |
| [grok-build-vprogs](https://github.com/STP-KAS/grok-build-vprogs) | Public. The description still says "Private." The README says "Private on purpose." It is the wallet-load note: mass table, short bursts, one hosted game whose L1 transactions were included while that session's demo state stayed put. The demo has moved since that session. |

[sixpack.wtf](https://github.com/STP-KAS/sixpack.wtf), [kaspa-master-file](https://github.com/STP-KAS/kaspa-master-file), [kns-spec](https://github.com/STP-KAS/kns-spec), and [kns-tn10-testing](https://github.com/STP-KAS/kns-tn10-testing) are on the same account and are outside this stress reading.

`grok-bot-vprogs` and `vprogs-tn-desk` are private. This opinion does not use them. The public "clean copies" are squashed history. Check a number against the log file that was copied with it.

I scanned the public trees for a committed mnemonic line, a bare 12- or 24-word phrase line, and a 64-hex private-key assignment. None turned up. That scan is not a proof that every log line is harmless. The explorer note and the desk public copy do publish testnet addresses. Those addresses are left out of this repository on purpose.

## What I did not do

I did not send a transaction, start a node, or start a miner. I did not re-run the storm, the guest, or the suggested PR 165 retest. I did not review the PR diff and I did not comment on it. I did not treat dev-mode execution as a proof, or the index-free runners as vprogs.

A next measurement that would actually move these findings is the one the synthesis already asks for, on a node with `--utxoindex`, against the PR head: games finished during a storm, fee per game, how often a carrier pays almost the whole coin, carrier assert panics, and in-mempool reuse. Beside that, every TPS table wants three columns that the current scripts already almost have: selected-chain accepted transactions, processed block-body transactions, and this sender's accepts. One window of fees against miner coinbase would turn the 57% sentence into a net number.

The assert at the mempool cap wants a run at the default cap as well as at `--ram-scale=0.1`. The crash in the log is the lowered cap.
