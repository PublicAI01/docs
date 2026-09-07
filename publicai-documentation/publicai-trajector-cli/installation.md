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
| `TRAJECTOR_VERSION`     | Pin a release, e.g. `TRAJECTOR_VERSION=v0.2.1` |

```sh
TRAJECTOR_INSTALL_DIR=/usr/local/bin TRAJECTOR_VERSION=v0.2.1 \
  sh -c "$(curl -fsSL https://raw.githubusercontent.com/PublicAI01/trajector-cli/main/install.sh)"
```

{% hint style="warning" %}
**On macOS, install with the command above rather than downloading the archive in a browser.** Releases are not code-signed yet, and macOS marks anything a browser downloaded. Unpacking such an archive in Finder passes that mark to the binary, and Gatekeeper then refuses to run it. A download made by `curl` carries no mark.
{% endhint %}

### Windows

There is no Windows build yet — the [releases page](https://github.com/PublicAI01/trajector-cli/releases) carries macOS and Linux archives only, until that platform has been tested end to end against the service.

Run trajector under [WSL](https://learn.microsoft.com/windows/wsl/install): install a Linux distribution, then run the same install command above inside it.

Homebrew is not available yet.

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

### Uninstalling

```sh
trajector uninstall
```

{% hint style="danger" %}
Run `trajector uninstall` **before** deleting the binary. Deleting the binary alone leaves the settings injections behind in every project you enabled, and Claude Code will keep trying to reach a proxy that no longer exists.
{% endhint %}

`uninstall` removes every injection, revokes the device token, and stops the proxy. Add `--delete-data` to also delete the local spool and any captured data still waiting on your machine. It ends by telling you the path of the binary to delete.
