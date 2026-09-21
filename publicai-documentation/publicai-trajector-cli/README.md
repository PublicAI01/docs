---
description: >-
  Turn your own Claude Code sessions into compensated data contributions, one  
  project at a  time, with nothing captured until you say so.
---

# 🛰️ PublicAI Trajector CLI

## Trajector CLI

Trajector is an open-source command-line tool. With your explicit per-project consent it records that project's coding sessions from three sources on your own machine — the API traffic it routes through a local proxy, the session files Claude Code writes, and the state of the project's git repository — masks secrets locally, and uploads only the redacted result.

Nothing is captured until you opt a project in, and every project you have not opted in is structurally incapable of being captured — not by policy, but because the code path does not exist for it.

{% hint style="info" %}
**Status: beta.** Releases before `1.0.0` are published as pre-releases. macOS and Linux are supported; there is no Windows build yet (run it under WSL).
{% endhint %}

### What it does

`trajector enable`, run inside a project, writes project-local Claude Code settings that point that project's API traffic at a local reverse proxy bound to `127.0.0.1:41100`, and installs session hooks. The proxy forwards every request verbatim to the configured upstream and records it on the side; the hooks let trajector read the project's session files while a session runs and observe the project's git state. `trajector enable --no-proxy` installs the hooks alone, recording from session files and git only and leaving `/remote-control` available inside the project. `trajector enable --no-earlier` leaves the session files the project already has alone, so only the sessions that run from now on are collected; it stands beside `--no-proxy`, and neither flag implies the other.

Upgrading to 0.3.3 moves the data agreement to version `2026-09-21`, so the next `trajector enable` shows it and asks you to confirm it once.

* **Forwarding is sacred.** Any failure on the recording side — disk full, malformed stream, internal error — never interrupts forwarding. Streaming responses pass through unbuffered.
* **Recording is unobservable.** What the proxy sends upstream does not depend on whether the exchange is being recorded.
* **Consent is the routing key.** A per-project token decides what may be recorded. Requests without a valid token are forwarded and never recorded.
* **Credentials never touch disk.** `Authorization` and `x-api-key` are not written to any file, in any state.
* **Consent is revocable.** `disable`, `logout`, and `uninstall` each undo a different amount, immediately.

The proxy starts on demand, exits when idle, and is never a permanent daemon.

### The shape of a session

```
claude  ──►  127.0.0.1:41100  ──►  api.anthropic.com
                    │                (or your own relay)
                    └──►  spool  ──►  redact  ──►  upload
                         ▲
  session files  ────────┤   (read while the session runs)
  git state      ────────┘   (paths and identifiers only)
```

Records from all three sources wait in a local spool with a bounded disk quota. Secrets are masked on your machine, and only redacted batches are uploaded. An upload is only considered done once the service acknowledges it by batch id.

### Where to go next

| If you want to                          | Read                          |
| --------------------------------------- | ----------------------------- |
| Install it                              | Installation                  |
| Get to a first recorded session         | Quickstart                    |
| Look up a flag or an exit code          | Command reference             |
| Understand what `status` is telling you | Reading `status` and `doctor` |
| Know exactly what is collected          | Data and privacy              |
| Fix something                           | Troubleshooting               |

The client is fully open source at [PublicAI01/trajector-cli](https://github.com/PublicAI01/trajector-cli); every statement in these pages can be checked against the code.
