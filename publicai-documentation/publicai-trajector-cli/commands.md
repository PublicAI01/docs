# Commands

## Command reference

```
usage: trajector <command>

commands:
  login        pair this device (opens a browser link)
  logout       sign out; recording pauses, forwarding is unaffected
  enable       start contributing data from the current project
  disable      stop contributing from the current project [--purge]
  uninstall    remove every injection and optionally local data [--delete-data]
  status       show pairing, project, proxy, capture, and upload state
  doctor       diagnose and repair injection, hooks, proxy, and spool issues
  upload       upload captured data now [--force]
  forget       delete one session's not-yet-uploaded records from this machine [<session-id>]
  upgrade      install the newest published release over this one
  version      print the trajector version
  proxy run    run the local proxy (internal; started automatically)
  hook         session hook entry points (internal; injected by enable)

flags:
  --no-color   print no colour, whatever the terminal takes
```

`--no-color` is global: it works in front of any command, and turns off the colour `status` and `doctor` would otherwise use on a terminal. `NO_COLOR` in the environment and `TERM=dumb` do the same — see Reading `status` and `doctor`.

### Exit codes

Every command uses the same three:

| Code | Meaning                                                           |
| ---- | ----------------------------------------------------------------- |
| `0`  | Success — including "there was nothing to do"                     |
| `1`  | The command failed, or `doctor` found problems it could not fix   |
| `2`  | You used it wrong: unknown command, unknown flag, extra arguments |

Commands take no flags they do not document, and reject extra arguments rather than ignoring them. `trajector status extra` exits `2`.

Every command answers `--help` or `-h` with its usage line and exits `0`, and refuses a flag it does not know with that same usage line, the flag it refused, and exit `2`. Both happen before the command reads its arguments, so `trajector forget --help` prints usage instead of looking for a session named `--help`.

```
$ trajector enable --help
usage: trajector enable [--no-proxy] [--no-earlier]
```

***

### `trajector login`

Pairs this device. Prints a link, opens it in your browser, and waits for you to approve. The resulting device token is stored in the OS keyring (Keychain / Secret Service / Credential Manager), or in owner-only files where no keyring is available.

Pairing is per device. You do not log in per project.

### `trajector logout`

Signs the device out. The token is revoked with the service and removed locally.

```
Signed out. Forwarding for enabled projects is unaffected, and kept data
uploads once you are back.
Recording is paused everywhere: this device is signed out; run `trajector login`.
```

Nothing is deleted. Enabled projects stay enabled, their traffic keeps flowing, and whatever is already spooled is uploaded once you sign back in. If the service cannot be reached the token is still removed locally, and a warning tells you to revoke it from your account page.

### `trajector enable [--no-proxy] [--no-earlier]`

Run inside the project you want to contribute from. This is the consent boundary — it is the only command that starts collection.

It shows the data contribution agreement in full and requires an explicit `yes`. Answering anything else, or closing the input, leaves everything untouched.

On acceptance:

1. If the device is not paired yet, pairing runs first.
2. If the project already has a base URL configured, that relay is detected and named, and traffic keeps flowing through it. Records are marked third-party origin.
3. A project token is minted and the grant recorded.
4. It asks to turn on Claude Code's `showThinkingSummaries` for this project only. The setting is required: if you say no, recording goes on, but sessions without reasoning summaries are not rewarded — see [Reasoning summaries](reasoning-summaries.md). `trajector disable` puts the setting back.
5. The base URL and session hooks are injected into the project's Claude Code settings; the file path is printed. With `--no-proxy`, the session hooks are installed and no base URL is injected.
6. The injected settings and diagnostic bundles are added to `.gitignore`.
7. A self-check runs against the live proxy, end to end.

```
Injected /path/to/project/.claude/settings.local.json (base URL and session hooks)
Added .claude/settings.local.json*, trajector-doctor-*.tar.gz, trajector-doctor-*/ to .gitignore
Self-check passed: routing and recording verified end to end.
Remote Control: inside this project, /remote-control will not be available. …
Calls the local proxy did not witness are rewarded at a lower rate than witnessed ones. …
This project now contributes data. Run `trajector disable` here to stop.
Recording: on (1 project(s))
```

