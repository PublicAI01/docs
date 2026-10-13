# Quickstart

Four commands from a fresh install to a recorded, uploaded session.

{% embed url="https://www.youtube.com/watch?v=Kf4_Vkw1N7Q" %}
Install, enable, work, upload — the whole flow in one sitting
{% endembed %}

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

#### Choosing a recording shape

`enable` installs one of two shapes, and says which one it installed. `status` and `doctor` say which shape a project records in.

```sh
trajector enable              # proxy + session files + git observation
trajector enable --no-proxy   # session files + git observation only
```

* **`trajector enable`** injects the base URL and the session hooks. Calls go through the recording proxy, the project's session files are read while sessions run, and the state of the project's git repository is observed. Inside the project, `/remote-control` is not available; `claude remote-control` still works and both sources are still recorded.
* **`trajector enable --no-proxy`** installs the session hooks and injects no base URL. The project records from its session files and git only, and `/remote-control` stays available inside the project.

{% hint style="info" %}
**Desktop Claude Code sessions cannot go through the proxy.** The desktop host sets its own API base URL and ignores the settings the CLI writes for a project, so no proxy can sit in front of it. Those sessions are still recorded from their session files, and a session file recorded live counts at the full rate — see Rewards for what "recorded live" means.
{% endhint %}

If an upgrade — by `trajector upgrade` or the automatic one — brings a changed data agreement, recording pauses and the next `trajector enable` asks you to confirm the updated agreement once; run it with `--no-proxy` and `--no-earlier` if the project uses them. Upgrading to 0.4.0 is such a case: the agreement moves to version `2026-10-13`, and accepting it is also what turns on the automatic upgrade check.

After you accept, it:

* mints a token for this project and records the grant;
* offers to turn on Claude Code's `showThinkingSummaries` for this project, and asks first;
* injects the base URL and session hooks into this project's Claude Code settings, and prints the path it wrote;
* adds the injected settings and diagnostic bundles to `.gitignore`, so they are never committed;
* runs an end-to-end self-check against the live proxy.

```
Injected /path/to/your-project/.claude/settings.local.json (base URL and session hooks)
Added .claude/settings.local.json*, trajector-doctor-*.tar.gz, trajector-doctor-*/ to .gitignore
Self-check passed: routing and recording verified end to end.
Remote Control: inside this project, /remote-control will not be available. …
Calls the local proxy did not witness are rewarded at a lower rate than witnessed ones. …
This project now contributes data. Run `trajector disable` here to stop.
Recording: on (1 project(s))
```

{% hint style="info" %}
**If the project already uses a relay**, `enable` detects it and says so before it changes anything:

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
Recording: on (1 project(s))
trajector 0.4.0

Device
  Signed in.
  Images and documents: uploaded.
  Automatic upgrades: last checked 2026-10-13T09:02:17Z, up to date.

Project /path/to/your-project
  Contributing; recording is on for this project.
  Remote Control: inside this project, /remote-control will not be available. …
  Calls the local proxy did not witness are rewarded at a lower rate than witnessed ones. …
  Judged from configuration readable on this machine, Claude Code will load trajector's hooks in this project
  Session files: 1 session(s) registered; last read 2026-10-13T09:01:55Z; 0 B not read yet.
  Sessions started under another spelling of this project's path, or under a directory name Claude Code was told to use instead, are stored under names this device does not compute and are not collected.

Proxy
  Running at 127.0.0.1:41100: version 0.4.0, up 4m12s.
  Recorded since it started: 3 (SSE degraded: 0, dropped: 0).

Spool
  184.2 KiB of 2.0 GiB used.

Uploads
  Never uploaded.
  Records waiting to upload: 3 rawcall(s), 12 segment(s); the oldest is from 2026-10-13T08:58:40Z.
```

The first line is the verdict: `Recording: on (N project(s))` when something is being recorded, and `Recording: PAUSED on this device`, `Recording: STOPPED on this device (spool full)`, `Recording: STOPPED on this device (routing table unreadable)` or `Recording: off in this project` when nothing is. `status` exits `1` when it printed an error.

`Images and documents: uploaded.` under Device says that the content of images and documents goes up with the rest: redaction cannot see what a screenshot or a PDF shows. Data and privacy says how to upload them as placeholders instead. `Automatic upgrades:` says when trajector last checked for a newer release and what it found, or why it does not check.

Uploads happen on their own once enough has accumulated. To send what is there right now:

```sh
trajector upload --force
```

```
Uploaded 1 batch(es), 3 record(s).
```

Records are deleted from the spool as soon as the service acknowledges the batch by id. A `2xx` that names no batch id proves nothing was stored, so the data is kept and retried.

### Stopping

| To stop                            | Run                         | Effect                                                                                   |
| ---------------------------------- | --------------------------- | ---------------------------------------------------------------------------------------- |
| This project                       | `trajector disable`         | Removes the injection, revokes the project token, deletes this project's unuploaded data |
| This project and its uploaded data | `trajector disable --purge` | The above, plus a deletion request for uploaded data not yet delivered                   |
| Recording everywhere               | `trajector logout`          | Recording pauses on every project; forwarding is unaffected                              |
| Everything                         | `trajector uninstall`       | Removes every injection and stops the proxy; `--delete-data` also deletes local data and the device token |
