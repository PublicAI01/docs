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
| `TRAJECTOR_VERSION`     | Pin a release, e.g. `TRAJECTOR_VERSION=v0.4.0` |

```sh
TRAJECTOR_INSTALL_DIR=/usr/local/bin TRAJECTOR_VERSION=v0.4.0 \
  sh -c "$(curl -fsSL https://raw.githubusercontent.com/PublicAI01/trajector-cli/main/install.sh)"
```

`TRAJECTOR_VERSION` takes a release's tag as the [releases page](https://github.com/PublicAI01/trajector-cli/releases) lists it. The service accepts uploads only from recent releases, so a pinned version falls out of use over time.

{% hint style="warning" %}
**On macOS, install with the command above rather than downloading the archive in a browser.** Releases are not code-signed yet, and macOS marks anything a browser downloaded. Unpacking such an archive in Finder passes that mark to the binary, and Gatekeeper then refuses to run it. A download made by `curl` carries no mark.
{% endhint %}

#### Homebrew

On macOS, Homebrew works too:

```sh
brew install publicai01/tap/trajector
```

A Homebrew install is updated by `brew upgrade trajector`. Neither `trajector upgrade` nor the automatic check replaces a binary Homebrew manages: when a newer release is out, the next trajector command you run says so once per release and names that command.

### Windows

There is no Windows build yet — the [releases page](https://github.com/PublicAI01/trajector-cli/releases) carries macOS and Linux archives only, until that platform has been tested end to end against the service.

Run trajector under [WSL](https://learn.microsoft.com/windows/wsl/install): install a Linux distribution, then run the same install command above inside it.

Scoop is not available.

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
**Desktop Claude Code cannot go through the proxy.** The desktop host sets its own API base URL and ignores the project settings the CLI writes, so there is no proxy to route it through. Those sessions are still recorded from their session files, and a session file recorded live counts at the full rate — see Rewards.
{% endhint %}

### Verifying a release yourself

Every release is built by GitHub Actions from the public repository and ships with a checksum file and a build provenance attestation:

```sh
shasum -a 256 -c --ignore-missing trajector_checksums.txt
gh attestation verify trajector_* --repo PublicAI01/trajector-cli
```

The attestation proves the binary was produced by that workflow from that source, not merely that it downloaded intact.

### Updating

Trajector keeps itself up to date. The session hooks ask GitHub for the list of releases at most once a day — at most once an hour while the trajector service answers that this version is too old — and when a newer one exists, trajector downloads it, verifies it against the release's checksum file, and installs it in the background. The request carries the trajector version and nothing else; Data and privacy says exactly what it sends. The next trajector command you run says that it happened:

```
trajector upgraded itself to <version> in the background.
```

and the session hooks of every enabled project are completed with any hook the new release adds. `status` and `doctor` show when the last check ran and what it found, or why automatic upgrades are off.

The automatic check starts only once you have accepted a data agreement that describes it — version `2026-10-13` or later — and it never runs in a build made from source. To turn it off, set `"auto_upgrade": false` in `config.json` (see Files, configuration, and environment): then nothing goes to GitHub unless you run `trajector upgrade` yourself.

Releases before 0.4.0 do not upgrade themselves. To move to the newest release from one of them, run it by hand once:

```sh
trajector upgrade
```

Or run the install command again. It works from any version, including one that has no `upgrade` command or whose `upgrade` fails, and installs the newest release over the one you have, verified the same way:

```sh
curl -fsSL https://raw.githubusercontent.com/PublicAI01/trajector-cli/main/install.sh | sh
```

If you installed into another directory, set `TRAJECTOR_INSTALL_DIR` to it again; otherwise you end up with a second copy in `~/.local/bin`. If Homebrew owns the installation, run `brew upgrade trajector` instead. `trajector version` then prints the version you have.

`upgrade` moves to the newest published release — including pre-releases, which is what a beta wants. The archive's checksum is verified **before** anything is replaced: a download that fails verification leaves the binary you have exactly as it was. The new release is also run once, and made to start a proxy of its own, before it is kept; if either fails, the previous binary is put back and the command says why and exits `1`.

If a package manager owns the installation, `upgrade` says so and hands the job back to that manager rather than overwriting its files.

A release that changes the data agreement bumps its version, and recording pauses until you reconfirm. After upgrading, the next `trajector enable` shows the updated agreement and asks you to accept it once; forwarding is untouched while the pause stands. Run it in any project you enabled, with the options that project was enabled with (`--no-proxy`, `--no-earlier`): `enable` installs the recording shape you give, and without `--no-earlier` it collects the session files the project had before it was first enabled.

0.5.0 is such a release: the agreement moves to version `2026-10-14`. Recording pauses after the upgrade until you run `trajector enable` once (with `--no-proxy` and `--no-earlier` if you use them) and confirm it.

### Uninstalling

```sh
trajector uninstall
```

{% hint style="danger" %}
Run `trajector uninstall` **before** deleting the binary. Deleting the binary alone leaves the settings injections behind in every project you enabled, and Claude Code will keep trying to reach a proxy that no longer exists.
{% endhint %}

`uninstall` removes every injection and the project-discovery hint, and stops the proxy. It then asks whether to delete local data — captured records still waiting on your machine, configuration, and the device token; `--delete-data` answers yes without asking. It ends by telling you the path of the binary to delete.

`uninstall` does not revoke the device token with the service. To do that, run `trajector logout` first.