{% hint style="info" %}
If the agreement text changes, recording pauses and `enable` asks you to reconfirm. It will say `The data agreement changed since you last accepted it.` before showing the new text. Upgrading to TBD-CLI-VERSION is such a change: the agreement moves to version `TBD-AGREEMENT`. Run it with `--no-proxy` and `--no-earlier` if the project uses them: `enable` installs the shape you give it, and without `--no-earlier` it collects the session files the project had before it was first enabled.
{% endhint %}

| Flag           | Effect                                                                                                                    |
| -------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `--no-proxy`   | Record this project from its session files and git observation only. No base URL is injected, and `/remote-control` stays available inside the project |
| `--no-earlier` | Leave the session files this project already has alone — they are neither registered nor read — so only the sessions that run from now on are collected |

`enable` states which shape it installed, and `status` and `doctor` say which shape a project records in. In the shape with a base URL, `/remote-control` is not available inside the project; `claude remote-control` still works and both sources are still recorded.

The two flags are orthogonal: `--no-earlier` decides what happens to the past, `--no-proxy` decides which sources record from now on, and neither implies the other. `--no-earlier` is recorded on the grant, and `status` states it under the project with the way back:

```
  Earlier sessions were skipped at enable; run `trajector enable` again (with --no-proxy if you use it) to collect them.
```

The way back re-declares the whole install, which is why the sentence names the shape flag too: a bare `trajector enable` in a project recorded without a proxy would also write the shape with one.

Before injecting, `enable` judges from the configuration readable on this machine whether Claude Code will load trajector's hooks in this project at all — hooks disabled in a settings layer, managed settings, a workspace not yet trusted, or a project on a Windows drive mounted into WSL. It says what it found and what to do, and never guesses past what it can read.

```
Judged from configuration readable on this machine, Claude Code will not load trajector's hooks in this project (...)
Only the proxy records this project for now; its session files are not read
```

`enable` also finds the project's earlier session files, up to a fixed limit and without listing any other project's directory, and tells you how many it found and how old the oldest is. It then asks for them to be read right away, rather than waiting for a session hook to name them, and says how many it is reading:

```
4 earlier session record(s) will be collected once; the oldest is from 2026-09-02.
Reading 4 earlier session record(s) in the background.
```

With `--no-earlier` nothing is looked for and nothing is read, and the line says so:

```
Earlier session records skipped.
```

Where no reader could take the ask, `enable` does not pretend it did: it says the files are registered, and names `trajector doctor` as the way to ask again. `enable` ends with the same `Recording: …` verdict line `status` opens with. And it states, before anything of yours is uploaded, what a call the local proxy did not witness is worth, with a link to Rewards for the current rates.

If `.gitignore` is a symbolic link it is left alone and you are warned to add the rules yourself — trajector does not write through symlinks it did not create.

### `trajector disable [--purge]`

Stops contributing from the current project, immediately.

```
Removed injection from /path/to/project/.claude/settings.local.json
Project token revoked; recording for this project is off.
Deleted 4 unuploaded record(s) for this project (3 from the spool, 1 from rejected batches).
This project no longer contributes data.
```

The deletion count is split by where the data was, because those are two different places to have to trust.

| Flag      | Effect                                                                                |
| --------- | ------------------------------------------------------------------------------------- |
| `--purge` | Also request deletion of this project's uploaded data that has not yet been delivered |

With `--purge` you additionally get:

```
Requested deletion of this project's uploaded, undelivered data.
```

Data already delivered and rewarded is covered by an irrevocable license and cannot be recalled. Everything else can.

Running `disable` in a project that was never enabled prints `This project is not enabled; nothing to do.` and exits `0`.

### `trajector status`

Prints the dashboard and changes nothing. It never repairs anything and never starts a proxy just to look at one — that is `doctor`'s job.

It opens with one verdict line, before the version and the blocks, so the answer to "is anything being recorded" is the first thing on screen:

```
Recording: on (3 project(s))
```

