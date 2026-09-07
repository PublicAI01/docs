# Quickstart

Four commands from a fresh install to a recorded, uploaded session.

### 1. Pair the device

```sh
trajector login
```

This prints a link and opens it in your browser. Approve the pairing there; the CLI waits and stores the resulting device token in your OS keyring (Keychain, Secret Service, or Credential Manager). Pairing is per device, not per project.

### 2. Opt a project in

```sh
cd your-project
trajector enable
```

`enable` shows the data contribution agreement in full and waits for an explicit `yes`. Nothing has been captured before this point and nothing will be captured from any project where you have not done this.

After you accept, it:

* mints a token for this project and records the grant;
* injects the base URL and session hooks into this project's Claude Code settings, and prints the path it wrote;
* adds the injected settings and diagnostic bundles to `.gitignore`, so they are never committed;
* runs an end-to-end self-check against the live proxy.

```
Injected /path/to/your-project/.claude/settings.local.json (base URL and session hooks)
Added .claude/settings.local.json to .gitignore
Self-check passed: routing and recording verified end to end.
This project now contributes data. Run `trajector disable` here to stop.
```

{% hint style="info" %}
**If the project already uses a relay**, `enable` detects it and says so before asking anything:

```
Detected an existing base URL (shell environment): https://your-relay.example
Your traffic will keep flowing through it unchanged. Records from this
project are marked as third-party origin; reward terms are the same
regardless of origin.
```

Your traffic keeps going where it was going. Trajector inserts itself in front of that destination, not instead of it.
{% endhint %}

### 3. Work normally

Start Claude Code in the project and use it as you always do. The session hook starts the proxy if it is not already running — you do not start it yourself, and there is no daemon to manage. It exits on its own once it has been idle for a while.

Nothing about your session changes: the same model, the same latency, the same streaming behaviour. If recording fails for any reason, forwarding continues regardless.

### 4. Check and upload

```sh
trajector status
```

```
trajector 0.1.0

Device
  Signed in.

Project /path/to/your-project
  Contributing; recording is on for this project.

Proxy
  Running at 127.0.0.1:41100: version 0.1.0, up 4m12s.
  Recorded today: 3 (SSE degraded: 0, dropped: 0).

Spool
  184.2 KiB of 2.0 GiB used.

Uploads
  Never uploaded.
```

Uploads happen on their own once enough has accumulated. To send what is there right now:

```sh
trajector upload --force
```

```
Uploaded 1 batch(es), 3 rawcall(s).
```

Records are deleted from the spool as soon as the service acknowledges the batch by id. A `2xx` that names no batch id proves nothing was stored, so the data is kept and retried.

### Stopping

| To stop                            | Run                         | Effect                                                                                   |
| ---------------------------------- | --------------------------- | ---------------------------------------------------------------------------------------- |
| This project                       | `trajector disable`         | Removes the injection, revokes the project token, deletes this project's unuploaded data |
| This project and its uploaded data | `trajector disable --purge` | The above, plus a deletion request for uploaded data not yet delivered                   |
| Recording everywhere               | `trajector logout`          | Recording pauses on every project; forwarding is unaffected                              |
| Everything                         | `trajector uninstall`       | Removes every injection and the device token; `--delete-data` also deletes local data    |
