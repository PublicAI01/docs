# Data and privacy

The client is fully open source. Every statement on this page can be checked against the code in [PublicAI01/trajector-cli](https://github.com/PublicAI01/trajector-cli), and the authoritative version lives in that repository's `PRIVACY.md`.

### What is collected

**Nothing, until you opt a project in.** `trajector enable` shows the data agreement and requires an explicit yes. Only then does that project's traffic flow through the local proxy.

For an enabled project, the proxy records **successful `POST /v1/messages` exchanges, verbatim**:

* the full request — system prompt, tools, messages, thinking configuration;
* the full response — content, thinking signatures, usage;
* the HTTP status, timestamps, API version headers;
* a **hash** of the project's root path.

Streamed responses are reassembled into the equivalent non-streaming object. When reassembly fails, the raw stream text is kept and the record is marked `garbled` rather than dropped or repaired.

Records are observed facts and are never rewritten. Model identifiers, thinking signatures, and usage figures stay exactly as the API produced them.

### What is never collected

| Not collected               | Why you can rely on it                                                                                                       |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Projects you did not enable | Only requests carrying an enabled project's token are recorded. Recording anything else is not a code path that exists       |
| Credentials                 | `Authorization`, `x-api-key`, and other credential headers are never written to disk, in any file, in any state              |
| Project paths               | Records identify a project only by a hash of its root path                                                                   |
| Telemetry                   | There is no separate reporting channel. A few counters ride inside uploads you already consented to — no upload, no counters |
| Diagnostics                 | `doctor bundle` writes an archive **you** inspect and attach. Nothing reports home on its own                                |

### On your machine

Recorded calls wait in a local spool, in files readable only by your user account (`0600`/`0700`), under a 2 GiB quota. A full spool stops recording; it never evicts what is already captured.

Before anything is uploaded, records pass a **local redaction pass** that masks secrets — API keys, tokens, passwords, credential-shaped strings — and personally identifying strings such as email addresses and phone numbers, while preserving JSON structure, message order, tool-call pairing, and thinking signatures.

**Unredacted data never leaves your machine.**

{% hint style="info" %}
**Known limitation:** masking applies to values only. A secret placed in a JSON _key_ position is not masked, because keys are structure and the pass never rewrites structure.
{% endhint %}

### What leaves your machine

Redacted records are packed into compressed batches — by default when 10 MiB or 24 hours accumulate — and uploaded over HTTPS, authenticated by your device pairing token.

Each batch carries a client-side idempotency key, so a retried upload can never be counted twice. Local records are deleted **only after** the service acknowledges the batch by echoing that key. Any other answer leaves your data in place for retry.

The upload destination can be changed only through `config.json` in your user config directory, never through an environment variable — see Files, configuration, and environment. A non-default destination is announced at `enable` and at proxy start; it is never silent.

### What happens to it

Contributed data is sold to third-party buyers and contributors are compensated. That fact is public by design; buyer identity and commercial terms are confidential, the sale itself is not.

Records captured through a third-party base URL — a relay you configured yourself — are labelled third-party origin. Reward terms are the same regardless of origin.

### Revoking consent and deleting data

| Command                               | What it undoes                                                                                                                 |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `trajector disable`                   | Removes the injection, revokes the project token, deletes that project's local unuploaded records — spool and quarantine alike |
| `trajector disable --purge`           | The above, plus a request to delete that project's uploaded but not yet delivered data                                         |
| `trajector doctor discard <batch-id>` | Deletes a quarantined batch from this machine for good. Local only — nothing already uploaded is affected                      |
| `trajector logout`                    | Revokes the device token and pauses recording everywhere. Forwarding is unaffected; logging in resumes                         |
| `trajector uninstall`                 | Removes every injection and stops the proxy; `--delete-data` also deletes all local data                                       |

Data already delivered and compensated is licensed under the agreement you accepted at enable time. Deletion requests reach everything before that point.

### Reporting a vulnerability

Through the repository's private vulnerability reporting, as described in [SECURITY.md](https://github.com/PublicAI01/trajector-cli/blob/main/SECURITY.md).