The other four readings are `Recording: PAUSED on this device`, `Recording: STOPPED on this device (spool full)`, `Recording: STOPPED on this device (routing table unreadable)`, and `Recording: off in this project`. `status` exits `1` when it printed anything at error severity, so a script can tell a recording device from a stopped one without reading the text.

Per enabled project it also reports the second source:

```
  Session files: 4 session(s) registered; last read 2026-09-20T09:12:41Z; 12.4 KiB not read yet.
  Sessions started under another spelling of this project's path, or under a directory name Claude Code was told to use instead, are stored under names this device does not compute and are not collected.
```

An injection made before one of the session hooks existed is stated as a problem, with `trajector doctor` as the fix:

```
  error: a session hook is missing from this project's injected settings
    why:  /path/to/project/.claude/settings.local.json was injected before this hook existed, so part of this project is not reported
    fix:  trajector doctor
```

The Uploads block counts what is waiting to upload, by kind:

```
  Records waiting to upload: 3 rawcall(s), 12 segment(s), 2 session snapshot(s), 1 git snapshot(s); the oldest is from 2026-09-20T08:41:02Z.
```

`Session files: none registered yet.` appears until a hook has reported one. The reader's findings about the files' shape are counts and field names only — never a path or a session id — and a count of assistant lines that carried no reasoning points at `showThinkingSummaries` when that setting is off, saying that sessions without reasoning summaries are not rewarded — see [Reasoning summaries](reasoning-summaries.md).

The Device block states what goes up of images and documents, whose content redaction cannot see — one of these, the second when `upload_images_and_documents` is `false` in `config.json`:

```
  Images and documents: uploaded.
  Images and documents: placeholders only; their content is not uploaded (upload_images_and_documents is false in config.json). Notebook images that older Claude Code versions keep are uploaded as they are; see PRIVACY.md.
```

The Device block also says how trajector keeps itself up to date: when the last automatic check ran and what it found, or why automatic upgrades are off.

```
  Automatic upgrades: last checked 2026-10-13T09:02:17Z, up to date.
  Automatic upgrades: off (config.json).
  Automatic upgrades: off until you accept the current data agreement (run `trajector enable`).
```

A failed attempt is a `warning:` that names `trajector upgrade`; it does not count towards the exit code, because recording goes on.

`status` also runs when `config.json` cannot be read, which stops every other command except `doctor bundle`. It then prints `error: Images and documents: placeholders only, because config.json cannot be read.` with the failure on the `why:` line, and exits `1`.

The Spool block adds a line when this build met a segment shape it cannot redact and kept the segment back:

```
  12 segment(s) from 2 session(s) (48.1 KiB) are held on this machine because their shape is new to this build; they are not uploaded.
```

The Uploads block states a standing refusal from the service, with when uploads stopped:

```
  Uploads are paused since 2026-09-20T14:32:00Z until 2026-09-20T14:47:00Z: the service refused this client access. Captured data is kept.
  Uploads are paused since 2026-09-20T14:32:00Z: the service refused this device's credential. Captured data is kept.
```

The first is retried by itself — one minute, then double, up to fifteen — and the line says when the next attempt is due. The second is never retried by itself: run `trajector login`. Only one of the two stands at a time; the later refusal replaces the earlier.

See Reading `status` and `doctor` for what each block means.

### `trajector doctor [--all]`

Diagnoses and repairs. Each line is prefixed by what happened:

| Prefix     | Meaning                                                |
| ---------- | ------------------------------------------------------ |
| `error:`   | Wrong, and you have to decide something                |
| `warning:` | Worth reading; recording continues                     |
| `fixed:`   | Was wrong, and `doctor` repaired it                    |
| `ok:`      | Checked, nothing wrong                                 |
| `note:`    | A fact that asks nothing of you                        |

`doctor` marks every line it prints; `status` marks only what is wrong.

Exit is `0` when nothing is left over, `1` when at least one `error:` remains.

