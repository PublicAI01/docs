# Troubleshooting

**Start here:**

```sh
trajector doctor
```

It checks everything below, repairs what it can on its own, and tells you what it cannot. Most of this page is only needed when `doctor` hands something back to you.

***

### Nothing is being recorded

Start with the first line of `trajector status`. It is the verdict — `Recording: on (3 project(s))`, `Recording: PAUSED on this device`, `Recording: STOPPED on this device (spool full)`, or `Recording: off in this project` — and it says which of the steps below applies. `status` exits `1` when it printed an error, so a script can ask the same question.

Then work through these in order:

1. **`trajector status` in the project.** Does the Project block say `Contributing`? If it says `Not enabled`, that project was never opted in — `trajector enable`.
2. **Is the device signed in?** `Not signed in` in the Device block pauses recording everywhere. Forwarding still works, which is why the session feels normal.
3. **Is the agreement current?** If it changed, recording pauses until you reconfirm with `trajector enable`. `status` prints `error: Recording is paused everywhere` with the reason on the `why:` line under it and the command that ends it on the `fix:` line. An unreadable consent record pauses the same way, under its own reason naming the file — `trajector enable` is the way out of that one too.
4. **Is Claude Code actually using the injected settings?** The injection is written to the project-local settings file. If you launched `claude` from a different directory, that project is not the one you enabled.
5. **`trajector doctor`.** The `live proxy confirms this project routes and records` line is an end-to-end check, not a guess.

While a pause stands or the spool is full, the first session hook of a session says so in the session itself, once per session:

```
trajector: nothing of this session is being recorded; run trajector status
```

Claude Code shows it as a non-blocking hook notice — in desktop Claude Code it appears once in the transcript — and nothing about the session changes. Run `trajector status` and the verdict line names the reason.

{% hint style="info" %}
`Recorded since it started: 0` on its own is not evidence. That counter resets whenever the proxy restarts, and the proxy exits after 30 minutes with neither traffic forwarded nor a session file gaining lines — it waits two hours instead while a session's process is still running. It also counts only what the proxy recorded, not what was read from session files. Look at `Spool` and `Uploads` instead.
{% endhint %}

### Desktop Claude Code: closing the tab does not end the session

Closing the tab or the window does not tell trajector the session is over. A session that was recorded live settles either when a session-end hook arrives, or 30 minutes after its last segment — whichever comes first. Until it settles it is not priced, so a session you just closed may show nothing for up to half an hour. There is nothing to do but wait.

Desktop Claude Code sessions never go through the recording proxy — the host sets its own API base URL — and are recorded from their session files instead.

### `A session hook is missing from …`

```
A session hook is missing from /path/to/project/.claude/settings.local.json; run `trajector doctor` to add it.
```

An injection made by an older build lacks a hook a later build installs. `trajector doctor` adds it:

```
fixed: rewrote the injection in /path/to/project/.claude/settings.local.json (token and session hooks restored)
```

`doctor` reads the project's recording shape from the grant written at enable time, so the repair keeps the shape you chose — with or without `--no-proxy`.

### Claude Code will not load the hooks

`enable`, `status` and `doctor` state what a static reading of Claude Code's configuration says:

```
Judged from configuration readable on this machine, Claude Code will not load trajector's hooks in this
project (...)
```

The reason is in the parentheses. What to check:

* **Hooks disabled in a settings layer**, or **managed settings** deciding it. Change the setting where it is set, or ask whoever manages it to. The reason names the settings layer that decided it, not a file path.
* **The workspace is not trusted yet.** Claude Code runs a project's hooks only after you accept the trust dialog, and no file records that answer, so no command of trajector's can confirm it. It is one of the causes listed under the next entry.
* **The project is on a Windows drive mounted into WSL.** Claude Code then runs on the Windows side and its hooks cannot reach this trajector. Run both on the same side.

With a base URL injected, the proxy keeps recording while the hooks do not load; the session files are simply not read. With `--no-proxy` the hooks are the only source, so nothing is recorded from the project until they load.

### `doctor` found session files no hook reported

`doctor` walks the project's session files and answers with the distinction it can make, never with a guess.

**Sessions written before the project was enabled** are named as such and registered on the spot:

```
  note: 3 session(s) of this project predate its grant, so no hook of trajector's reported them
      Run `trajector enable` in this project (with --no-proxy if you use it) to register them.
```

That is a note, not a problem, and `doctor` does not exit `1` over it. From 0.3.3, `enable` and `doctor` also ask for those files to be read right away instead of waiting for a session hook to name them — earlier builds registered them and then never read them. A project enabled with `--no-earlier` is left alone here by design.

