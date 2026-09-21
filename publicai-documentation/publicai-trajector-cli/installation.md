# Installation

### macOS and Linux

```sh
curl -fsSL https://raw.githubusercontent.com/PublicAI01/trajector-cli/main/install.sh | sh
```

The script picks the archive for your platform, verifies it against the release's checksum file, and installs `trajector` into `~/.local/bin`. **An archive that does not match its checksum is never installed** — the script stops and leaves whatever you already had untouched.

If `~/.local/bin` is not on your `PATH`, the script tells you the line to add for your shell.

#### Options

| Variable                | Effect                                         |
| ----------------------- | ---------------------------------------------- |
| `TRAJECTOR_INSTALL_DIR` | Install somewhere other than `~/.local/bin`    |
| `TRAJECTOR_VERSION`     | Pin a release, e.g. `TRAJECTOR_VERSION=v0.3.3` |

```sh
TRAJECTOR_INSTALL_DIR=/usr/local/bin TRAJECTOR_VERSION=v0.3.3 \
  sh -c "$(curl -fsSL https://raw.githubusercontent.com/PublicAI01/trajector-cli/main/install.sh)"
```

{% hint style="warning" %}
**On macOS, install with the command above rather than downloading the archive in a browser.** Releases are not code-signed yet, and macOS marks anything a browser downloaded. Unpacking such an archive in Finder passes that mark to the binary, and Gatekeeper then refuses to run it. A download made by `curl` carries no mark.
{% endhint %}

### Windows

There is no Windows build yet — the [releases page](https://github.com/PublicAI01/trajector-cli/releases) carries macOS and Linux archives only, until that platform has been tested end to end against the service.

Run trajector under [WSL](https://learn.microsoft.com/windows/wsl/install): install a Linux distribution, then run the same install command above inside it.

Homebrew is not available yet.

### Two recording shapes

`trajector enable` installs one of two shapes in a project, and you choose which at enable time.

| Command                       | What records                                                        | Remote Control                                                    |
| ----------------------------- | ------------------------------------------------------------------- | ----------------------------------------------------------------- |
| `trajector enable`            | The recording proxy, the project's session files, and git observation | `/remote-control` is not available inside the project             |
| `trajector enable --no-proxy` | The project's session files and git observation only; no base URL is injected | `/remote-control` stays available inside the project |

`enable` states which shape it installed, and `status` and `doctor` say which shape a project records in. With `trajector enable`, `/remote-control` is unavailable inside the project, but `claude remote-control` still works and both sources are still recorded.

A third flag is orthogonal to the shape: `trajector enable --no-earlier` leaves the session files the project already has alone — they are neither registered nor read — so only the sessions that run from now on are collected. It can be combined with `--no-proxy`, and neither flag implies the other. The choice is recorded on the grant, and `status` states it under the project together with the way back:

```
  Earlier sessions were skipped at enable; run `trajector enable` again (with --no-proxy if you use it) to collect them.
```

{% hint style="info" %}
**Desktop Claude Code cannot go through the proxy.** The desktop host sets its own API base URL and ignores the project settings the CLI writes, so there is no proxy to route it through. Those sessions are still recorded from their session files, and from 0.3.1 a session file recorded live counts at the full rate — see Rewards.
{% endhint %}

### Verifying a release yourself

Every release is built by GitHub Actions from the public repository and ships with a checksum file and a build provenance attestation:

```sh
shasum -a 256 -c --ignore-missing trajector_checksums.txt
gh attestation verify trajector_* --repo PublicAI01/trajector-cli
```

The attestation proves the binary was produced by that workflow from that source, not merely that it downloaded intact.

### Updating

```sh
trajector upgrade
```

`upgrade` moves to the newest published release — including pre-releases, which is what a beta wants. The archive's checksum is verified **before** anything is replaced: a download that fails verification leaves the binary you have exactly as it was.

If a package manager owns the installation, `upgrade` says so and hands the job back to that manager rather than overwriting its files.

A release that changes the data agreement bumps its version, and recording pauses until you reconfirm. After upgrading, the next `trajector enable` shows the updated agreement and asks you to accept it once; forwarding is untouched while the pause stands.

0.3.3 is such a release: the agreement moves to version `2026-09-21`, so upgrading to it asks you to confirm the agreement once.

### Uninstalling

```sh
trajector uninstall
```

{% hint style="danger" %}
Run `trajector uninstall` **before** deleting the binary. Deleting the binary alone leaves the settings injections behind in every project you enabled, and Claude Code will keep trying to reach a proxy that no longer exists.
{% endhint %}

`uninstall` removes every injection, revokes the device token, and stops the proxy. Add `--delete-data` to also delete the local spool and any captured data still waiting on your machine. It ends by telling you the path of the binary to delete.