```
trajector 0.4.0 doctor

  ok: automatic upgrades: last checked 2026-10-13T09:02:17Z, up to date
  ok: proxy not running; it starts on demand with the next session
  ok: this project is not enabled; nothing to reconcile
  fixed: re-added the project-discovery hint to /home/you/.claude/settings.json
  ok: capture spool writable (0 B of 2.0 GiB used)
  ok: no rejected batches quarantined

Everything checks out.
```

`doctor` additionally adds a session hook missing from an injection made before that hook existed, and walks the project's session files:

```
  fixed: completed the session hooks in /path/to/project/.claude/settings.local.json (1 added)
  ok: every session file of this project is registered (4 session(s))
```

Sessions written after the grant that no hook of trajector's reported are a warning:

```
  warning: could not determine why the hooks did not report 1 session(s) of this project
```

That is not a problem `doctor` exits `1` over, and neither is the `note:` it adds for sessions that predate the grant; what it counts is a repair it could not make. See Troubleshooting for the causes listed under the warning.

These are the repairs it reports on the session files and the pause:

```
  fixed: recording resumed: this build read the session files again and every line it read ended in a newline
  fixed: recording resumed after upgrade: it was paused by <previous version>, and this build is <version>
  fixed: asked for 2 never-read session file(s) of this project to be read
  fixed: registered 1 session file(s) of this project that no hook reported, and asked for them to be read
```

Where no reader takes the ask, `doctor` reports that instead of counting it repaired. It also notes what is held on this machine, and moves a held segment back for upload once this build reads it cleanly:

```
  note: 12 segment(s) from 2 session(s) (48.1 KiB) are held on this machine because their shape is new to this build; they are not uploaded
  fixed: 12 held segment(s) read cleanly under this build and will be uploaded
```

`doctor bundle` carries the same findings. `doctor` reads a project's recording shape from the grant written at enable time: the project's settings file is checked against that shape and repaired when it disagrees, never the other way round.

What it checks: the device token store, whether recording is paused, automatic upgrades, the proxy, the injection and routing table agreeing with each other, the project's upstream, `.gitignore` coverage, the project-discovery hint, the spool, quarantined batches, and a live self-check against the running proxy.

When an upgrade was interrupted before the new build accepted it, `doctor` puts the previous binary back in place.

#### `trajector doctor --all`

Checks this device once, and then every project enabled on it, each under its own heading — the project's root — instead of only the project you are in. A project whose directory is gone is noted and skipped, and one that cannot be checked is an `error:` under its own heading that does not stop the rest.

```
trajector 0.4.0 doctor

Device
  ok: automatic upgrades: last checked 2026-10-13T09:02:17Z, up to date
  ok: proxy running at 127.0.0.1:41100 (version 0.4.0, up 4m12s)
  ...

/path/to/project-a
  ok: injection and routing agree for this project
  ...

/path/to/project-b
  note: this directory is not there any more, so it was skipped

Everything checks out.
```

The exit code is the same as for one project: `1` when an `error:` is left anywhere.

#### `trajector doctor bundle`

Writes a diagnostic archive into the current directory. It runs even when `config.json` cannot be read, and records why together with whether images and documents go up as placeholders.

```
Wrote /path/to/project/trajector-doctor-20260810-094902.tar.gz
It contains diagnostics only: no captured data, no credentials, no clear-text
tokens. Review its contents, then attach it to your report.
```

Nothing is sent anywhere. You inspect it and decide whether to attach it.

#### `trajector doctor requeue <batch-id>|--all`

Puts a quarantined batch back in the spool so the next upload repacks and retries it. Use this once whatever stopped the batch is fixed.

A quarantined batch is one the service refused, or one whose records this machine could no longer read back. It is moved out of the spool so one bad batch cannot block every upload behind it.

#### `trajector doctor discard <batch-id>|--all [--yes]`

Deletes a quarantined batch and its records from this machine for good. Use it to give up on a batch that will never upload. `--yes` skips the confirmation prompt.

### `trajector upload [--force]`

Uploads what is waiting now.