**Sessions written after the grant that no hook reported** are reported as the open question they are:

```
  warning: could not determine why the hooks did not report 1 session(s) of this project
      Claude Code runs a project's hooks only once the workspace is trusted, and that answer is given in a dialog no file records.
      A session started with `claude --bare`, or with `--settings` naming another file, loads other settings than the ones this project carries.
      A setting in the environment of the session's own process is not readable from here either; under WSL, so are managed settings on the Windows side.
```

None of those three can be confirmed from this machine, which is why the headline states the question rather than an answer. Work through them yourself: accept the trust dialog, start sessions without `--bare` or a `--settings` file of their own, and check the environment the session's own process runs in. Older builds answered this case with `This workspace is not trusted yet`, which was one cause stated as if it were the finding.

### Segments held on this machine

Where an older build paused recording everywhere with `recording paused: session records changed shape`, this one holds back the one segment and carries on. A session file line whose shape this build's redaction does not cover no longer stops recording anywhere:

```
  12 segment(s) from 2 session(s) (48.1 KiB) are held on this machine because their shape is new to this build; they are not uploaded.
```

That one segment is held on your machine where nothing uploads from, reading goes on from the next segment, and the rest of the session is recorded and uploaded as usual. Held segments sit outside the spool quota.

What to do: `trajector upgrade`, then `trajector doctor`. A build that reads a held segment cleanly moves it back for upload when `doctor` runs, and reports it:

```
  fixed: 12 held segment(s) read cleanly under this build and will be uploaded
```

If you would rather it never left your machine, `trajector forget <session-id>` deletes what is held for that session.

Recording still pauses on the whole device when a read contradicts itself — an incomplete line, where nothing that read produced can be trusted. `trajector doctor` lifts that pause once **this** build has read the session files again and found nothing it cannot redact; it no longer takes a different build, and a pause an older build set is still lifted after an upgrade.

### Sessions started under another spelling of the project path

```
Sessions started under another spelling of this project's path, or under a directory name Claude Code was
told to use instead, are stored under names this device does not compute and are not collected.
```

Claude Code stores a session's files under a name derived from the path the session was started in. A session started through a symlink, a different case, or a directory name Claude Code was told to use instead lands under a name trajector does not compute from the project root it enabled, and those files are never opened. Start sessions from the same path you ran `trajector enable` in.

Where a folder name trajector would derive could also belong to a different path on your machine, it leaves that whole folder alone rather than guess, and `status` says which one and why:

```
Not collected: … stores its session files under the same name as …, and they cannot be told apart.
```

### Another process holds the proxy port

```
  error: another process holds the proxy port
    why:  another process holds the proxy port, so enabled projects cannot route through it
      Enabled projects route API credentials at this address; free the port by stopping whatever
      holds it, or run `trajector disable` in enabled projects.
      To find the holder: lsof -nP -iTCP:41100 -sTCP:LISTEN
      At the port as this device reads it: node (pid 48213).
      Once the port is free, run `trajector doctor`.
```

Something that is not trajector holds `127.0.0.1:41100` and answers nothing. There is no `fix:` line, because no command of trajector's takes a port from another process: freeing it is yours to do, and `doctor` is the step after. A silent holder this build can prove is its own proxy is stopped and replaced instead, as a hung proxy of ours; anything else is left alone.

Ask your own machine who holds it:

```sh
lsof -nP -iTCP:41100 -sTCP:LISTEN     # macOS
ss -ltnp 'sport = :41100'             # Linux
netstat -ano | findstr 41100          # Windows
```

Where this device can read the holder's name and process id, the line above states them as the operating system reports them. It never says what the program is for — that is yours to recognize.

Until the port is free, enabled projects are pointing their API traffic at a stranger. This is the one error worth acting on immediately.

#### Worked example: VS Code Remote forwarded the port

A common holder is not a program you started. If you run trajector on a remote host and open that host in VS Code Remote, the VS Code on your **local** machine forwards `41100` automatically as soon as trajector's proxy listens on it. The forwarded listener can then outlive the proxy and hold the port against the next one.

Two ways out, either of which is permanent:

* In the VS Code **Ports** panel, remove the forward for `41100`.
* Tell VS Code not to forward it at all, in your settings:

```json
"remote.portsAttributes": {
  "41100": { "onAutoForward": "ignore" }
}
```

Then run `trajector doctor`.

### `it answered the admin-token challenge, but no admin token could be read`

