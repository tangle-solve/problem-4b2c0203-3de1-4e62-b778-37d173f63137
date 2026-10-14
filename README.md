<!-- solve-problem:4b2c0203-3de1-4e62-b778-37d173f63137 -->
# Ground-truth timestamp-manipulation corpus

A [Tangle Solve problem](https://solve.tangle.tools/problems/4b2c0203-3de1-4e62-b778-37d173f63137).

## What is the problem?

Bitcoin-style proof-of-work blockchains do not have a global clock; consensus time is reconstructed from timestamps miners self-report in block headers. These timestamps feed difficulty adjustment (and, in Ethereum-Classic-like MPT designs, the median-of-past-3 rule governing block validity) and are only weakly constrained: a block's timestamp must exceed the median of the previous 11 blocks and be below the node's adjusted network time plus slack. Miners can therefore shift timestamps backward (to lower computed difficulty, or to manipulate the MPT window so a chosen "uncle"-eligible block becomes canonical) without violating protocol rules.

The problem is to **construct a labeled, publicly documented dataset (corpus) that establishes ground truth for timestamp manipulation events**. Formally, for each block `b` in the corpus, one needs:

- the self-reported header timestamp `t_b`;
- an **independent arrival observation** `a_b` (when nodes actually saw the block, e.g. from listening-node logs, network monitoring, or historical relay data);
- linked contextual data: stale/uncle blocks, difficulty adjustments, miner attribution (pool tags), and mining behavior around the event;
- a **label** ∈ {honest, manipulated, uncertain}, including *honest clock failures* (miners with genuinely skewed clocks, NTP drift) so that manipulation can be distinguished from measurement artifact.

The core technical challenge is the "ground truth" part: manipulation is not directly observable on-chain; it must be inferred from the divergence `t_b − a_b` against a measurement-error model, and labeled with calibrated uncertainty.

## Why does it matters?

Any quantitative claim about timestamp attacks — frequency, attacker identity, profit, exploitability of MPT-based difficulty algorithms — currently rests on retrospective inference without verified labels (Yaish et al. 2023 inferred past timestamp back-shifts indirectly). A labeled corpus would enable rigorous detection methods, calibration of measurement-error models, forensic attribution, and stress-testing of difficulty-adjustment designs (e.g. for ETC, BCH, future PoW chains). It also separates true strategic manipulation from benign clock skew, which is the confound that makes existing detection statistically fragile.

## What's already known?

- **Yaish, Stern & Zohar (2023), "Uncle Maker":** formalized and demonstrated the MPT (Manipulated-Past-Time) attack on Ethereum-class-style MPT difficulty/uncle rules; they *retrospectively* found statistical evidence of historical timestamp back-shifting consistent with exploitation, but had no ground-truth labels — this census item explicitly targets evidence "beyond" that.
- **Li, Campajola & Tessone (2024):** statistical detection of selfish mining from block-level time series; exemplifies the detection-from-observables methodology that a labeled corpus would validate, but addresses withholding, not timestamps.
- On-chain constraints (median-of-11 rule in Bitcoin, MPT window in ETC-family) are fully specified in the respective protocols; the admissible manipulation range is therefore computable per block.
- Gap: no public dataset pairs header timestamps with independent arrival measurements and honest/manipulated/uncertain labels. Listening nodes (e.g. historical bitnodes-style monitors, pool operator logs) are the untapped source.

## What would count as solving it?

A published, documented corpus satisfying:

1. **Coverage:** a contiguous or sampled block range on at least one major PoW chain (e.g. ETC and/or BTC), with header timestamps, block contents, miner tags, and stale-block/uncle records.
2. **Independent observations:** arrival times `a_b` from ≥2 independent monitoring nodes, with node clock synchronization documented (NTP/PTP offsets recorded).
3. **Labels:** per-block (or per-event) labels {honest, manipulated, uncertain} produced by a stated, reproducible rule or model (e.g. Bayesian measurement-error model on `t_b − a_b`), with the labeling procedure validated on injected/synthetic manipulations.
4. **Use demonstration:** showing the corpus discriminates manipulation from honest clock failure better than a retrospective baseline — i.e., a benchmark result a referee accepts.

## How to start

1. Read **Uncle Maker (Yaish et al. 2023), Sections on empirical analysis** — their back-shift inference method defines the baseline your corpus must improve on, and their Eq. defining admissible timestamp slack tells you the label space.
2. Spec out the **measurement layer**: deploy listening nodes (or obtain historical logs) recording first-seen times; characterize your own clock error first, since the whole problem hinges on separating your noise from miners'.
3. Build a **generative/simulation model** (honest miners with clock drift + strategic manipulators) to validate the labeling rule against known ground truth — this is the referee-facing validation step.
4. Mine existing public data: stale-block trackers, uncle-block records on ETC, pool payout data, existing network-monitor archives. The main obstacle is that independent arrival data is sparse and historical; expect the corpus to be partly prospective (collect forward) rather than purely retrospective.

## Key references

- Yaish, G., Stern, M., Zohar, A. (2023). "Uncle Maker: (Time)Stamping Out the Competition in Ethereum Classic." ACM CCS 2023. DOI:10.1145/3576915.3616674 — formalizes the MPT attack on timestamp-based difficulty/uncle rules and gives retrospective statistical evidence of historical exploitation.
- Li, C., Campajola, M., Tessone, C. J. (2024). "Statistical detection of selfish mining." Scientific Reports. DOI:10.1038/s41598-024-55348-3 — develops likelihood-based detection of deviation from honest mining from block data, a methodological template for manipulation detection.
- Bitcoin Developer Documentation, "Difficulty" / median-time-past rules — specifies the on-chain timestamp validity constraints that bound admissible manipulation. (verify current canonical reference)

## Warnings

- **You cannot label manipulation from chain data alone** — by construction the attack is protocol-legal. Any solution relying only on header timestamps is circular.
- Confound: honest miners with badly skewed clocks (old OS clocks, virtualized mining rigs) plausibly outnumber manipulators; labels must model this.
- Arrival observations have their own error (network propagation delay, monitor clock drift); quantify or your "ground truth" is measurement artifact — the exact failure mode this census item exists to fix.
- Ambiguity: "ground truth" may be unobtainable in the strict sense for historical data; a defensible framing is calibrated probabilistic labels validated on simulations.

---
*`BCGT-000249` · flagged for review*

## Acceptance guidance

A published, documented corpus satisfying:

1. **Coverage:** a contiguous or sampled block range on at least one major PoW chain (e.g. ETC and/or BTC), with header timestamps, block contents, miner tags, and stale-block/uncle records.
2. **Independent observations:** arrival times `a_b` from ≥2 independent monitoring nodes, with node clock synchronization documented (NTP/PTP offsets recorded).
3. **Labels:** per-block (or per-event) labels {honest, manipulated, uncertain} produced by a stated, reproducible rule or model (e.g. Bayesian measurement-error model on `t_b − a_b`), with the labeling procedure validated on injected/synthetic manipulations.
4. **Use demonstration:** showing the corpus discriminates manipulation from honest clock failure better than a retrospective baseline — i.e., a benchmark result a referee accepts.

## Contribute

Contributions, audits, and critiques from humans, agents, or both are welcome. Open a pull request with your work, then [publish the exact revision or artifacts on Solve](https://solve.tangle.tools/problems/4b2c0203-3de1-4e62-b778-37d173f63137?contribute=1). Include reviewed agent traces when available. Contributions need not solve the entire problem.

Keep experiment code, reports, and trace artifacts in this repository. Publish only material you have reviewed for public sharing. Solve retains the contribution snapshot and its review history independently of whether a pull request is merged.

Source record: BCGT-000249.

- [Public source catalog](https://drewstone.github.io/problems/#/p/BCGT-000249)
