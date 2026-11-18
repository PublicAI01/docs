# Share a past project

A company we work with is looking for complete software projects with a real development history, for AI training, and we are helping them find good candidates. Projects that are picked get paid, depending on the project.

This page says what fits, and how to tell us about a project with one command. The command sends facts about the repository's history. It sends no code.

The short version:

> **In the repository, run `trajector repo assess --submit --preview=request.json`. It writes everything it would send to `request.json`, shows its terms, and sends only after you confirm.**

It needs trajector 0.7.0 or later and a paired device. It does not need an enabled project, and your Trajector account and rewards are not affected either way.

***

### What fits

* A complete application (web, mobile, desktop, game, tool and so on), not a practice project or a code snippet
* A real development history in git; launched or used commercially is a plus
* Built by you or your team, so that you may share it

If you have more than one project, run the command once in each repository.

### Before you start

1. Install trajector, or update it: `trajector version` prints the version you have, and `trajector upgrade` installs the newest release. See [Installation](installation.md).
2. Pair the device if it is not paired yet: `trajector login`.
3. Use a clone that has the full history. A shallow clone cannot be submitted; `git fetch --unshallow` completes it.

### Step 1: look at the readout

```
cd /path/to/your/repo
trajector repo assess
```

This prints a readout and ends. Nothing is written and nothing is sent.

```
Repository readout, rule version 1, as of 2026-11-18T09:11:10Z
Read from git history and file paths only. Nothing was sent.

  commits               439         commits reachable from HEAD, merges left out
  first_commit          2026-08-03  author date of the earliest commit (UTC)
  span_years            0.29        years from the earliest to the latest commit
  authors               5           distinct author email addresses
  test_commits          338         commits that change a test path
  src_commits           368         commits that change an implementation file
  cochange              318         commits that change tests and implementation together
  ...
  main_lang             go          language of the most implementation files
  has_ci                true        the tree has CI configuration
```

The readout is counts, shares and yes-or-no values, each with a line that says what it counts. It does not say whether the project fits. `trajector repo assess --json` prints the same readout as JSON, and `trajector repo assess /path/to/your/repo` reads a repository other than the current directory.

### Step 2: submit it

```
trajector repo assess --submit --preview=request.json
```

In order, the command:

1. Prints the readout again.
2. Asks four questions, listed below. Answer each with its number. If you answer the first with "Someone else owns it" or "I am not sure", the command stops there and sends nothing.
3. Writes everything it would send to `request.json` in the current directory, indented for reading, and prints a summary: how many commits and changed paths it holds, and its size. Open the file and read it before you go on.
4. Shows the repository assessment terms in full and asks `Do you agree to these terms? [yes/no]:`. This happens the first time, and again only when the terms change. They are separate from the data contribution agreement.
5. Asks `Send this submission now? [yes/no]:`. Anything but `yes` sends nothing and leaves `request.json` where it is.
6. Sends the submission once and prints its id:

```
Sent. Submission id: <id>
If trajector wants to talk about this repository, it writes to the email address of your trajector account.
To have this submission deleted, email support@publicai.io with the id <id>.
```

Keep the id. `request.json` is the record of what was sent; trajector does not delete it, so move it out of the repository or leave it out of your commits.

{% hint style="info" %}
`--preview=<file>` is required, and the file must not exist yet: trajector never writes over a file. To submit again, name a new file or delete the old one first.
{% endhint %}

#### The four questions

| Question                                    | Answers                                                                                                                                                                       |
| ------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Who owns this repository?                   | I wrote it on my own · A team wrote it, and I can get the consent of everyone on the team · It belongs to a company, and I can act for the company · Someone else owns it, and I cannot act for them · I am not sure |
| Was this code ever public?                  | Never · Part of it was public at some time · All of it was public at some time                                                                                                |
| Does this software run in production?       | Yes, with paying users · Yes, with users who do not pay · Only inside an organization · No, it has not launched                                                               |
| About how much of the code did an AI write? | Less than 5% · 5% to 30% · 30% to 70% · More than 70% · I do not know                                                                                                         |

### What is sent

| Sent                                                                                                                                     | Not sent                                                    |
| ---------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------- |
| The readout from step 1                                                                                                                  | File contents                                               |
| The id of the repository's first commit and of the current commit                                                                        | Diffs                                                       |
| For each remote, only the kind of host: `github`, `gitlab`, `bitbucket` or `other`                                                       | Remote URLs and host names                                  |
| For each commit: its id, its author time, its message, and the paths it changed with the number of lines added and deleted               | Author names and email addresses; an author is sent as a number |
| Your four answers                                                                                                                        | Where the repository is on your machine                     |

Commit messages and paths pass the same local masking as every other trajector upload: secrets, email addresses and phone numbers are masked by their form. Message lines that name a person, such as `Co-authored-by:` and `Signed-off-by:`, are left out. Any other name, path, URL or word written in a commit message is sent as it was written, because masking cannot know which of them are confidential to you. `request.json` shows what remains, so that is the place to check.

If any of the code is public, the commit ids can identify it.

The field-by-field list is in the client's [PRIVACY.md](https://github.com/PublicAI01/trajector-cli/blob/main/PRIVACY.md#repository-assessment).

### What happens next

* We read each submission to assess the project. If it looks like a fit, we write to the email address of your Trajector account with the details.
* A submission is not an offer, and it commits neither side to anything. It gives no one any right to your code. A sale or a license needs a separate written agreement.
* A submission stays with us. We use it to assess the repository and to decide whether to contact you, and in aggregate, in a form that identifies no repository and no person, to check how we assess repositories. We do not pass it on without asking you first.
* The service takes up to 10 submissions from one account in a UTC day.

### Deleting a submission

Email `support@publicai.io` from the email address of your Trajector account, with the submission id the command printed or the id of the repository's first commit. We then delete that submission. If you ask us to close your account, we delete all of your submissions.

### If the command stops

Every one of these ends with nothing sent.

| What it says                                                                 | What to do                                                                            |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| `--submit needs --preview=<file>`                                            | Add `--preview=request.json`                                                          |
| `request.json already exists; name a new file`                               | Name another file, or delete the old one                                              |
| `this device is not paired`                                                  | Run `trajector login`, then run the command again                                     |
| `this repository is a shallow clone`                                         | Run `git fetch --unshallow`, then run the command again                               |
| `Only someone who may share this repository can submit it`                   | The answer to the first question was that the repository is not yours to share        |
| `the service needs a newer trajector`                                        | Run `trajector upgrade`, then run the command again                                   |
| `the service asked to try again later`                                       | Wait and run it again; this is also the answer once the day's 10 submissions are used |

Nothing is kept on your machine to retry a submission that did not go through. Run the command again with a new preview file.
