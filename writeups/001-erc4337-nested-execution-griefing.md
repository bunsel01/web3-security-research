# 001 — Nested-Execution Griefing in ERC-4337 EntryPoint (≤ v0.8)

**Area:** ERC-4337 account abstraction — EntryPoint validation/execution flow
**Affected:** `eth-infinitism/account-abstraction` tags v0.6.0, v0.7.0, v0.8.0
**Fixed in:** v0.9.0
**Disclosure status:** Publicly disclosed by TrustSec (February 2026) and paid as a
High ($50K) through the Ethereum Foundation's account-abstraction bug bounty on
HackenProof. Safe to study and discuss.

> Pattern study for educational purposes. The snippet below is an illustrative
> simplification of the pattern, not the original PoC. Local experimentation only.

## The pattern

ERC-4337 separates a user operation into a **validation phase** (is this operation
legitimate, and who pays?) and an **execution phase** (run the call). The EntryPoint
is a singleton: every account, paymaster, and bundler interacts through the same
contract, and operations can cause further calls — including, in the affected
versions, calls that re-enter the EntryPoint's execution context while an outer
operation is still being processed.

That nesting is the hazard. When an inner execution can influence the state or the
gas accounting of the outer operation *after* the outer operation has already
passed validation, an attacker gains a lever that validation was supposed to take
away:

- Force the outer operation to **revert after validation succeeded** — the classic
  griefing outcome: the operation is dropped, the bundler's work is wasted, and if
  it can be repeated cheaply, specific accounts or paymasters can be censored.
- Distort **who pays for what**: gas and deposit accounting that assumes a single,
  linear execution flow can be made to charge costs to the wrong party.

The important detail is that none of this requires breaking signature validation
or stealing funds. In account abstraction, *griefing is a severity class of its
own* — the EF program's ladder rates "on-chain griefing/censorship, including
post-validation forced revert" as **High**, which is exactly where this issue
landed.

## Why it survived multiple versions

The vector was present by design across v0.6–v0.8: nested execution was a
legitimate feature of how operations compose, and each version's audits focused
heavily on the validation rules (ERC-7562) and signature paths. The bug lived in
the *interaction* between composition and accounting — the kind of issue that only
appears when you model the EntryPoint as a stateful system under adversarial
re-entry, not when you read each function in isolation.

It was fixed in v0.9 as part of hardening the execution flow, and the disclosure
immediately made the older tags' version of the vector ineligible for further
reports — a useful reminder that in bounty hunting, a real bug in old code can be
worth $0 if someone else disclosed it first.

## Illustrative shape (simplified)

```solidity
// ILLUSTRATIVE — the pattern, not the original code or PoC.
// An operation whose execution calls back into the EntryPoint
// before the outer operation has finished accounting.

function handleOp(UserOperation calldata op) external {
    validate(op);            // phase 1: checks pass, payment reserved
    execute(op.callData);    // phase 2: target call can re-enter handleOp()
    settlePayment(op);       // accounting assumes phase 2 was atomic & linear
}
```

If `execute` can trigger a nested operation that mutates the state `settlePayment`
relies on — or forces a revert that validation promised would not happen — the
linear assumption breaks.

## Defensive takeaways

1. **Treat post-validation reverts as a security property**, not a UX detail. If
   your protocol's safety story says "once validated, it executes," any path that
   breaks that promise is a finding.
2. **Audit the composition, not just the functions.** Re-entry into a singleton
   coordinator (EntryPoint, router, vault) deserves the same suspicion as
   re-entry into a token contract.
3. **Gas/deposit accounting must be re-entry-aware.** Any ledger updated after an
   external call should be checked against nested mutation of the same ledger.
4. **Check the disclosure lists before hunting old versions.** A bug that is real
   in code but already disclosed and paid is a duplicate on every version — verify
   the known-issue surface first, as the programs themselves instruct.

## References

- Ethereum Foundation "Account Abstraction Bugs" program, HackenProof (scope,
  severity ladder, eligible tags).
- `eth-infinitism/account-abstraction` — v0.9.0 release (fix) vs v0.6–v0.8 tags.
- Public audit history for the codebase: OpenZeppelin (2023–2024), Spearbit
  (Mar 2025), Cantina (v0.9 review).
