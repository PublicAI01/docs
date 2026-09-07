# Rewards

Trajector pays you for the coding sessions you contribute. This page explains exactly how much you earn, when it lands in your balance, and how you get it out.

The short version:

> **You earn USD for every token in an accepted session. You withdraw that balance as a stablecoin (USDC or USDT) on Ethereum or BSC once it reaches $10. The network fee is deducted from the payout.**

### During the beta: what is collected

Only **`claude-fable-5`**, **`claude-fable-5-1`** and **`claude-opus-5`** coding sessions are collected and rewarded right now. If you are already on one of those, there is nothing to do. Otherwise switching takes one flag:

```bash
# One-off: launch Claude Code on a collected model
claude --model claude-opus-5

# Make it stick for every session (add to your shell profile)
export ANTHROPIC_MODEL=claude-opus-5
```

Inside a running session you can also type `/model claude-opus-5` (or `/model claude-fable-5`).

What happens to everything else:

* Sessions from any other model are **rejected**: they earn nothing and never enter anything we deliver. They appear as _rejected_ in your session list, you can remove them yourself at any time, and ones you leave are cleared under the beta retention policy.
* Rejected data is **not retroactively accepted** if the scope later widens — please don't keep uploading in the hope that it will be.
* Sessions waiting on manual review are kept until a decision is made.

Tip: sessions recorded with thinking effort set to `max` are more valuable to the current buyer. That is a recommendation, not a requirement — every effort level is accepted and rewarded the same.

Withdrawals are open during the beta. Beta rules — collection scope, rates, limits and acceptance rules — may change, as posted here; rewards already credited are never repriced.

***

### 1. How much you earn

Rewards are priced per **LLM token**, not per session, per hour, or per line of code:

```
reward (USD) = tokens in the accepted session × rate per token
```

The current rate is **$0.000001 per token** — that is, **$1.00 per million tokens**.

| Tokens contributed | Reward |
| ------------------ | ------ |
| 100,000            | $0.10  |
| 1,000,000          | $1.00  |
| 10,000,000         | $10.00 |

A few things worth knowing about the rate:

* **It is set by the platform and can change.** It is a configuration value, not something hard-coded into a release, so it can be adjusted as the business evolves.
* **A rate change is never retroactive.** The rate in effect at the moment a session is credited is written permanently into that ledger entry. Rewards you already earned are never repriced — up or down — by a later change.
* **The rate is the same for everyone.** There is no per-user pricing.

#### What counts as a token

A session's token total is the sum of three parts:

| Part                   | What it covers                                                                          |
| ---------------------- | --------------------------------------------------------------------------------------- |
| **Input tokens**       | Your prompts, the system prompt, and the definitions of tools the agent actually called |
| **Output tokens**      | The agent's completions, including its thinking                                         |
| **Tool output tokens** | The payloads returned by tool calls                                                     |

Notes on how this is counted:

* When your model provider reports exact token usage, that number is used as-is. When it doesn't report a field, that field is estimated with a deterministic counter.
* Embedded images and other base64 blobs are stripped before counting. You are paid for text that a human or a model actually wrote.
* Tool results are counted **once**, even though agents re-send them on every subsequent request in the same conversation. This is a deliberate choice: counting them per request would inflate long sessions, and the count needs to mean the same thing for everyone.
* The count is computed identically on your machine and on the server, so what the CLI shows you and what you get paid for do not drift apart.

***

### 2. When a reward lands in your balance

Uploading a session is not the same as earning from it. Every uploaded session is checked on the server, and lands in one of three states.

#### ✅ Accepted — credited

The session passed every check. Your **Claimable Reward** goes up immediately, and the session appears in **My data** with the reward it earned.

#### ⏸️ Held — under review, not lost

Something about the session needs a human to look at it. Held sessions earn nothing _yet_. If a reviewer accepts one, it is credited at that point exactly as if it had been accepted on arrival — nothing is lost by being held.

Common reasons a session is held:

* **Daily volume caps.** There is a per-account and a per-device cap on tokens credited per UTC day. Going over the cap does not destroy the sessions — they queue for review, because legitimately high-volume contributors exist.
* **Thin sessions.** No real prompt from you, no tool call, and no file change. There is nothing to learn from a session where nothing happened.
* **Quality gates.** The trajectory didn't meet the collection thresholds (for example, too few assistant turns, or a tool-error ratio high enough that the session is mostly failure noise).
* **Fragments.** Content that is already fully contained in another session you uploaded.

#### ❌ Rejected — never credited

Rejection is reserved for two categories only — **fraud and compliance** — and it is final:

* **Forged trajectories.** Sessions that were not produced by a real agent run.
* **Secrets or personal data found by the server's second-pass scan.** The CLI redacts locally before anything leaves your machine; the server scans again as a safety net. A hit means we cannot license that data, so it is not credited.
* **The same content uploaded from more than one account.** One recording, one payment. Coordinated re-uploading across accounts is the thing this check exists to stop.

Everything that is _not_ fraud or compliance is held for review rather than rejected.

***

### 3. Reading your balance

The dashboard shows three numbers:

| Number               | Meaning                                                               |
| -------------------- | --------------------------------------------------------------------- |
| **Claimable Reward** | What you can withdraw right now. Goes to $0 when you claim.           |
| **Total Reward**     | Everything you have ever earned. Does **not** go down when you claim. |
| **Total Tokens**     | Lifetime tokens credited across all accepted sessions.                |

Below them you'll find a breakdown by agent type (Claude Code, Cursor, Codex, Gemini CLI, …) with tokens and session counts, and a daily contribution chart so you can see your contribution rhythm over time.

Every movement in your balance is recorded in an append-only ledger. Nothing is ever edited or deleted — a correction is posted as its own entry, so the history always adds up to the balance you see. Entry types:

| Type       | Effect                                                                 |
| ---------- | ---------------------------------------------------------------------- |
| `accrual`  | + a session was credited                                               |
| `claim`    | − a payout was sent to you                                             |
| `fee`      | − the network fee on that payout                                       |
| `reversal` | ± a correction (a failed payout returning your balance, or a deletion) |

#### Why one conversation can appear as several "parts"

The CLI uploads in batches, so one long conversation may be flushed to the server in several slices. **My data** groups them back into one row per conversation and shows a `parts` count. Each slice carries its own distinct API calls and its own tokens, and **all of them are paid** — you are not paid only for the biggest slice.

***

### 4. Claiming your rewards

#### Requirements

* **Minimum $10.00.** Below that, the claim button is disabled and your balance keeps accumulating.
* **A wallet address** bound for the network you're claiming on. You can add or change it at claim time, and it is saved for next time.
* **Networks:** Ethereum and BSC.
* **Assets:** USDC and/or USDT, depending on what is currently open for that network. If more than one is open you choose; if exactly one is open it is selected for you.

#### The flow

1.  **Preview.** Before you confirm anything, the dashboard shows you three lines: claimable amount, estimated network fee, and the net amount you will receive.

    ```
    Claimable amount          $42.00
    Estimated network fee   − $ 1.30
    ─────────────────────────────────
    You receive               $40.70
    ```

    The network fee (typically $1–2) is deducted from the payout, not billed separately.
2. **Confirm.** Your claimable balance drops to $0 the moment the claim is created — before the transaction is confirmed on chain. This is intentional: the money is reserved for your claim so it cannot be spent twice by a second request. Your **Total Reward** is unaffected.
3.  **Settlement.** The payout worker builds, signs, and broadcasts the transfer, then waits for on-chain confirmation. This takes anywhere from seconds to a few minutes.

    | State       | Meaning                                                                            |
    | ----------- | ---------------------------------------------------------------------------------- |
    | `pending`   | Created and queued or broadcast; waiting for confirmation                          |
    | `confirmed` | On chain. The transaction hash is recorded and shown to you                        |
    | `failed`    | Did not go through — **your full balance, fee included, is returned to Claimable** |

**A failed payout never costs you anything.** The balance is restored in full and you can claim again. Claims never leave you with a negative balance, and a stuck transaction never silently eats a balance.

Claims are rate-limited per account. This protects the service, not your money — a rate-limited request is simply refused, never partially executed.

***

### 5. Deleting your data, and what it does to your rewards

You can browse every session you have uploaded under **My data**, including the exact redacted payload we hold — the same bytes a data buyer would receive.

**Sessions whose reward you have not yet withdrawn can be deleted yourself.** Deleting one blanks the stored payload and **reverses that session's reward out of your claimable balance**. The exact amount to be reversed is shown in the confirmation dialog before you commit. This unwinds the exchange cleanly: we no longer hold the data, and we no longer owe you for it.

**Sessions whose reward has already been paid out cannot be deleted with a button.** That data is the consideration for a payment already sent on chain. Submit a deletion request instead: it goes to a review queue, and when it is approved the payload is erased for real. **The money already paid is not clawed back** — your right to erasure does not depend on returning it.

Two things survive a deletion, deliberately:

* **A content fingerprint** of the session (a hash, not readable content). It carries the duty of preventing the same content being uploaded again for a second payout.
* **The ledger history.** Your accrual and its reversal both remain visible, so your balance is always reconstructible from the record.

***

### FAQ

**Do I get paid more for better code?** Not in this version. Pricing is purely per token on accepted sessions. A retention-based multiplier — rewarding agent-written lines that survive to a commit — is a planned future addition, and the attribution data needed for it is already being stored.

**Does re-uploading the same session pay twice?** No. Each session is credited exactly once, and duplicate uploads of the same API calls are detected and not paid again.

**My session was held. What do I do?** Nothing. Held sessions sit in a review queue. If accepted, they are credited at that point.

**Can my rate change after I've contributed?** The rate can change going forward, but never for rewards already credited — the rate is locked into each ledger entry when it is written.

**I claimed and my balance went to zero, but the transaction isn't confirmed yet.** That's expected. The balance is reserved at the moment you claim. If the transaction ends up failing, the full amount — including the fee — returns to your claimable balance automatically.

**Why is my Total Reward still showing an amount after I claimed?** Total Reward is lifetime earnings and never resets. Only Claimable Reward goes to zero.
