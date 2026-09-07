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
  upgrade      install the newest published release over this one
  version      print the trajector version
  proxy run    run the local proxy (internal; started automatically)
  hook         session hook entry points (internal; injected by enable)
```

### Exit codes

Every command uses the same three:

| Code | Meaning                                                           |
| ---- | ----------------------------------------------------------------- |
| `0`  | Success — including "there was nothing to do"                     |
| `1`  | The command failed, or `doctor` found problems it could not fix   |
| `2`  | You used it wrong: unknown command, unknown flag, extra arguments |

Commands take no flags they do not document, and reject extra arguments rather than ignoring them. `trajector status extra` exits `2`.

***

### `trajector login`

Pairs this device. Prints a link, opens it in your browser, and waits for you to approve. The resulting device token is stored in the OS keyring (Keychain / Secret Service / Credential Manager), or in owner-only files where no keyring is available.

Pairing is per device. You do not log in per project.

### `trajector logout`

Signs the device out. The token is revoked with the service and removed locally.

```
Signed out. Forwarding for enabled projects is unaffected; recording is
paused everywhere until you run `trajector login` again, and kept data
uploads once you are back.
```

Nothing is deleted. Enabled projects stay enabled, their traffic keeps flowing, and whatever is already spooled is uploaded once you sign back in. If the service cannot be reached the token is still removed locally, and a warning tells you to revoke it from your account page.

### `trajector enable`

Run inside the project you want to contribute from. This is the consent boundary — it is the only command that starts collection.

It shows the data contribution agreement in full and requires an explicit `yes`. Answering anything else, or closing the input, leaves everything untouched.

On acceptance:

1. If the device is not paired yet, pairing runs first.
2. If the project already has a base URL configured, that relay is detected and named, and traffic keeps flowing through it. Records are marked third-party origin.
3. A project token is minted and the grant recorded.
4. The base URL and session hooks are injected into the project's Claude Code settings; the file path is printed.
5. The injected settings and diagnostic bundles are added to `.gitignore`.
6. A self-check runs against the live proxy, end to end.

```
Injected /path/to/project/.claude/settings.local.json (base URL and session hooks)
Added .claude/settings.local.json to .gitignore
Self-check passed: routing and recording verified end to end.
This project now contributes data. Run `trajector disable` here to stop.
```

{% hint style="info" %}
If the agreement text changes, recording pauses and `enable` asks you to reconfirm. It will say `The data agreement changed since you last accepted it.` before showing the new text.
{% endhint %}

If `.gitignore` is a symbolic link it is left alone and you are warned to add the rules yourself — trajector does not write through symlinks it did not create.

### `trajector disable [--purge]`

Stops contributing from the current project, immediately.

```
Removed injection from /path/to/project/.claude/settings.local.json
Project token revoked; recording for this project is off.
Deleted 4 unuploaded rawcall(s) for this project (3 from the spool, 1 from rejected batches).
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

See Reading `status` and `doctor` for what each block means.

### `trajector doctor`

Diagnoses and repairs. Each line is prefixed by what happened:

| Prefix     | Meaning                                 |
| ---------- | --------------------------------------- |
| `ok:`      | Checked, nothing wrong                  |
| `fixed:`   | Was wrong, and `doctor` repaired it     |
| `problem:` | Wrong, and you have to decide something |

Exit is `0` when nothing is left over, `1` when at least one `problem:` remains.

```
trajector 0.1.0 doctor

  ok: proxy not running; it starts on demand with the next session
  ok: this project is not enabled; nothing to reconcile
  fixed: re-added the project-discovery hint to /home/you/.claude/settings.json
  ok: capture spool writable (0 B of 2.0 GiB used)
  ok: no rejected batches quarantined

Everything checks out.
```

What it checks: the device token store, whether recording is paused, the proxy, the injection and routing table agreeing with each other, the project's upstream, `.gitignore` coverage, the project-discovery hint, the spool, quarantined batches, and a live self-check against the running proxy.

#### `trajector doctor bundle`

Writes a diagnostic archive into the current directory.

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

Deletes a quarantined batch and its rawcalls from this machine for good. Use it to give up on a batch that will never upload. `--yes` skips the confirmation prompt.

### `trajector upload [--force]`

Uploads what is waiting now.

| Situation                    | Output                                                               | Exit |
| ---------------------------- | -------------------------------------------------------------------- | ---- |
| Uploaded                     | `Uploaded 1 batch(es), 3 rawcall(s).`                                | `0`  |
| Spool empty                  | `Nothing to upload.`                                                 | `0`  |
| Below thresholds             | `Below the upload thresholds; use --force to upload anyway.`         | `0`  |
| Not signed in                | "Not signed in; run `trajector login` first. Captured data is kept." | `0`  |
| Service wants a newer client | "Uploads are paused: the service requires trajector X or newer…"     | `0`  |
| A batch was rejected         | The rejection is printed loudly, the batch is quarantined            | `1`  |

`--force` uploads regardless of the size and age thresholds, and retries a paused upload.

Unreadable records are set aside rather than blocking the rest:

```
Uploaded 1 batch(es), 1 rawcall(s).
Set aside 1 unreadable rawcall(s); they were never sent. Run `trajector doctor` to inspect them.
```

### `trajector upgrade`

Installs the newest published release over this one. Four outcomes, all exit `0`:

```
Downloading trajector 0.2.0...
Upgraded trajector 0.1.0 -> 0.2.0.
A proxy from the previous build may still be running; the next session replaces it.
```

```
trajector 0.1.0 is already the newest release.
```

```
This trajector was installed with Homebrew, which owns the binary.
Run `brew upgrade trajector` instead.
```

```
This build reports version dev, which is not a published release.
Nothing was changed.
```

Anything that fails — network, checksum mismatch, no matching asset, a path that cannot be written — exits `1` and **leaves the binary you have exactly as it was**. The checksum is verified before anything is replaced, and a target that is not a file is refused outright.

### `trajector version`

```
trajector 0.1.0
```

A build that was not made from a published tag reports `dev`.

### Internal commands

These exist because something else calls them. You should not need to run them by hand.

| Command                       | Who runs it                                       |
| ----------------------------- | ------------------------------------------------- |
| `trajector proxy run`         | Started automatically by the session hook         |
| `trajector hook ensure-proxy` | The Claude Code session hook injected by `enable` |
| `trajector hook discovery`    | The one-time project discovery hint               |
