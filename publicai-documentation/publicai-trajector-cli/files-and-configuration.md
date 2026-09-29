# Files and configuration

## Files, configuration, and environment

### Where trajector keeps its files

Everything lives under per-user directories your platform dictates. Nothing is written outside them except the injection inside a project you enabled.

| Platform | Config                                    | Data                       | State                      |
| -------- | ----------------------------------------- | -------------------------- | -------------------------- |
| Linux    | `~/.config/trajector`                     | `~/.local/share/trajector` | `~/.local/state/trajector` |
| macOS    | `~/Library/Application Support/trajector` | same                       | same                       |
| Windows  | `%APPDATA%\trajector`                     | `%LOCALAPPDATA%\trajector` | `%LOCALAPPDATA%\trajector` |

`XDG_CONFIG_HOME`, `XDG_DATA_HOME`, and `XDG_STATE_HOME` override these on **every** platform, which is what makes a fully relocated sandbox possible.

#### What is in them

| File or directory     | Where  | Holds                                                                 |
| --------------------- | ------ | --------------------------------------------------------------------- |
| `config.json`         | config | Your own settings — see below                                         |
| `proxy_projects.json` | config | The routing table: which token routes where, and what may be recorded |
| `consent.json`        | config | The durable record of what you agreed to                              |
| `secrets/`            | config | The device token, when no OS keyring is available (owner-only files)  |
| `rawcalls/`           | data   | The spool: captured calls waiting to upload, in per-day directories   |
| `rawcalls-rejected/`  | data   | Quarantined batches, each with its records and a `reason.json`        |
| `upload/`             | data   | Upload bookkeeping: what was acknowledged, what the service last said |
| `proxy.log`           | state  | The proxy's log                                                       |

Captured records are stored with the project identified only by a **hash** of its root path. Credential headers are never in any of these files.

### Per-project files

`trajector enable` writes to the project's own Claude Code settings — the project-local file, not the shared one — and prints the path. It also adds the injected settings file and the diagnostic-bundle pattern to `.gitignore` so neither is ever committed.

`trajector disable` and `trajector uninstall` remove the injection again.

### `config.json`

Optional. It does not exist until you create it.

```json
{
  "platform_url": "https://api.trajector.example",
  "releases_url": "https://api.github.com/repos/PublicAI01/trajector-cli/releases",
  "upload_images_and_documents": false
}
```

| Key                           | Meaning                                                                                                   |
| ----------------------------- | --------------------------------------------------------------------------------------------------------- |
| `platform_url`                | Where captured data is uploaded                                                                           |
| `releases_url`                | Where `trajector upgrade` looks for a newer build                                                         |
| `upload_images_and_documents` | `false` uploads each image and document as a placeholder instead of its content. Absent means `true`: they are uploaded |

`upload_images_and_documents` exists because redaction cannot see what an image or a PDF shows. The next upload uses it, also for records captured before you changed it, and the proxy does not need a restart. `trajector status` shows which of the two is in effect. What a placeholder holds and the one copy the setting does not cover are in Data and privacy.

{% hint style="danger" %}
**None of these can be set from an environment variable, by design.**

Settings a repository ships reach a session hook's environment. If the upload destination, the source of the next binary, or what is uploaded could be chosen that way, a cloned repository could redirect your credentialed traffic, hand you a binary of its choosing, or turn image upload back on. Every answer comes from this file, in your own config directory, where nothing inside a repository can write.

`install.sh` is the deliberate exception: `TRAJECTOR_API_BASE` and `TRAJECTOR_DL_BASE` do work there, because that script is only ever run by a person at a shell, never from a hook.
{% endhint %}

A `config.json` that exists but cannot be read — cut short, not JSON, not readable by your user — is a loud failure, not a silent fallback. Every command stops on it rather than quietly sending data somewhere you did not intend, with two exceptions: `trajector status` and `trajector doctor bundle` still run, name the file and the failure, and say that images and documents go up as placeholders until the file reads again. A proxy that is already running when the file breaks reads it again for every batch: until it reads, images and documents go up as placeholders, and `proxy.log` says so. A proxy does not start on such a file.

```
Device
  error: Images and documents: placeholders only, because config.json cannot be read.
    why:  reading /home/you/.config/trajector/config.json: unexpected end of JSON input; images and documents go up as placeholders until it reads.
  Signed in.
```

Placeholders are the side a wrong guess costs least on: an upload cannot be taken back, a placeholder only withholds content. `status` exits `1` while the file cannot be read.

### Environment variables

| Variable                                               | Effect                                                               |
| ------------------------------------------------------ | -------------------------------------------------------------------- |
| `TRAJECTOR_TOKEN_STORE=file`                           | Store the device token in owner-only files instead of the OS keyring |
| `TRAJECTOR_PROXY_ADDR`                                 | Override the proxy address; for tests and unusual setups             |
| `XDG_CONFIG_HOME` / `XDG_DATA_HOME` / `XDG_STATE_HOME` | Relocate trajector's directories, on every platform                  |

Variables read from the surrounding Claude Code configuration, which trajector reacts to but never sets for you:

| Variable                                             | Effect on trajector                                                           |
| ---------------------------------------------------- | ----------------------------------------------------------------------------- |
| `ANTHROPIC_BASE_URL`                                 | Your own relay. Trajector forwards to it and marks records third-party origin |
| `CLAUDE_CODE_USE_BEDROCK` / `CLAUDE_CODE_USE_VERTEX` | Unsupported channels. `doctor` reports that the project is not being captured |

### The proxy

| Property       | Value                                                                    |
| -------------- | ------------------------------------------------------------------------ |
| Address        | `127.0.0.1:41100`, loopback only, never any other interface              |
| Lifetime       | Started on demand by the session hook; exits after 30 minutes of silence |
| Started by you | No. There is no daemon, no service unit, nothing to enable at boot       |

A newer build takes over the port from an older one at the next session; the older proxy drains what it is carrying before exiting, so requests in flight are not disturbed.
