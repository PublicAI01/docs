# Troubleshooting

**Start here:**

```sh
trajector doctor
```

It checks everything below, repairs what it can on its own, and tells you what it cannot. Most of this page is only needed when `doctor` hands something back to you.

***

### Nothing is being recorded

Start with the first line of `trajector status`. It is the verdict — `Recording: on (3 project(s))`, `Recording: PAUSED on this device`, `Recording: STOPPED on this device (spool full)`, `Recording: STOPPED on this device (routing table unreadable)`, or `Recording: off in this project` — and it says which of the steps below applies. `status` exits `1` when it printed an error, so a script can ask the same question.

Then work through these in order:

1. **`trajector status` in the project.** Does the Project block say `Contributing`? If it says `Not enabled`, that project was never opted in — `trajector enable`.
2. **Is the device signed in?** `Not signed in` in the Device block pauses recording everywhere. Forwarding still works, which is why the session feels normal.
3. **Is the agreement current?** If it changed, recording pauses until you reconfirm with `trajector enable` — with `--no-proxy` and `--no-earlier` if the project uses them: `enable` installs the shape you give it, and without `--no-earlier` it collects the session files the project had before it was first enabled. An upgrade to TBD-CLI-VERSION is such a change. `status` prints `error: Recording is paused everywhere` with the reason on the `why:` line under it and the command that ends it on the `fix:` line. An unreadable consent record pauses the same way, under its own reason naming the file — `trajector enable` is the way out of that one too.
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

### `a session hook is missing from this project's injected settings`

```
  error: a session hook is missing from this project's injected settings
    why:  /path/to/project/.claude/settings.local.json was injected before this hook existed, so part of this project is not reported
    fix:  trajector doctor
```

An injection made before a hook existed lacks it. `trajector doctor` adds it:

```
  fixed: completed the session hooks in /path/to/project/.claude/settings.local.json (1 added)
```

It adds only the hooks that are missing and writes nothing else of the file, so the shape you chose at enable time — with or without `--no-proxy` — stays as it is.

### Claude Code will not load the hooks

`enable`, `status` and `doctor` state what a static reading of Claude Code's configuration says:

```
Judged from configuration readable on this machine, Claude Code will not load trajector's hooks in this project (...)
```

The reason is in the parentheses. What to check:

* **Hooks disabled in a settings layer**, or **managed settings** deciding it. Change the setting where it is set, or ask whoever manages it to. The reason names the settings layer that decided it, not a file path.
* **The workspace is not trusted yet.** Claude Code runs a project's hooks only after you accept the trust dialog, and no file records that answer, so no command of trajector's can confirm it. It is one of the causes listed under the next entry.
* **The project is on a Windows drive mounted into WSL.** Claude Code then runs on the Windows side and its hooks cannot reach this trajector. Run both on the same side.

With a base URL injected, the proxy keeps recording while the hooks do not load; the session files are simply not read. With `--no-proxy` the hooks are the only source, so nothing is recorded from the project until they load.

### `doctor` found session files no hook reported

`doctor` walks the project's session files and answers with the distinction it can make, never with a guess.

**Sessions written before the project was enabled** are registered on the spot, and `doctor` asks for them to be read right away instead of waiting for a session hook to name them, as `enable` does:

```
  fixed: registered 3 session file(s) of this project that no hook reported, and asked for them to be read
```

Where they cannot be asked for, `doctor` says so as an `error:` and names them, with the way to register them:

```
  note: 3 session(s) of this project predate its grant, so no hook of trajector's reported them
      Run `trajector enable` in this project (with --no-proxy if you use it) to register them.
```

The note itself is not a problem, and `doctor` does not exit `1` over it. A project enabled with `--no-earlier` is left alone here by design.

**Sessions written after the grant that no hook reported** are reported as the open question they are:

```
  warning: could not determine why the hooks did not report 1 session(s) of this project
      Claude Code runs a project's hooks only once the workspace is trusted, and that answer is given in a dialog no file records.
      A session started with `claude --bare`, or with `--settings` naming another file, loads other settings than the ones this project carries.
      A setting in the environment of the session's own process is not readable from here either; under WSL, so are managed settings on the Windows side.
```