This is almost always **not** a foreign process. It is usually your own proxy with a stale published token:

```
This is usually an authentication problem (the proxy's published admin token
is missing or stale), not a foreign process. The proxy publishes a fresh
token each time it starts and exits on its own once idle, so a later session
usually clears it; there is no process to stop.
```

Start a new session, or wait for the idle exit. If it persists, `trajector doctor`.

### Batches are quarantined

```
warning: 3 rawcall(s) in 1 rejected batch(es) are quarantined and will not be
retried automatically.
```

A quarantined batch is one the service refused, or one this machine could no longer read back. It is moved out of the spool so a single bad batch cannot block every upload behind it. Nothing retries it automatically — you decide:

```sh
trajector doctor                              # see the batches and why
trajector doctor requeue --all                # try again, once the cause is fixed
trajector doctor discard <batch-id> --yes     # give up on it, delete for good
```

`requeue` puts the records back in the spool under a fresh batch id. `discard` is permanent and local — nothing is sent anywhere.

### The spool is full

```
The spool is full. Run `trajector upload --force` to upload and free it.
```

A full spool stops new recording rather than evicting what is already there. `trajector upload --force` sends what is waiting and frees the space.

### Uploads say the service wants a newer client

```
Uploads are paused: the service requires trajector 0.3.3 or newer (this is 0.3.2).
```

Your captured data is kept. `trajector upgrade`, then upload again.

### Uploads say the service refused this client or this device

```
Uploads are paused since 2026-09-20T14:32:00Z until 2026-09-20T14:47:00Z: the service
refused this client access. Captured data is kept.
```

The service refused this client access to the upload endpoint. Nothing is lost and nothing is asked of you: trajector waits and asks again by itself — one minute, then double, up to fifteen — and the line says when the next attempt is due. When the next attempt is due is kept on disk, so restarting waits out what is left of it. `trajector upload --force` retries at once.

```
Uploads are paused since 2026-09-20T14:32:00Z: the service refused this device's
credential. Captured data is kept.
```

This one is never retried on its own, because a refused credential does not become valid by waiting. Run `trajector login` to pair the device again; what was captured is kept and uploads once you are back.

Only one of the two stands at a time — the later refusal replaces the earlier — and `status` and `doctor` state whichever holds.

### `trajector upgrade` fails

Whatever the reason — no network, a checksum mismatch, no matching asset, a directory that cannot be written — **the binary you have is untouched**. The archive is verified before anything is replaced.

If a package manager owns the installation, `upgrade` refuses and names the command to use instead. That is not a failure; overwriting a package manager's files would break it.

### Gatekeeper or SmartScreen blocks the binary

Releases are not code-signed yet, and both systems act on the mark a _browser_ puts on a download.

* **macOS:** install with the `curl | sh` command rather than downloading in a browser. `curl` leaves no mark. Unpacking in Finder passes the mark to the binary.
* **Windows:** run `Unblock-File .\trajector.exe` after extracting.

### Bedrock or Vertex

```
problem: CLAUDE_CODE_USE_BEDROCK is set: Bedrock and Vertex channels are not
supported, so this project's traffic is not being captured.
```

Those channels are not supported. Your sessions work normally; they are simply not recorded.

### A base URL that is refused

```
problem: this project's base-URL configuration moved to http://relay.example,
which is refused: a non-loopback upstream must use https. The recorded
upstream is unchanged.
```

Trajector will not follow a base-URL change to a plaintext address off your own machine, because that would carry your credentials unencrypted. It keeps the upstream you enabled with. Use an `https` address, or change it back.

### WSL and Windows-native Claude Code

```
ok: running under WSL: Claude Code must also run inside WSL; a Windows-native
claude cannot reach this trajector
```

They cannot see each other's loopback in a useful way. Run both inside WSL, or both natively on Windows.

### A firewall prompt for the proxy

Allow loopback access. The proxy only ever binds `127.0.0.1` and is never reachable from another machine.

### The keyring is unavailable

On a headless or minimal system there may be no Secret Service. Fall back to owner-only files:

```sh
export TRAJECTOR_TOKEN_STORE=file
```

### `.gitignore` is a symbolic link

Trajector leaves it alone and warns you, rather than writing through a symlink it did not create. Add the rules yourself so the injected settings and any diagnostic bundles are never committed.

***

### Reporting a problem

```sh
trajector doctor bundle
```

This writes an archive into the current directory containing diagnostics only: identities, counters, timestamps. No captured records, no credentials, no clear-text tokens. **Nothing is sent anywhere** — inspect it yourself, then attach it to your report.
