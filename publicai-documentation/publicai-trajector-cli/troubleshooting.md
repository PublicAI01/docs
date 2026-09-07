# Troubleshooting

**Start here:**

```sh
trajector doctor
```

It checks everything below, repairs what it can on its own, and tells you what it cannot. Most of this page is only needed when `doctor` hands something back to you.

***

### Nothing is being recorded

Work through these in order:

1. **`trajector status` in the project.** Does the Project block say `Contributing`? If it says `Not enabled`, that project was never opted in — `trajector enable`.
2. **Is the device signed in?** `Not signed in` in the Device block pauses recording everywhere. Forwarding still works, which is why the session feels normal.
3. **Is the agreement current?** If it changed, recording pauses until you reconfirm with `trajector enable`. `status` says `Recording is paused everywhere: …`.
4. **Is Claude Code actually using the injected settings?** The injection is written to the project-local settings file. If you launched `claude` from a different directory, that project is not the one you enabled.
5. **`trajector doctor`.** The `live proxy confirms this project routes and records` line is an end-to-end check, not a guess.

{% hint style="info" %}
`Recorded today: 0` on its own is not evidence. That counter resets whenever the proxy restarts, and the proxy exits after 30 minutes of silence. Look at `Spool` and `Uploads` instead.
{% endhint %}

### `WARNING: another process holds 127.0.0.1:41100`

Something that is not trajector is on the port.

```
Enabled projects route API credentials at this address; find and stop the
process holding the port, or run `trajector disable` in enabled projects.
```

Find it with `lsof -i :41100` (macOS/Linux) or `netstat -ano | findstr :41100` (Windows). Until it is gone, enabled projects are pointing their API traffic at a stranger — this is the one warning worth acting on immediately.

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
WARNING: 3 rawcall(s) in 1 rejected batch(es) are quarantined and will not be
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
Uploads are paused: the service requires trajector 0.2.0 or newer (this is 0.1.0).
```

Your captured data is kept. `trajector upgrade`, then upload again.

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
