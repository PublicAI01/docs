# Reasoning summaries

From **TBD-CUTOVER**, a session recorded without reasoning summaries is not rewarded. It is still recorded and uploaded, but it earns nothing.

This page is the rule. It says what counts as a session with reasoning summaries, and how to turn them on for each way you run Claude Code. Every message trajector, the upload service and the dashboard show you about reasoning summaries links here.

The short version:

> **Turn reasoning summaries on before you start a session — in a terminal with `showThinkingSummaries`, in the desktop app with the Thinking transcript view, and with `--thinking-display summarized` for `claude -p` and the Agent SDK.**

***

### What reasoning summaries are

Before it answers, the model reasons. Since v2.1.89, Claude Code no longer returns that reasoning as readable text by default: what comes back is empty, apart from an occasional one-line progress note. Turning summaries on makes Claude Code return a summary of the reasoning instead, and that text is what makes a session worth something to the buyer of the data.

Turning them on does not change your token usage.

### The rule

A session has reasoning summaries when, in its main conversation, **at least half of the model's responses that contain reasoning carry reasoning text**.

* **Main conversation only.** Subagents are not counted: whether a subagent's model writes summaries is not something you control.
* **At least three responses with reasoning.** A session with fewer is too short to tell, and this rule does not decide it; the other checks on the [Rewards](rewards.md) page still apply.
* **Progress notes alone don't reach half.** With summaries off, Claude Code still returns a short progress note now and then. Those notes are too sparse to get a session over the line.
* **Encrypted reasoning counts as a response without text.**
* **The whole session is judged, and judged again as it grows.** A session that is not rewarded under this rule stays that way, even if later responses carry summaries.
* **It applies to sessions first uploaded on or after TBD-CUTOVER.** A session that had started uploading before then is not affected by this rule, however long it runs.

Two consequences follow from the rule:

1. **Turn it on before you start a session.** Whether a session has summaries is decided over the whole session, so switching them on halfway doesn't rescue the session you are in — the next one counts.
2. **It applies to every way of running Claude Code.** If your setup can't produce reasoning summaries, its sessions aren't rewarded. There is no exemption by how you run Claude Code.

***

### Turn them on

#### Claude Code in a terminal

The switch is Claude Code's `showThinkingSummaries` setting. Either:

* **Run `trajector enable` in the project and answer yes** when it asks about `showThinkingSummaries`. Add `--no-proxy` and `--no-earlier` if the project uses them. This turns the setting on for that project only, in the project's local Claude Code settings, and `trajector disable` puts it back.
* **Or set it yourself for every project**, in your user settings file `~/.claude/settings.json`:

  ```json
  {
    "showThinkingSummaries": true
  }
  ```

  trajector never writes this file. It is your change to make and to undo.

Then start a new session.

#### Claude desktop app

The desktop app does not use `showThinkingSummaries`. Its switch is the **transcript view**: summaries are produced while a session's view is **Thinking** or **Verbose**.

* **For every new session:** Settings → **Default transcript view** → **Thinking**.
* **For one session:** press **Ctrl+O**, or pick **Transcript view** from the session's menu, and choose **Thinking** before your first message. The app confirms with _"Thinking summaries will appear from the next response."_

What it costs you, honestly:

* The Thinking view shows the summaries in the transcript, so the transcript gets longer.
* Switching a session back to **Normal** stops the summaries about three minutes later. Responses from then on have none, and they count against the session.

**If you also use Claude Code in a terminal**, keep `showThinkingSummaries` on there. Whether it is set for a project or in `~/.claude/settings.json`, it does not get in the way of the desktop app's Thinking view.

**If the Thinking view shows no summaries**, the desktop app has not made them available to your account, and its sessions cannot have them. Record the sessions you contribute with Claude Code in a terminal instead. (On such an account, `showThinkingSummaries` also hides the desktop app's short progress notes; it does not bring summaries.)

#### `claude -p` and the Agent SDK

Neither reads `showThinkingSummaries`. Ask for summaries on each run:

```sh
claude -p "your prompt" --thinking-display summarized
```

In the Agent SDK, set `display` in the `thinking` option:

```ts
query({
  prompt: "your prompt",
  options: { thinking: { type: "adaptive", display: "summarized" } },
});
```

***

### Check that it worked

* **Your dashboard's My data page** shows how many of your sessions in the last seven days had reasoning summaries, grouped by how you ran Claude Code, and marks each session. It counts the same way this rule does; sessions too short to tell are listed apart.
* **`trajector status`** in the project counts the assistant lines of its session files that carry no reasoning:

  ```
    12 of 40 assistant lines carry no reasoning.
  ```

* **In the session itself**, when most of a session's reasoning carries no text, trajector says so once a day per project, with the step for the way you ran it:

  ```
  trajector: no reasoning summaries in this session; run trajector enable, then start a new session
  trajector: no reasoning summaries in this session; set transcript view to Thinking, then start anew
  trajector: no reasoning summaries in this session; add --thinking-display summarized and rerun
  trajector: no reasoning summaries in this session; run trajector status to see how to turn them on
  ```

  It never blocks you. These counts are trajector's reading on your machine; the rule above is applied on the server.

### When this takes effect

* **From TBD-NOTICE-START to TBD-CUTOVER:** nothing changes for your rewards. Sessions without reasoning summaries are marked on your dashboard, and the upload service's message in `trajector status` says how many of each upload had none, so you can fix your setup in time.
* **From TBD-CUTOVER:** sessions first uploaded from then on without reasoning summaries are not rewarded. They show as rejected in My data — see [Rewards](rewards.md).
* **Nothing is retroactive.** Sessions uploaded before TBD-CUTOVER, and rewards already credited, are not touched.

trajector TBD-CLI-VERSION and later ask for `showThinkingSummaries` as a required setting. Its data agreement changes with it, so after upgrading, the next `trajector enable` asks you to confirm the updated agreement once — and asks about this setting at the same time.