| Situation                    | Output                                                               | Exit |
| ---------------------------- | -------------------------------------------------------------------- | ---- |
| Uploaded                     | `Uploaded 1 batch(es), 3 record(s).`                                 | `0`  |
| Spool empty                  | `Nothing to upload.`                                                 | `0`  |
| Below thresholds             | `Below the upload thresholds; use --force to upload anyway.`         | `0`  |
| Not signed in                | "Uploads are paused: this device is signed out."                     | `0`  |
| Service wants a newer client | "Uploads are paused: the service requires client version X or newer; this build is Y." | `0`  |
| The service refused this client access | "Uploads are paused since … until …: the service refused this client access. Captured data is kept." | `0`  |
| The service refused this device's credential | "Uploads are paused since …: the service refused this device's credential. Captured data is kept." | `0`  |
| A batch was rejected         | The rejection is printed loudly, the batch is quarantined            | `1`  |

`--force` uploads regardless of the size and age thresholds, and retries a paused upload.

A refused client access is also retried on its own — after a minute, then double that, up to fifteen minutes — and when the next attempt is due is kept on disk, so a restart waits out what is left of it. A refused device credential is never retried on its own: `trajector login` ends it.

Unreadable records are set aside rather than blocking the rest:

```
Uploaded 1 batch(es), 1 record(s).
Set aside 1 unreadable record(s); they were never sent. Run `trajector doctor` to inspect them.
```

### `trajector forget [<session-id>]`

Deletes one session's not-yet-uploaded records from this machine — from both spool slots and from any quarantined batch — and reports one total. With no argument it acts on the current session, so it can be run from inside a Claude Code session; outside one, pass the session id.

```
Forgetting session 0f0c1f6e-...: deleting its records that have not been uploaded yet from this machine. Uploaded data is deleted from the Dashboard instead. Recording of this session continues; only what was collected so far is gone.
Deleted 7 record(s) of session 0f0c1f6e-....
```

It acts on the session, not on the working directory, so it runs from anywhere. An argument that is not shaped like a session id — only letters, digits, `.`, `_` and `-` can appear in one — is refused with exit `2` rather than matched against nothing.

### `trajector upgrade`

Installs the newest published release over this one. Before it keeps the new release, it runs it once and has it start a proxy of its own; if either fails, the previous binary is put back in place.

These outcomes exit `0`:

```
Upgraded trajector 0.3.7 -> 0.4.0.
A proxy from the previous build may still be running; the next session replaces it.
```

```
trajector 0.4.0 is already the newest release.
```

```
This trajector was installed with Homebrew, which owns the binary.
Run `brew upgrade trajector` instead.
```

```
This build reports version dev, which is not a published release.
Nothing was changed.
```

When trajector also checks on its own — the binary in place is a published release, `auto_upgrade` is not `false`, and the data agreement you accepted describes the check — the first two end with:

```
trajector also checks for a newer release once a day on its own; set "auto_upgrade": false in config.json to stop that.
```

A release that changes the data agreement bumps its version: recording pauses until you reconfirm, and the next `trajector enable` (with `--no-proxy` and `--no-earlier` if you use them) shows the new text and asks you to accept it once. Forwarding is untouched while the pause stands.

These exit `1`:

* **The new release does not start.** It was not kept: the command says which version it kept and why the new one was not installed. If the previous binary could not be put back either, the error says where it is.
* **Another upgrade is running on this device.** `upgrade` waits two seconds, then says so; try again in a minute.
* **An earlier upgrade was interrupted** before the new build accepted it. `upgrade` puts the previous binary back, says which version it put back, and stops there; run it again.
* **Anything else fails** — network, checksum mismatch, no matching asset, a path that cannot be written. The binary you have is left **exactly as it was**: the checksum is verified before anything is replaced, and a target that is not a file is refused outright.

### `trajector version`

```
trajector 0.4.0
```

A build that was not made from a published tag reports `dev`.

### Internal commands

These exist because something else calls them. You should not need to run them by hand.

| Command                       | Who runs it                                       |
| ----------------------------- | ------------------------------------------------- |
| `trajector proxy run`         | Started automatically by the session hook         |
| `trajector hook ensure-proxy` | The Claude Code session hook injected by `enable` |
| `trajector hook discovery`    | The one-time project discovery hint               |
