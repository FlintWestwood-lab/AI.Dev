---
name: new-project
description: "Start a new project: scaffold a folder, create the matching GitHub repo under the same name, and wire in the idea-to-ship config the engineering skills expect."
disable-model-invocation: true
---

# New Project

One name becomes three things: the **folder** under `$HOME`, the **GitHub repo**
under your own account, and the project name inside the scaffolded files.

The output is an empty but fully wired project: no app code, a glossary waiting
to be filled, and `START-HERE.md` describing the route from idea to ship.

## Input

The project name, e.g. `/new-project sleeping-app`.

If the user didn't give one, ask for it. Validate before doing anything:

- Lowercase letters, digits, hyphens and underscores only
- Not already a directory at `$HOME/<name>`
- Not already a repo on their account (`gh repo view <name>` returning success means it exists)

If any check fails, stop and say which one. Don't invent a variant name.

## Step 1: Auth preflight, before touching anything

**Do this first.** Auth failures here are slow to diagnose and cost far more
than the 10 seconds this check takes.

```bash
gh auth status
```

**If it reports a valid login, continue to Step 2.**

If it reports an invalid or expired token, the fix is a full logout then a
browser login. A logout first matters: without it `gh` may keep serving the
stale entry.

```bash
gh auth logout -h github.com -u <account>
gh auth login
```

Tell the user to pick `GitHub.com` → `HTTPS` → **`Login with a web browser`**,
and to say yes to authenticating Git. This is interactive, so **they** run it,
not you. Suggest they type it with a `!` prefix so the output lands in the
session. Browser OAuth is the recommendation over pasting a token: it needs
nothing copied, nothing expires in 30 days, and it writes a valid token to the
one place the git credential helper reads.

### The credential-helper trap

If `git push` fails with `Invalid username or token` **without ever prompting
for a username**, do not start generating new tokens. Something handed git a
dead credential automatically. Diagnose in this order:

```bash
git config --get-urlmatch credential https://github.com
```

If that returns `!/usr/bin/gh auth git-credential`, git delegates all GitHub
auth to `gh`, and a stale `gh` token breaks every push while git stays silent
about it, because as far as git is concerned the helper *succeeded*. Fixing
`gh` (above) fixes git too. That is the whole bug, and it is the single most
likely cause.

Two traps worth knowing when reading that output:

- A **URL-scoped** key (`credential.https://github.com.helper`) always beats a
  generic `credential.helper`, **regardless of scope**. A repo-local
  `credential.helper=store` does not override a global URL-scoped helper.
- Unsetting a key without `--global` only touches repo-local config. If the
  helper lives in `~/.gitconfig`, a local unset silently does nothing.

Also check nothing is shadowing the login: `GH_TOKEN` and `GITHUB_TOKEN` in the
environment take precedence over what `gh auth login` stores.

If a token is involved at any point, verify it before use rather than guessing.
Have the user run this; input is hidden, so the token never enters the
transcript:

```bash
read -rs -p "token: " T && echo && curl -sS -o /dev/null -w 'HTTP %{http_code}\n' \
  -H "Authorization: Bearer $T" https://api.github.com/user; unset T
```

`200` means the token is fine and the problem is elsewhere. `401` means the
token is dead: **`401` is the token being rejected outright, not a permissions
problem**, which would be `403`. Generate a fresh one instead of retrying it.

## Step 2: Scaffold

Resolve the account name rather than hardcoding it:

```bash
gh api user --jq .login
```

Create `$HOME/<name>`, then copy every file from this skill's `template/`
directory, applying this mapping:

| Template | Destination |
| --- | --- |
| `CLAUDE.md` | `CLAUDE.md` |
| `CONTEXT.md` | `CONTEXT.md` |
| `README.md` | `README.md` |
| `START-HERE.md` | `START-HERE.md` |
| `gitignore` | `.gitignore` |
| `claude-settings.json` | `.claude/settings.json` |
| `scratch-README.md` | `.scratch/README.md` |
| `docs/adr-README.md` | `docs/adr/README.md` |
| `docs/agents/*.md` | `docs/agents/*.md` |

The dotfiles are stored undotted in `template/` on purpose: a real `.gitignore`
inside the template directory would apply to this repo.

Then:

- Replace every `{{PROJECT_NAME}}` with the project name
- Symlink `AGENTS.md` to `CLAUDE.md` (`ln -s CLAUDE.md AGENTS.md`)
- Create the empty `.scratch/` and `docs/adr/` directories

**Leave `CONTEXT.md` and `docs/adr/` empty of real content.** That is the
correct starting state, not an omission. `docs/agents/domain.md` tells the
skills to proceed silently when they are bare, and `grill-with-docs` fills them
lazily as terms and decisions actually get resolved. A glossary invented upfront
is worse than no glossary.

## Step 3: Commit and publish

```bash
cd "$HOME/<name>"
git init -b main
git add -A
git commit -m "Scaffold idea-to-ship template"
gh repo create <name> --private --source=. --remote=origin --push
```

Default to `--private`. Ask first only if the user has said anything suggesting
they want it public.

`gh repo create --source` creates the repo empty and pushes, so there is no
README to reconcile. If you instead attach a remote to a repo the user already
created through the web UI, it may carry an auto-generated README. That makes
the first push a non-fast-forward. Resolve it with `git pull --rebase origin
main`, never `--force`.

## Step 4: Report

Give the user the repo URL and these three lines:

1. `cd` into the new folder and open a session there.
2. Run `/grill-with-docs` and describe the project. It interviews them, and
   writes terminology into `CONTEXT.md` and decisions into `docs/adr/`.
3. `START-HERE.md` has the full route; `/ask-matt` routes them if they forget.

## Notes

The issue tracker is set to **local markdown** (`.scratch/`), which suits a
fresh repo and needs no auth. To switch to GitHub Issues later, rewrite
`docs/agents/issue-tracker.md`, or tell the user to run
`/setup-matt-pocock-skills` and pick a different tracker. Nothing else changes:
every skill reads that one file to decide whether to run `gh issue create` or
write a file locally.

One caveat if they switch: `to-tickets` wants **native blocking links**, which
are a GraphQL-side feature, and fine-grained personal access tokens are poorly
supported by GitHub's GraphQL API. If blocking edges fail, re-auth with browser
OAuth rather than fighting the token.

The three files under `template/docs/agents/` are copies of the seed templates
owned by `setup-matt-pocock-skills`. If that skill's seeds change upstream,
re-copy them.

The scaffolded `.claude/settings.json` declares the `mattpocock-skills` plugin
at project scope. Plugins are normally enabled at **user** scope, so they follow
you into any directory already; the project-scope declaration is what makes the
scaffold portable to another machine.
