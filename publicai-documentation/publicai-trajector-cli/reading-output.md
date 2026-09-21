# Reading output

## Reading `status` and `doctor`

Two commands answer "is this thing working". `status` describes; `doctor` describes **and repairs**. Neither ever guesses.

### How a line reads

Both commands read from one scale, so a line means the same thing wherever it appears.

**The first line is the verdict.** `status` opens with it, before the version and the blocks, and `enable` ends with the same line:

```
Recording: on (3 project(s))
```

The other three readings are `Recording: PAUSED on this device`, `Recording: STOPPED on this device (spool full)`, and `Recording: off in this project`.

**Every marked line opens with its severity.**

| Prefix     | On a terminal   | Meaning                                  |
| ---------- | --------------- | ---------------------------------------- |
| `error:`   | red, `✗`        | Wrong, and a decision is yours           |
| `warning:` | yellow, `!`     | Worth reading; recording continues       |
| `fixed:`   | green, `✓`      | Was wrong, and `doctor` repaired it      |
| `ok:`      | green, `✓`      | Checked, nothing wrong                   |
| `note:`    | no glyph, no colour | A fact that asks nothing of you      |

`doctor` marks every line it prints. `status` marks only what is wrong — `error:` or `warning:` — and leaves the rest of its lines unmarked, so a screen with nothing wrong has no marked line on it. Within a `status` block, what is broken is printed first.

**A problem is three lines**: what stopped, why, and the one command that ends it, on a line of its own so it can be copied whole.

```
  error: Recording is paused everywhere
    why:  the consent record at /home/you/.config/trajector/consent.json could not be read (unexpected end of JSON input)
    fix:  trajector enable
```

The `fix:` line carries a command and nothing else. Where nothing trajector runs can end it — a port another process holds, for instance — there is no `fix:` line, and what to do is stated under the problem instead.

**Colour and glyphs are for terminals only.** Colour is used only when a terminal is reading, and the glyphs `✓ ✗ !` only where the locale states UTF-8; elsewhere they are `+ x !`. `NO_COLOR`, `TERM=dumb`, and the global `--no-color` flag each turn the colour off and keep the glyphs. Output to a pipe or a file is plain ASCII with no escape sequences, so it can be read by a script or pasted into a report.

**Exit codes.** `status` exits `1` when it printed anything at error severity, `0` otherwise. `doctor` exits `1` when an `error:` is left over after its repairs.

### `status`, block by block

```
Recording: on (3 project(s))
trajector 0.3.3

Device
  Signed in.

Project /path/to/project
  Contributing; recording is on for this project.

Proxy
  Running at 127.0.0.1:41100: version 0.3.3, up 4m12s.
  Recorded since it started: 3 (SSE degraded: 0, dropped: 0).

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
| "warning: the device token store could not be read." | The keyring or the fallback files are unreadable — run `doctor`        |
| "Recording is paused everywhere: …"                  | Something suspended recording on every project, and the line says what |

#### Project

The heading is the project root `status` resolved from your current directory.

| Line                                                             | Meaning                                                |
| ---------------------------------------------------------------- | ------------------------------------------------------ |
| "Contributing; recording is on for this project."                | Injection and routing table agree                      |
| "Not enabled. Run `trajector enable`…"                           | Nothing is injected and nothing is granted             |
| "warning: the injected settings and the routing table disagree." | The two records of consent do not match — run `doctor` |

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
| "error: another process holds the proxy port…"            | Something else is on the port — see Troubleshooting          |
| "error: …could not confirm…"                              | Ours could not be verified; usually a stale admin token      |

{% hint style="warning" %}
**`Recorded since it started` counts from when the proxy started, not from midnight.** The proxy exits after 30 minutes with neither traffic forwarded nor a session file gaining lines — it waits two hours instead while a session's process is still running — and is restarted by the next session, so this number resets more often than the label suggests. It counts what the proxy recorded only, not what was read from session files. A low number here does not mean recording is broken — check `Spool` and `Uploads`.
{% endhint %}

`SSE degraded` counts streamed responses that could not be reassembled into the equivalent non-streaming object; those records are kept as raw stream text and marked, not dropped. `dropped` counts captures that were abandoned — the forwarded request was unaffected either way.

#### Session files

Per enabled project, `status` also reports the second source: how many sessions are registered, when one was last read, and how much is not read yet, followed by the standing notice that sessions started under another spelling of the project's path are not collected.

```
  Session files: 4 session(s) registered; last read 2026-09-20T09:12:41Z; 12.4 KiB not read yet.
