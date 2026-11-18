# Web3 Security Research

Notes, pattern studies, and educational write-ups from my smart-contract security
research and bug bounty work.

**Ground rules for everything in this repo:**

- Every write-up covers an issue that is **already publicly disclosed and fixed**,
  or a general pattern documented in public audits. Nothing here comes from a live,
  unresolved report.
- All proof-of-concept work is done on **local forks / local testnets only**. Never
  run anything from here against mainnet or a live system.
- Code snippets marked *illustrative* are simplified teaching examples, not the
  original researchers' PoCs.

## Method

1. Read the full in-scope source at a pinned commit.
2. List candidate issues; try to refute each one on the code before believing it.
3. Confirm survivors with a local PoC — a candidate is not a finding until then.
4. Disclose through the program's official channel; publish only after resolution.

## Write-ups

| # | Title | Area | Status of the issue |
|---|-------|------|---------------------|
| 001 | [Nested-execution griefing in ERC-4337 EntryPoint (≤ v0.8)](writeups/001-erc4337-nested-execution-griefing.md) | Account abstraction | Disclosed Feb 2026, paid, fixed in v0.9 |

## Focus areas

- ERC-4337 account abstraction (EntryPoint, paymasters, validation/execution flow)
- Cross-chain bridges, exit predicates, oracle & governance committees
- Access control, initialization, and approval surfaces in launch contracts

## About

I'm [bungseula](https://github.com/bunsel01) — Web3 security researcher, active on
Immunefi, Cantina, Sherlock, and HackenProof.
