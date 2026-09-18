# Coffer — Security Audits

Security audit reports for the on-chain programs of [Coffer](https://coffer.so) (a weighted multi-token AMM on Solana, formerly "Cube DEX").

The audits were performed by [Serokell OÜ](https://serokell.io) against the **private** contracts repository. The reports are published here for transparency; the source repository itself is not public.

## Reports

| File | Title | Reviewed revision | Date |
|------|-------|-------------------|------|
| [`serokell-initial-audit.pdf`](./serokell-initial-audit.pdf) | Smart Contract Code Review and Security Analysis Report for Cube DEX | `070a55dac94fe539c5896feb33ac719c3f6b9d4b` | 31 July 2026, statuses revised 9 September 2026 |
| [`serokell-audit-fixes.pdf`](./serokell-audit-fixes.pdf) | Follow-up Security Review (Second Round) — verification of the fixes | `6f64097c60630676e7fa779b460e8237dd7ac533` | 9 September 2026 |

Scope of both reports: the three on-chain programs (`cubic-pool`, `protocol-admin`, `single-token-liquidity`). Off-chain tooling, the SDK, the frontend and the backend were out of scope.

Methods used by the auditors: architecture review, two independent manual line-by-line reviews, functional testing, static analysis, dependency audit, stateful fuzzing, and runnable proof-of-concept exploits.

## Deployed programs (Solana mainnet-beta)

The revision reviewed in the second round (`6f64097`) is the revision deployed on mainnet. All three programs were upgraded in place on 7 September 2026.

| Program | Program ID |
|---------|------------|
| `cubic-pool` (core AMM: pools, swaps, liquidity) | `8iQtGj9mcUfFUGaiCpPy89swC3s8YTC8FhVZWfgeZhwu` |
| `protocol-admin` (governance, Treasury PDA) | `3jiojHZbjJQ7QLMGSTjFwxVEmx4NtuRy34nLAmsJME81` |
| `single-token-liquidity` (single-sided deposit) | `7BpdUH1tzTSXLuQNo6YpjJ8Eagw8AkrS6cnkxiJdCFS2` |

Upgrade authority of all three programs: the Treasury PDA `B4gyhrqLzX36VEu54ShqgTkb4TpXooTdxtcmvER3d3Bg`, controlled by protocol governance.

## Outcome

- **No Critical issues were found** in either round.
- The initial review reported 36 items (8 High, 6 Medium, 13 Low, 7 Informational, 2 accepted design notes). In the follow-up review the auditors confirmed **29 of 36 resolved**; the remainder are items the team consciously accepted (see below).
- The follow-up review raised 12 additional items (1 High, 2 Medium, 4 Low, 5 Informational), most of them residuals of, or consequences of, the original fixes.

**Every finding in both reports is in one of two states in the deployed revision: resolved, or explicitly acknowledged and accepted by the Coffer team as a deliberate product/design decision.** There are no findings the team is unaware of or has left unaddressed by oversight. The accepted items are listed here so that users and integrators can make their own judgement:

| Item | What is accepted | Why |
|------|------------------|-----|
| H-05 / H-09, L-17 | Pool creators may override the default Token-2022 extension ban list; only `NonTransferable` is a hard floor. A pool creator who admits an issuer-controlled mint (e.g. `PermanentDelegate`) exposes that pool's LPs to the issuer. | Pool creation is permissionless and the chosen bitmap is stored on the pool and emitted in events, so LPs and frontends can inspect it before depositing. Full Token-2022 support is planned for a later release. |
| L-09 | Protocol-admin rotation may target any key, not only the Treasury PDA. | Allows handover to a multisig/DAO/recovery key. |
| I-04 | Sell-off cap and surge fee are off by default. | Per-pool opt-in by the pool admin. |
| M-07, L-16 | The single-token deposit ("zap") uses a linear split, so a large deposit relative to pool depth mints fewer BPT than a curve-aware split would; the unused excess is refunded to the depositor. | No fund loss; a curve-aware planner is scheduled for the next release. |
| M-08 | Rounding in `add_liquidity` can favour the depositor by at most one raw unit per token per call. | Economically negligible. |
| L-14, L-15, I-12 | Sell-off window edge cases involving range-manager updates, cap toggling, and thresholds placed near 100% fill. | Only reachable on pools with the opt-in sell-off cap enabled; fixes are scheduled for the next release. |
| H-07 (residual), I-11 | `migrate_to_v5` reactivates disabled tokens only when explicitly requested, and trusts operator verification of legacy pool bytes. | Operator-controlled instruction with off-chain pre-checks. |
| I-08, I-09, I-10 | Pool-creation PDA is front-runnable (mitigated in tooling), `PoolInitialized` lacks `pool_id`, range-manager band can require admin intervention after large organic drift. | Tooling / indexing / operational items. |
| S-01, S-02 | Just-in-time liquidity can skim a share of LP fees; withdrawing liquidity amplifies price impact of the next swap. | Inherent properties of the AMM design. |

## Disclaimer

An audit is a point-in-time review of a specific revision and does not constitute a guarantee that the code is free of vulnerabilities (see the Disclaimers section of each report). Use the protocol at your own risk.

If you believe you have found a security issue, please contact the team privately via [coffer.so](https://coffer.so) before any public disclosure.