```

`Session files: none registered yet.` means no hook has reported one. No line here names a session or a session file.

A project enabled with `--no-earlier` also carries the way back to its earlier files:

```
  Earlier sessions were skipped at enable; run `trajector enable` again (with --no-proxy if you use it) to collect them.
```

#### Spool

Usage against a 2 GiB quota. When the quota is reached, new captures are dropped rather than evicting what is already there, and the line tells you to upload.

A second line appears when this build met a segment whose shape its redaction does not cover and kept that segment back:

```
  12 segment(s) from 2 session(s) (48.1 KiB) are held on this machine because their shape is new to this build; they are not uploaded.
```

Held segments sit outside the quota, nothing uploads them, reading went on from the next segment, and recording was not paused. `trajector doctor` moves them back for upload once a build reads them cleanly; `trajector forget <session-id>` deletes what is held for a session.

#### Uploads

Last successful upload, last error if there was one, and a warning if any batch is quarantined:

```
  warning: 3 rawcall(s) in 1 rejected batch(es) are quarantined and will not be retried automatically.
  Run `trajector doctor` to inspect them, then requeue or discard them.
```

A standing refusal from the service is stated here too, with when uploads stopped:

```
  error: Uploads are paused since 2026-09-20T14:32:00Z until 2026-09-20T14:47:00Z: the service refused this client access. Captured data is kept.
  error: Uploads are paused since 2026-09-20T14:32:00Z: the service refused this device's credential. Captured data is kept.
```

The first retries itself and names when the next attempt is due; the second waits for `trajector login`. Only one of the two stands at a time.

#### Service

This block only appears when the service said something on the last upload — a required version, a message, or a notice.

### `doctor`, line by line

```
trajector 0.3.3 doctor

  ok: proxy running at 127.0.0.1:41100 (version 0.3.3, up 4m12s)
  ok: injection and routing agree for this project
  ok: every session file of this project is registered (4 session(s))
  ok: capture spool writable (184.2 KiB of 2.0 GiB used)
  ok: no rejected batches quarantined
  ok: live proxy confirms this project routes and records

Everything checks out.
```

Every line carries one of the five prefixes above. Exit is `1` when any `error:` remains, so `doctor` works in a script.

#### What it repairs on its own

* Starts the proxy, or replaces a proxy from an older build — including a hung proxy of ours that holds the port and answers nothing.
* Rewrites a damaged injection: token and session hooks restored — including a session hook missing from an injection made before that hook existed.
* Lifts a device-wide recording pause once **this** build has read the session files again and found nothing it cannot redact: `fixed: recording resumed: this build read the session files again and found nothing it cannot redact`. A pause an older build set is still lifted after an upgrade.
* Asks for session files this device registered but never read: `fixed: asked for 2 never-read session file(s) of this project to be read`.
* Registers session files no hook reported, and asks for those too: `fixed: registered 1 session file(s) of this project that no hook reported, and asked for them to be read`.
* Moves a held segment back for upload once this build reads it cleanly.
* Removes a stale injection whose token no longer records.
* Follows a base-URL configuration change to a new upstream.
* Re-adds the project-discovery hint to your user settings.
* Adds the missing rule to `.gitignore`.

#### What it hands back to you

* Both consent records exist but disagree about the project's identity.
* `ANTHROPIC_BEDROCK` / `ANTHROPIC_VERTEX` is set — those channels are not supported, so the project is not being captured.
* A base URL moved to a plaintext non-loopback address, which is refused: a non-loopback upstream must use `https`.
* The spool is unwritable or full.
* Batches are quarantined and you have to choose `requeue` or `discard`.
* Sessions of the project were written after its grant that no hook reported, and nothing readable on this machine explains why: `warning: could not determine why the hooks did not report 1 session(s) of this project`, with the causes it cannot check listed under it. Sessions written *before* the grant are a `note:`, not a problem, and `doctor` does not exit `1` over them.
* Another process holds `127.0.0.1:41100` and answers nothing — the line names the command that names the holder on this platform, and `trajector doctor` as the step after the port is free.
* A settings layer or managed settings keep Claude Code from loading trajector's hooks; the line names the layer that decided it, not a file path.

#### Environment notes

On WSL and Windows `doctor` adds an `ok:` line about the topology:

```
  ok: running under WSL: Claude Code must also run inside WSL; a Windows-native claude cannot reach this trajector
  ok: Windows: if a firewall prompt appears for the proxy, allow loopback access; the proxy only ever binds 127.0.0.1
```

These are notes, not problems.
