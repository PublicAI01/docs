# Reading output

## Reading `status` and `doctor`

Two commands answer "is this thing working". `status` describes; `doctor` describes **and repairs**. Neither ever guesses.

### `status`, block by block

```
trajector 0.1.0

Device
  Signed in.

Project /path/to/project
  Contributing; recording is on for this project.

Proxy
  Running at 127.0.0.1:41100: version 0.1.0, up 4m12s.
  Recorded today: 3 (SSE degraded: 0, dropped: 0).

Spool
  184.2 KiB of 2.0 GiB used.

Uploads
  Last upload: 12 rawcall(s) (612.4 KiB) at 2026-08-10T08:41:02Z.
```

#### Device

| Line                                                 | Meaning                                                                |
| ---------------------------------------------------- | ---------------------------------------------------------------------- |
| "Signed in."                                         | The device is paired                                                   |
| "Not signed in. Run `trajector login`…"              | No device token                                                        |
| "WARNING: the device token store could not be read." | The keyring or the fallback files are unreadable — run `doctor`        |
| "Recording is paused everywhere: …"                  | Something suspended recording on every project, and the line says what |

#### Project

The heading is the project root `status` resolved from your current directory.

| Line                                                             | Meaning                                                |
| ---------------------------------------------------------------- | ------------------------------------------------------ |
| "Contributing; recording is on for this project."                | Injection and routing table agree                      |
| "Not enabled. Run `trajector enable`…"                           | Nothing is injected and nothing is granted             |
| "WARNING: the injected settings and the routing table disagree." | The two records of consent do not match — run `doctor` |

Two extra lines appear when they apply:

```
  Upstream: https://your-relay.example (third-party origin).
  The upstream moved from https://old-relay.example at 2026-08-10T08:12:00Z (base-URL configuration change).
```

The first shows whenever traffic goes somewhere other than the official endpoint. The second appears after trajector noticed your base-URL configuration change and followed it.

#### Proxy

| Line                                                      | Meaning                                                      |
| --------------------------------------------------------- | ------------------------------------------------------------ |
| "Running at …: version X, up Y."                          | Our proxy holds the port                                     |
| "Not running; it starts on demand with the next session." | Normal when you are not in a session — it is not a daemon    |
| "WARNING: …"                                              | Something else holds the port, or ours could not be verified |

{% hint style="warning" %}
**`Recorded today` counts from when the proxy started, not from midnight.** The proxy exits after 30 minutes of silence and is restarted by the next session, so this number resets more often than the label suggests. A low number here does not mean recording is broken — check `Spool` and `Uploads`.
{% endhint %}

`SSE degraded` counts streamed responses that could not be reassembled into the equivalent non-streaming object; those records are kept as raw stream text and marked, not dropped. `dropped` counts captures that were abandoned — the forwarded request was unaffected either way.

#### Spool

Usage against a 2 GiB quota. When the quota is reached, new captures are dropped rather than evicting what is already there, and the line tells you to upload.

#### Uploads

Last successful upload, last error if there was one, and a warning if any batch is quarantined:

```
  WARNING: 3 rawcall(s) in 1 rejected batch(es) are quarantined and will not be retried automatically.
  Run `trajector doctor` to inspect them, then requeue or discard them.
```

#### Service

This block only appears when the service said something on the last upload — a required version, a message, or a notice.

### `doctor`, line by line

```
trajector 0.1.0 doctor

  ok: proxy running at 127.0.0.1:41100 (version 0.1.0, up 4m12s)
  ok: injection and routing agree for this project
  ok: capture spool writable (184.2 KiB of 2.0 GiB used)
  ok: no rejected batches quarantined
  ok: live proxy confirms this project routes and records

Everything checks out.
```

| Prefix     | Meaning                        | Your move                   |
| ---------- | ------------------------------ | --------------------------- |
| `ok:`      | Checked, nothing wrong         | Nothing                     |
| `fixed:`   | Was wrong, repaired            | Nothing, but worth reading  |
| `problem:` | Wrong, and a decision is yours | Read the line; it says what |

Exit is `1` when any `problem:` remains, so `doctor` works in a script.

#### What it repairs on its own

* Starts the proxy, or replaces a proxy from an older build.
* Rewrites a damaged injection: token and session hooks restored.
* Removes a stale injection whose token no longer records.
* Follows a base-URL configuration change to a new upstream.
* Re-adds the project-discovery hint to your user settings.
* Adds the missing rule to `.gitignore`.

#### What it hands back to you

* Both consent records exist but disagree about the project's identity.
* Something else holds `127.0.0.1:41100`.
* `ANTHROPIC_BEDROCK` / `ANTHROPIC_VERTEX` is set — those channels are not supported, so the project is not being captured.
* A base URL moved to a plaintext non-loopback address, which is refused: a non-loopback upstream must use `https`.
* The spool is unwritable or full.
* Batches are quarantined and you have to choose `requeue` or `discard`.

#### Environment notes

On WSL and Windows `doctor` adds an `ok:` line about the topology:

```
  ok: running under WSL: Claude Code must also run inside WSL; a Windows-native claude cannot reach this trajector
  ok: Windows: if a firewall prompt appears for the proxy, allow loopback access; the proxy only ever binds 127.0.0.1
```

These are notes, not problems.
