# Data and privacy

The client is fully open source. Every statement on this page can be checked against the code in [PublicAI01/trajector-cli](https://github.com/PublicAI01/trajector-cli), and the authoritative version lives in that repository's `PRIVACY.md`.

This page describes version **`2026-09-21`** of the data agreement — the version `trajector enable` shows you in full and records when you accept it. The agreement and this page state the same terms and change together; the version is the day the build carrying those terms was released.

### What is collected

**Nothing, until you opt a project in.** `trajector enable` shows the data agreement and requires an explicit yes. Only after that is anything collected from the project, and it is collected from three sources: the project's Claude Code API traffic, which flows through the local proxy; the session files Claude Code writes on your machine for that project; and the state of that project's git repository at a few moments during a session.

#### From the proxy

For an enabled project, the proxy records **successful `POST /v1/messages` exchanges, verbatim**:

* the full request — system prompt, tools, messages, thinking configuration;
* the full response — content, thinking signatures, usage;
* the HTTP status, timestamps, API version headers;
* a **hash** of the project's root path.

Streamed responses are reassembled into the equivalent non-streaming object. When reassembly fails, the raw stream text is kept and the record is marked `garbled` rather than dropped or repaired.

#### From the session files

Claude Code writes a file for every session it runs in a project. For an enabled project, trajector reads those files and records their lines as Claude Code wrote them, while the session runs: a session hook reports each turn to the resident process, which reads the file then, and looks at the files of running sessions on its own every few minutes in case a hook was missed.

The session files contain what the proxy cannot see:

* your tool results, including the contents of the files that were read and the changes that were written;
* your working directory and git branch;
* the full conversations of subagents.

Files are read only for projects you enabled. Trajector never lists the folder where Claude Code keeps session files: the paths it opens are derived from the enabled projects' own paths, so the paths to other projects' session files are never constructed and those files are never opened.

**Sessions from before you enabled the project.** When you enable a project, trajector also collects, once, the session files that project had already written before that moment. `trajector enable` tells you how many there are and how old the oldest one is, and reads them in the background right away. After that, trajector does not scan backwards again unless you ask it to.

`trajector enable --no-earlier` leaves them alone: those files are then never registered, and nothing reads them. Only the sessions that run from then on are collected. `status` says so under the project, and names the way back.

#### From git

When a session starts, when it ends, and after a shell command that makes a commit, trajector runs `git` in the enabled project and records what it prints:

* the commit that is checked out, and its first parent;
* the branch name;
* the list of paths that changed between two commits, together with the blob identifiers git prints beside them.

**Only paths and identifiers are recorded — never the content of any file.** No `git` command trajector runs writes anything to your repository. Each such record also says how far inside the project the session was running, never the absolute directory.

Records are observed facts and are never rewritten. Model identifiers, thinking signatures, and usage figures stay exactly as the API produced them, session lines stay exactly as Claude Code wrote them, and what `git` prints is copied as printed.

### What is never collected

| Not collected               | Why you can rely on it                                                                                                       |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Projects you did not enable | Consent is enforced structurally on every source. Only requests carrying an enabled project's token are recorded; the paths to other projects' session files are never constructed, so those files are never opened |
| Credentials                 | `Authorization`, `x-api-key`, and other credential headers are never written to disk, in any file, in any state              |
| Project paths               | Stored records and uploads identify a project only by a hash of its root path. In session records, the few fields whose value is the directory the session ran in are masked before upload; every other path is kept as observed |
| File contents, from git     | Git observations carry paths and identifiers only — never the content of any file, and nothing is written to your repository |
| Telemetry                   | There is no separate reporting channel. A few counters ride inside uploads you already consented to — no upload, no counters |
| Diagnostics                 | `doctor bundle` writes an archive **you** inspect and attach. Nothing reports home on its own                                |

### On your machine

Recorded calls, recorded session lines, and recorded git observations wait in a local spool, in files readable only by your user account (`0600`/`0700`), under a 2 GiB quota. A full spool stops recording; it never evicts what is already captured. Trajector only reads the session files Claude Code writes; it never writes to them.

Before anything is uploaded, records from every source pass the **same local redaction pass**. It masks secrets — API keys, tokens, passwords, credential-shaped strings — and personally identifying strings such as email addresses and phone numbers, while preserving JSON structure, message order, tool-call pairing, and thinking signatures.

#### Masked, or uploaded as observed

| Masked before upload                                                                                     | Uploaded as observed                                                                                          |
| -------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------- |
| Secrets and personally identifying strings, in records from every source                                 | Model identifiers, thinking signatures, usage figures                                                         |
| The few session-record fields whose value is, by construction, the directory the session ran in — `cwd` is the one you will recognize — replaced with a placeholder | Every other path a record holds, including the working directory stated in the environment description Claude Code writes at the start of a session, and any file your messages and tool results name |
| In a git record, the branch name and the changed paths                                                   | In a git record, the commit and blob identifiers, exactly as git printed them                                 |

Trajector does not rewrite an observation, because rewriting it would destroy the record.

**Unredacted data never leaves your machine.** The raw copy stays on the machine it was recorded on.

#### Segments held on this machine

A session file line whose shape this build's redaction does not cover no longer stops anything. That one segment is held on your machine, in a place nothing uploads from, and reading goes on from the next segment. Nothing unmasked leaves the machine, and nothing else is delayed by it.

`trajector status` and `trajector doctor` say how many segments are held, for how many sessions, and how much disk they take:

```
  12 segment(s) from 2 session(s) (48.1 KiB) are held on this machine because their shape is new to this build; they are not uploaded.
```

Held segments do not count against the spool quota. A later build that reads a held segment cleanly moves it back for upload the next time you run `trajector doctor`, and `trajector forget <session-id>` deletes what is held for that session.

#### The device-wide pause

Recording still pauses on the whole device when a read contradicts itself — an incomplete line, where nothing the read produced can be trusted. `trajector status` names the reason, `trajector upgrade` tells you the next step while the pause stands, and `trajector doctor` lifts the pause once **this** build has read the session files again and found nothing it cannot redact. It no longer takes a different build; a pause an older build set is still lifted after the upgrade, as before.

A consent record that cannot be read or parsed also pauses recording device-wide, under a reason of its own. `status`, `doctor` and the diagnostic bundle name the file and the failure, and `trajector enable` is the way out: accepting the agreement writes a new record and recording resumes.

`trajector logout` pauses recording device-wide in the same way; forwarding is unaffected in every case, and logging in again resumes.

{% hint style="info" %}
**Known limitation:** masking applies to values only. A secret placed in a JSON _key_ position is not masked, because keys are structure and the pass never rewrites structure.
{% endhint %}

### What leaves your machine

Redacted records of all three kinds — recorded API calls, recorded session lines, and recorded git observations — are packed into compressed batches, by default when 10 MiB or 24 hours accumulate, and uploaded over HTTPS, authenticated by your device pairing token. Records read from session files leave on thresholds of their own — 1 MiB or five minutes, adjustable by the service handshake — and at once when a session ends or its process is gone and its file stopped growing.

Each batch carries a client-side idempotency key, so a retried upload can never be counted twice. Local records are deleted **only after** the service acknowledges the batch by echoing that key. Any other answer leaves your data in place for retry.

The upload destination can be changed only through `config.json` in your user config directory, never through an environment variable — see Files, configuration, and environment. A non-default destination is announced at `enable` and at proxy start; it is never silent.

### What happens to it

Contributed data is sold to third-party buyers and contributors are compensated. That fact is public by design; buyer identity and commercial terms are confidential, the sale itself is not.

Records captured through a third-party base URL — a relay you configured yourself — are labelled third-party origin. Reward terms are the same regardless of origin.

What a record is worth does depend on whether the local proxy witnessed the call it is of. A call is witnessed when the proxy handled its request and its response on your machine. A call trajector read from the session file while the session was still running — which is how a project enabled with `--no-proxy` is recorded — may also count as witnessed. The sessions a project had before you enabled it, and any session whose file was read only after it ended, are not witnessed. Calls that were not witnessed are rewarded at a lower rate than witnessed ones; the tokens are counted in full either way, and only the amount is reduced. The current rates are published on the Rewards page and are not fixed by the client.

### Revoking consent and deleting data

| Command                               | What it undoes                                                                                                                 |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------ |
| `trajector disable`                   | Removes the injection, revokes the project token, deletes that project's local unuploaded records — spool and quarantine alike |
| `trajector disable --purge`           | The above, plus a request to delete that project's uploaded but not yet delivered data                                         |
| `trajector doctor discard <batch-id>` | Deletes a quarantined batch from this machine for good. Local only — nothing already uploaded is affected                      |
| `trajector logout`                    | Revokes the device token and pauses recording everywhere. Forwarding is unaffected; logging in resumes                         |
| `trajector forget [session-id]`       | Deletes one session's not-yet-uploaded records from this machine — both spool slots and any quarantined batch. Defaults to the current session |
| `trajector uninstall`                 | Removes every injection and stops the proxy; `--delete-data` also deletes all local data                                       |

Every deletion above applies to all three kinds of records in the same way, and reaches trajector's own copies only: the session files Claude Code writes on your machine are yours, and trajector never modifies or deletes them — not on `disable`, not on `uninstall`.

Data already delivered and compensated is licensed under the agreement you accepted at enable time. Deletion requests reach everything before that point.

### Reporting a vulnerability

Through the repository's private vulnerability reporting, as described in [SECURITY.md](https://github.com/PublicAI01/trajector-cli/blob/main/SECURITY.md).