None of those three can be confirmed from this machine, which is why the headline states the question rather than an answer. Work through them yourself: accept the trust dialog, start sessions without `--bare` or a `--settings` file of their own, and check the environment the session's own process runs in.

### Segments held on this machine

A session file line whose shape this build's redaction does not cover does not stop recording. Its segment is held back, and `status` says how much is held:

```
  12 segment(s) from 2 session(s) (48.1 KiB) are held on this machine because their shape is new to this build; they are not uploaded.
```

That one segment is held on your machine where nothing uploads from, reading goes on from the next segment, and the rest of the session is recorded and uploaded as usual. Held segments sit outside the spool quota.

What to do: `trajector upgrade`, then `trajector doctor`. A build that reads a held segment cleanly moves it back for upload when `doctor` runs, and reports it:

```
  fixed: 12 held segment(s) read cleanly under this build and will be uploaded
```

If you would rather it never left your machine, `trajector forget <session-id>` deletes what is held for that session.

Recording does pause on the whole device when a read contradicts itself — a line read without the newline that ends it, where nothing that read produced can be trusted. `status` names `trajector upgrade` and then `trajector doctor` as the way out. `trajector doctor` lifts the pause once it has read the session files again and every line it read was whole, with the build you have:

```
  fixed: recording resumed: this build read the session files again and every line it read ended in a newline
```

After an upgrade, `doctor` lifts a pause the previous build set:

```
  fixed: recording resumed after upgrade: it was paused by <previous version>, and this build is <version>
```

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
  error: the holder of the proxy port did not answer a probe: 127.0.0.1:41100 (…)
    why:  another process holds the proxy port, so enabled projects cannot route through it
      Enabled projects route API credentials at this address; free the port by stopping whatever holds it, or run `trajector disable` in enabled projects.
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

### `it answered the admin-token challenge, but no admin token for this address could be read`

This is almost always **not** a foreign process. It is usually your own proxy with a stale published token:

```
This is usually an authentication problem (the proxy's published admin token is missing or stale), not a foreign process. The proxy publishes a fresh token each time it starts and exits on its own once idle, so a later session usually clears it; there is no process to stop.
```

Start a new session, or wait for the idle exit. If it persists, `trajector doctor`.

### Batches are quarantined

```
error: 3 record(s) in 1 rejected batch(es) are quarantined and will not be retried automatically
  why:  the service refused them or this machine could not read them, and neither is retried on its own
  fix:  trajector doctor requeue <batch-id>
    Name --all in place of a batch id to take every quarantined batch at once.
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
  error: recording is stopped: the capture spool is full
    why:  captured data fills the spool to its quota, so nothing more can be written
    fix:  trajector upload --force
```

A full spool stops new recording rather than evicting what is already there. `trajector upload --force` sends what is waiting and frees the space.

### Uploads say the service wants a newer client

```
Uploads are paused: the service requires client version <minimum> or newer; this build is <version>.
Captured data is kept.
Run `trajector upgrade` to install the newest release. Or retry with --force.
```

Your captured data is kept. `trajector upgrade`, then upload again. The service accepts uploads only from recent releases: from 14 October 2026 it refuses uploads from builds older than 0.3.5.

A build the service calls too old checks for a newer release every hour instead of every day, and installs it on its own — once you have accepted agreement `2026-10-13`, unless `auto_upgrade` is `false` or Homebrew owns the installation. Builds before 0.4.0 never upgrade themselves — run `trajector upgrade` (or `brew upgrade trajector`) once.

### Uploads say the service refused this client or this device

```
Uploads are paused since 2026-09-20T14:32:00Z until 2026-09-20T14:47:00Z: the service refused this client access. Captured data is kept.
```

The service refused this client access to the upload endpoint. Nothing is lost and nothing is asked of you: trajector waits and asks again by itself — one minute, then double, up to fifteen — and the line says when the next attempt is due. When the next attempt is due is kept on disk, so restarting waits out what is left of it. `trajector upload --force` retries at once.

```
Uploads are paused since 2026-09-20T14:32:00Z: the service refused this device's credential. Captured data is kept.
```

This one is never retried on its own, because a refused credential does not become valid by waiting. Run `trajector login` to pair the device again; what was captured is kept and uploads once you are back.

