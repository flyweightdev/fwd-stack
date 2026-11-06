# fwd-stack

Our fork of [tstack](https://github.com/t3dotgg/tstack).

The agent skills I use every day, cleaned up so anyone can use them. They work
in Claude Code and Codex. Some use [T3 Code](https://github.com/pingdotgg/t3code)
features when they are available and fall back to native tools when they are
not.

## Skills

| Skill | Say | What it does |
|---|---|---|
| `full-send` | "full send" | Takes a change to a merged PR with nobody watching: proves it, reviews it, files it, babysits it, merges it. |
| `full-stop` | "full stop" | Same as `full-send`, but stops when the PR is ready for you to merge. |
| `council` | "council" | Gets review from two models, Claude and Codex, and loops until they agree. |
| `sol-loop` | "sol loop" | GPT Sol reviews and Claude fixes, in a loop until Sol finds no more errors. |
| `sol-review` | "sol review" | Gets one review from GPT Sol and posts the findings as inline PR comments. Changes no code. |
| `show-proof` | (automatic) | Takes a screenshot of visible changes, checks it, and shows it in the T3 thread when the task is done. |
| `file-pr` | "file a PR" | Files a short PR that leads with the problem, not an inventory of changes. |
| `takeover` | "take over" | Finishes someone else's PR, branch, or agent thread, and keeps their credit. |
| `babysit-pr` | "babysit" | Watches a PR through CI and review bots, and fixes the real findings. |
| `modernize` | "modernize" | Rebases an old PR and decides if it should still exist. |
| `leaving-pr-comments` | (automatic) | Formats comments that an agent posts on your behalf. |

Some skills call others. Install the ones they need too:

- `full-send` uses `sol-loop`, `file-pr`, and `babysit-pr`.
- `full-stop` uses `full-send`.
- `sol-review` uses `sol-loop` and `leaving-pr-comments`.
- `takeover` uses `modernize`, `file-pr`, `babysit-pr`, and `leaving-pr-comments`.
- `file-pr`, `babysit-pr`, and `modernize` use `leaving-pr-comments`.

## Install

```bash
git clone https://github.com/flyweightdev/fwd-stack
cp -R fwd-stack/skills/* ~/.claude/skills/   # Claude Code
cp -R fwd-stack/skills/* ~/.codex/skills/    # Codex
```

## Requirements

- An authenticated [GitHub CLI](https://cli.github.com) (`gh`).
- `python3` for `babysit-pr`'s `pr-wait` script.
- `council`, `sol-loop`, and `sol-review` need subagents on other providers'
  models. T3 Code does this with `delegate_task`. Otherwise you need a harness
  that can run those models.

## Make them yours

These are starting points. Edit them. Put personal settings, like which models
sit on your council, in your `CLAUDE.md` or `AGENTS.md`, so you can update the
skills without losing them.

## Fork

Push to `origin` (`flyweightdev/fwd-stack`). Never push to the original tstack.
We only pull from it, as `upstream`.

Set this up once in each clone. Git does not copy it with the repo:

```sh
git remote add upstream https://github.com/t3dotgg/tstack
git remote set-url --push upstream no_push
gh repo set-default flyweightdev/fwd-stack
```

To pull updates from tstack:

```sh
git pull --no-rebase upstream main
git push origin main
```

Merge rather than rebase, so `main` never needs a force-push.

## License

[MIT](LICENSE)
