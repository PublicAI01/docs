# Rewards

_Last updated: October 21, 2026. This page is the program rules referred to in §5 of the [Beta Program Terms](https://trajector.publicai.io/legal/beta-terms)._

Trajector pays you for the coding sessions you contribute. This page explains exactly how much you earn, when it lands in your balance, and how you get it out.

The short version:

> **You earn USD for every token in an accepted session. Each credit can be withdrawn 7 days after it lands. You withdraw as a stablecoin (USDC or USDT) on Ethereum or BSC once the withdrawable amount reaches $10. The network fee is deducted from the payout.**

### During the beta: what is collected

Only **`claude-fable-5`**, **`claude-fable-5-1`**, **`claude-opus-5`** and **`claude-opus-5-5`** coding sessions are collected and rewarded right now. If you are already on one of those, there is nothing to do. Otherwise switching takes one flag:

```bash
# One-off: launch Claude Code on a collected model
claude --model claude-opus-5-5

# Make it stick for every session (add to your shell profile)
export ANTHROPIC_MODEL=claude-opus-5-5
```

Inside a running session you can also type `/model claude-opus-5-5` (or `/model claude-opus-5`, `/model claude-fable-5` or `/model claude-fable-5-1`).

What happens to everything else:

* Sessions from any other model are **rejected**: they earn nothing and never enter anything we deliver. They appear as _rejected_ in your session list, you can remove them yourself at any time, and ones you leave are cleared under the beta retention policy.
* Rejected data is **not retroactively accepted** if the scope later widens — please don't keep uploading in the hope that it will be.
* Sessions waiting on manual review are kept until a decision is made.

Tip: sessions recorded with thinking effort set to `max` are more valuable to the current buyer. That is a recommendation, not a requirement — every effort level is accepted and rewarded the same.

Withdrawals are open during the beta. The rules on this page — collection scope, rates, limits, acceptance rules and the holding period — may change. A change applies from the moment it is published here, with the date at the top of this page; rewards already credited are never repriced. A new version of the [Beta Program Terms](https://trajector.publicai.io/legal/beta-terms) is a different matter: you have 30 days to accept it, after which uploads require it. Your balance can always be withdrawn either way.

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
* Embedded images and other base64 blobs are stripped before counting. You are paid for text that a human or a model actually wrote. Turning image and document upload off in the CLI (`upload_images_and_documents`) does not change your rewards.
* Tool results are counted **once**, even though agents re-send them on every subsequent request in the same conversation. This is a deliberate choice: counting them per request would inflate long sessions, and the count needs to mean the same thing for everyone.
* The count is computed identically on your machine and on the server, so what the CLI shows you and what you get paid for do not drift apart.

#### Calls the recording proxy didn't see earn 25% of the token rate

This page is the rule. The dashboard and the CLI state the conclusion and link here; when they differ from this page, this page is what you are paid by.

The rule applies from 17 September 2026. It changes the USD amount of a reward, never the token count.

**Definitions.**

* A _call_ is one request to the model and its response. Rewards are computed per call.
* The recording proxy **saw** a call when it handled that call's request and response on your machine. Every call of a session shown as _Recording proxy_ in **My data** was seen. In a session shown as _Proxy + local file_, each call counts on its own: the calls the proxy handled were seen, the calls it missed were not.
* The proxy **didn't see** a call when neither of two things happened: the proxy did not handle it, and the CLI did not record it live. _Recorded live_ means the CLI was following the session file while the session was still running, and the parts of it reached our servers while the work was still going on — our clocks, not yours, decide that. A call of a session backfilled from your local history, or uploaded in one go after the work was finished, was not seen.

**The rule.**

* A call the proxy saw earns the full token rate: `tokens × rate`.
* A call the proxy didn't see earns **25% of the token rate**: `tokens × rate × 0.25`.
* The tokens of a call the proxy didn't see are still counted in full — in **Total Tokens Contributed**, in the **Tokens** column of **My data**, and against the daily volume caps. Only the USD amount is reduced.
* Whether a session is accepted, held or rejected (§2) is decided separately from this rule. This rule only changes what an accepted call is worth.

**What you see.** Each ledger entry records whether the proxy saw the call it paid for and the multiplier applied, so every amount in your balance can be traced back to this rule. **My data** shows, per session, how many of its calls the proxy didn't see and the amount earned at 25%; each chain line shows which of its calls those were. On a day when some of your calls weren't seen, `trajector status` says so too.

**Not retroactive.** Calls credited before this rule took effect keep the amount they were credited at. The multiplier is written permanently into each ledger entry when the call is credited; a later change to the multiplier — in either direction — applies only to calls credited after that change, exactly as a change to the token rate does (§1). A call credited at 25% is not re-credited if the proxy later reports the same call.

**Why the proxy is the witness.** The proxy is the part of Trajector that handles a call as it happens. A session file, on its own, is written by Claude Code and read after the fact: nothing in it can be checked against a record we made ourselves. Paying the full rate for that would make a fabricated session file worth as much as a real recording.

**Why live recording counts too.** A session file that reaches us _while the session is still running_ carries one thing a file uploaded afterwards does not: the times its parts arrived at our servers, measured by our clocks. Work that is really happening arrives in step with itself — a morning of work cannot arrive in ten seconds. That arrival rhythm is ours, not yours, and it is what a live recording is paid on. Nothing you send us about when something happened is part of this: only when we received it.

This matters most if you cannot run the proxy at all. The Claude desktop app talks to Anthropic directly and ignores the settings the CLI uses, so there is no proxy to run — those sessions are not a lower tier of trust, they simply have no proxy available. Running Claude Code through the CLI's recording proxy — the default when the CLI is installed and enabled — still means the proxy sees every call, and is the simplest way to earn the full rate.

***

### 2. When a reward lands in your balance

Uploading a session is not the same as earning from it. Every uploaded session is checked on the server, and lands in one of three states.

#### What an accepted session needs

Besides being on a collected model (top of this page) and carrying reasoning summaries (below), a session needs all of these:

| Requirement             | Accepted when                                                                                                                                                                                       |
| ----------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| It changes code         | At least one code file is written — a source, configuration or build file — by `Edit`, `MultiEdit`, `Write` or `NotebookEdit`, or by a shell command that writes one. Prose such as `.md` or `.txt` and data such as `.csv` do not count |
| You asked for something | At least one prompt you typed                                                                                                                                                                       |
| The agent acted         | At least one tool call                                                                                                                                                                              |
| Model responses         | At least 5                                                                                                                                                                                          |
| Responses with thinking | At least 3                                                                                                                                                                                          |
| Failed tool calls       | At most 25% of tool results are errors                                                                                                                                                              |
| Scheduled prompts       | At most 25% of your prompts come from a scheduled task                                                                                                                                              |

Code writes are counted over the whole session, subagents included. The other requirements are checked on each conversation in the session — the main one and each subagent's — separately, and **My data** shows each on its own chain line.

#### ✅ Accepted — credited

The session passed every check. It is credited to your balance, and the session appears in **My data** with the reward it earned. The credit can be withdrawn 7 days later (§4).

**Daily volume caps.** There is a per-account and a per-device cap on tokens credited per UTC day. Tokens over the cap are not held and not lost: they are credited the next UTC day, and so on until all of them are credited.

#### ⏸️ Held — under review, not lost

Something stops the session being accepted for now. Held sessions earn nothing _yet_. If a reviewer accepts one, it is credited at that point exactly as if it had been accepted on arrival — nothing is lost by being held.

Common reasons a session is held:

* **Short of a requirement above.** A session that is only too short so far — too few responses, no code written yet — needs no reviewer: it is accepted on its own once it grows past the line. A session over the failed-tool-call or scheduled-prompt limit goes to a reviewer.
* **Exceptionally large sessions.** A session too large for the server to check yet is held, and is checked once the server can take it.
* **Fragments.** Content that is already fully contained in another session you uploaded.
* **Secrets or personal data found by the server's second-pass scan.** The CLI redacts locally before anything leaves your machine; the server scans again as a safety net. A hit sends the session to review rather than rejecting it, because the scan can also miss a new kind of secret the CLI does not mask yet — that is something we need to see. The session is released only if a reviewer finds the hit was a false positive. You can delete a held session yourself from **My data** at any time before you claim it.

#### ❌ Rejected — never credited

Rejection is reserved for fraud, for content we will never license, and for sessions without reasoning summaries; it is final:

* **Forged trajectories.** Sessions that were not produced by a real agent run.
* **The same content uploaded from more than one account.** One recording, one payment. Coordinated re-uploading across accounts is the thing this check exists to stop.
* **Sessions without reasoning summaries**, first uploaded from Oct 17, 2026. They carry no record of the model's reasoning, which is what the buyer pays for. The rule, and how to turn summaries on for each way you run Claude Code, are on the [Reasoning summaries](reasoning-summaries.md) page.

Everything else is held for review rather than rejected.

***

### 3. Reading your balance

The dashboard shows three numbers:

| Number               | Meaning                                                                                                     |
| -------------------- | ----------------------------------------------------------------------------------------------------------- |
| **Withdrawable Now** | What you can withdraw right now. Credits from the last 7 days are shown under it as _still maturing_, with the date they become withdrawable. Goes to $0 when you claim. |
| **Total Reward**     | Everything you have ever earned. Does **not** go down when you claim.                                       |
| **Total Tokens**     | Lifetime tokens credited across all accepted sessions, counted in full whether or not the proxy saw the call. |

Total Reward isn't simply Total Tokens × rate: tokens are counted in full for every credited call, while the USD amount is the full rate for calls the recording proxy saw and 25% of it for calls it didn't see (§1, _Calls the recording proxy didn't see_).

Below them you'll find a breakdown by agent type (Claude Code, Cursor, Codex, Gemini CLI, …) with tokens and session counts, and a daily contribution chart so you can see your contribution rhythm over time.

Every movement in your balance is recorded in an append-only ledger. Nothing is ever edited or deleted — a correction is posted as its own entry, so the history always adds up to the balance you see. The **Ledger** section of the Claim page lists it: credits are grouped into one line per UTC day, and every other entry has its own line with its date, type and amount. Entry types:

| Type       | Effect                                                                 |
| ---------- | ---------------------------------------------------------------------- |
| `accrual`  | + a session was credited                                               |
| `claim`    | − a payout was sent to you                                             |
| `fee`      | − the network fee on that payout                                       |
| `bonus`    | + a bonus, or a balance moved from another account of yours when accounts are merged, shown with its reason |
| `reversal` | ± a correction: a failed payout returning your balance, a deletion, a cancelled bonus, or an offset (below) |

#### If rewards have to be returned

If we conclude that a reward was obtained through abuse, or that you contributed work that was not yours to contribute, we email you the kind of problem and the facts we rely on, the sessions concerned and the amount. You have 14 days to reply to [support@publicai.io](mailto:support@publicai.io); someone who was not involved in the finding reviews your reply, and we answer in writing within 14 days; withdrawals are frozen meanwhile. Only after that is anything taken: an unwithdrawn reward is cancelled, and an amount already withdrawn is deducted from your balance and from later credits; if that is not enough, we may ask you to repay the rest. That deduction appears in the ledger as an offset with its reason. The full rule is §6 of the [Beta Program Terms](https://trajector.publicai.io/legal/beta-terms).

#### Why one conversation can appear as several "parts"

The CLI uploads in batches, so one long conversation may be flushed to the server in several slices. **My data** groups them back into one row per conversation and shows a `parts` count. Each slice carries its own distinct API calls and its own tokens, and **all of them are paid** — you are not paid only for the biggest slice.

***

### 4. Claiming your rewards

#### Requirements

* **Minimum $10.00**, counted on the withdrawable amount. Below that, the claim button is disabled and your balance keeps accumulating.
* **Holding period: 7 days.** Each credit becomes withdrawable 7 days after it was credited. A session credited after review counts from the moment it is credited. Bonuses are withdrawable at once. Credits made before the holding period took effect on October 27, 2026 are not subject to it. The holding period may change, but never to more than 30 days.
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
2. **Confirm.** The amount you claim leaves your balance the moment the claim is created — before the transaction is confirmed on chain. This is intentional: the money is reserved for your claim so it cannot be spent twice by a second request. Your **Total Reward** is unaffected.
3.  **Settlement.** The payout worker builds, signs, and broadcasts the transfer, then waits for on-chain confirmation. This takes anywhere from seconds to a few minutes.

    | State       | Meaning                                                                            |
    | ----------- | ---------------------------------------------------------------------------------- |
    | `pending`   | Created and queued or broadcast; waiting for confirmation                          |
    | `confirmed` | On chain. The transaction hash is recorded and shown to you                        |
    | `failed`    | Did not go through — **the full amount, fee included, is returned to your balance** |

**A failed payout never costs you anything.** The balance is restored in full and you can claim again. Claims never leave you with a negative balance, and a stuck transaction never silently eats a balance.

Claims are rate-limited per account. This protects the service, not your money — a rate-limited request is simply refused, never partially executed.

***

### 5. Deleting your data, and what it does to your rewards

You can browse every session you have uploaded under **My data**, including the exact redacted payload we hold — the same bytes a data buyer would receive.

**Sessions whose reward you have not yet withdrawn can be deleted yourself.** Deleting one blanks the stored payload and **reverses that session's reward out of your balance**. The exact amount to be reversed is shown in the confirmation dialog before you commit. This unwinds the exchange cleanly: we no longer hold the data, and we no longer owe you for it.

**Sessions whose reward has already been paid out cannot be deleted with a button.** That data is the consideration for a payment already sent on chain. Submit a deletion request instead: it goes to a review queue, where one admin reviews it and a second admin carries out the deletion, and then the payload is erased for real. **The money already paid is not clawed back** — your right to erasure does not depend on returning it.

Two things survive a deletion, deliberately:

* **A content fingerprint** of the session (a hash, not readable content). It carries the duty of preventing the same content being uploaded again for a second payout.
* **The ledger history.** Your accrual and its reversal both remain visible, so your balance is always reconstructible from the record.

***

### FAQ

**Do I get paid more for better sessions?** The token rate is the same for every accepted session. Separately, Trajector may periodically pay a quality bonus for sessions with substantial human direction and a verified result. How sessions are selected for it is not published.

**Can I use more than one account?** No. One person, one account. Pair all your machines with the same account. If we find more than one account of yours, we merge them: the balance moves to the account you keep, and the others are closed. Using extra accounts to earn more or to get around a limit is abuse, and its rewards are recovered.

**Does re-uploading the same session pay twice?** No. Each session is credited exactly once, and duplicate uploads of the same API calls are detected and not paid again.

**My session was held. What do I do?** Nothing. Held sessions sit in a review queue. If accepted, they are credited at that point.

**Can my rate change after I've contributed?** The rate can change going forward, but never for rewards already credited — the rate is locked into each ledger entry when it is written.

**Why can't I withdraw everything in my balance?** Credits from the last 7 days are still maturing. The dashboard shows the amount and the date it becomes withdrawable.

**I claimed and my balance went down, but the transaction isn't confirmed yet.** That's expected. The balance is reserved at the moment you claim. If the transaction ends up failing, the full amount — including the fee — returns to your balance automatically.

**Why is my Total Reward still showing an amount after I claimed?** Total Reward is lifetime earnings and never resets. Only the withdrawable amount goes down.