Only one of the two stands at a time — the later refusal replaces the earlier — and `status` and `doctor` state whichever holds.

### `trajector upgrade` fails

Whatever the reason — no network, a checksum mismatch, no matching asset, a directory that cannot be written — **the binary you have is untouched**. The archive is verified before anything is replaced.

`upgrade` also runs the new release once, and has it start a proxy of its own, before it keeps it. When either fails, the previous binary is put back in place, the command says that the new version was not installed and why, and it exits `1`. If the previous binary cannot be put back, the error says where it is.

Two more cases exit `1` and only ask you to run it again:

* **Another upgrade is running on this device.** `upgrade` waits two seconds, then says so. Try again in a minute.
* **An earlier upgrade was interrupted** before the new build accepted it. `upgrade` puts the previous binary back, says which version it put back, and stops; `trajector doctor` does the same. Run `trajector upgrade` again.

If a package manager owns the installation, `upgrade` refuses and names the command to use instead. That is not a failure; overwriting a package manager's files would break it.

### `Automatic upgrades: the last attempt … failed`

```
  warning: Automatic upgrades: the last attempt at 2026-10-13T09:02:17Z failed: <reason>. Run `trajector upgrade`.
```

The background check failed, and the line says why — GitHub could not be reached, for instance. The binary you have is still in place, and recording goes on: this line does not change the exit code of `status`. `trajector upgrade` tries the same thing in front of you and says what went wrong. `doctor` reports it as an `error:` with `trajector upgrade` on the `fix:` line.

When a release the background check found does not start on this machine, the next command you run says so, together with the version that was kept:

```
trajector <version> does not start on this machine, so <kept> was kept. `trajector upgrade` tries again.
```

### `cannot start its proxy, so trajector does not work until it is reinstalled`

```
trajector <version> cannot start its proxy, so trajector does not work until it is reinstalled: see Installation in README.
```

A newer build took over the proxy from an older one and then could not start its own. The session hooks say it as their first line, and `status` and `doctor` report it. Reinstall it as Installation describes, then run `trajector doctor`.

### Gatekeeper blocks the binary

Releases are not code-signed yet, and Gatekeeper acts on the mark a _browser_ puts on a download. Install with the `curl | sh` command rather than downloading in a browser: `curl` leaves no mark. Unpacking in Finder passes the mark to the binary.

### Bedrock or Vertex

```
  error: CLAUDE_CODE_USE_BEDROCK is set: Bedrock and Vertex channels are not supported, so this project's traffic is not being captured.
```

Those channels are not supported. Your sessions work normally; they are simply not recorded.

### A base URL that is refused

```
  error: this project's base-URL configuration moved to http://relay.example, which is refused: a non-loopback upstream must use https. The recorded upstream is unchanged.
```

Trajector will not follow a base-URL change to a plaintext address off your own machine, because that would carry your credentials unencrypted. It keeps the upstream you enabled with. Use an `https` address, or change it back.

### WSL and Windows-native Claude Code

```
  ok: running under WSL: Claude Code must also run inside WSL; a Windows-native claude cannot reach this trajector
```

They cannot see each other's loopback in a useful way. Run both inside WSL, or both natively on Windows.

### A firewall prompt for the proxy

Allow loopback access. The proxy only ever binds `127.0.0.1` and is never reachable from another machine.

### `config.json` cannot be read

```
error: Images and documents: placeholders only, because config.json cannot be read.
  why:  reading /home/you/.config/trajector/config.json: unexpected end of JSON input; images and documents go up as placeholders until it reads.
```

Your `config.json` exists but does not read — cut short, not JSON, or not readable by your user. Every command except `trajector status` and `trajector doctor bundle` stops on it and names the file, rather than guess at settings you may have changed. A proxy that is already running keeps the settings it started with, except that it reads the file again for every batch: until the file reads, it uploads images and documents as placeholders, and `proxy.log` says so. Fix the file — or remove it to go back to every default, which uploads images and documents — and the next command runs as before.

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

This writes an archive into the current directory containing diagnostics only: identities, counters, timestamps. No captured records, no credentials, no clear-text tokens. It is written even when `config.json` cannot be read. **Nothing is sent anywhere** — inspect it yourself, then attach it to your report.
