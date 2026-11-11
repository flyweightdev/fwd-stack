---
name: sol-review
description: Use when the user asks for a review with Sol, or says "sol review". Sol reviews once and findings go on the PR as inline comments. Changes no code. To fix findings, use sol-loop.
---

# sol-review

Get one independent review of the current change from GPT Sol. Post the
findings as inline comments if there is a PR, then report them. This never
changes code.

## 1. Scope

Run `git status --short`, then `git fetch origin`. Find the base:

- PR: `gh pr view <n> --json baseRefName`, then
  `git merge-base HEAD origin/<baseRefName>`.
- Local: `git merge-base HEAD origin/<default-branch>`. No remote: a base the
  user names, or `HEAD` for uncommitted changes only.

The change is base to working tree, with untracked files. If it is empty, say
so and stop. State the intent in 1-2 sentences. Include any focus the user
named, such as security or migrations.

## 2. Start Sol

Use GPT 6.1 Sol (`gpt-6.1-sol`) at high reasoning effort, unless the user names
another version. Start one reviewer as described in "Start Sol" in
[sol-loop](../sol-loop/SKILL.md), which covers the T3 and native routes,
waiting, and failures. If no route runs this model, report the blocker. Do not
substitute a model. Use this brief:

```
Review only. Do not edit, commit, push, comment on PRs, or start subagents.
Repo: <abs path> Base: <sha> Intent: <intent> Focus: <focus or none>
See: git diff <base>; git status --short (read untracked).
Read every changed and untracked file in full (no head/tail truncation),
plus the callers it touches. A skim is not a review.
Read repo guidance (CLAUDE.md, AGENTS.md). Repo guidance
and the focus above override these defaults.

You are reviewing a change by another engineer. Flag an issue only if all hold:
- It really hurts correctness, performance, security, or maintainability.
- It is one specific, fixable problem, not a general complaint.
- This change introduced it. Do not flag existing bugs.
- The fix asks for no more rigor than the rest of the codebase has.
- The author would fix it if they knew, and did not clearly intend it.
- It needs no unstated assumptions. If you claim other code breaks, name that
  code.

List every issue that qualifies. If none do, return none. Do not pad the list.
Skip style unless it hides meaning or breaks a documented rule.

For each issue, one short paragraph: why it is a bug, and the inputs or
conditions that trigger it. State the severity honestly. Plain tone, no praise.
No code over 3 lines. Keep line ranges under 10 lines. One entry per issue.

Reply:
VERDICT: APPROVE or CHANGES
FILES READ: <list>
#1 <title>
<paragraph>
File: <path>:<line>
```

Do not edit scoped files while the review runs.

Check the reply. No FILES READ, a list that misses changed files, or a run
implausibly fast for the diff size is not a review: restart it (same effort)
and tell the user.

## 3. Check findings

Check each finding against the code yourself. Keep every finding in your
report. If you think one is wrong, add a line that says why, with evidence.

Do not change code, even when the fix is obvious. For fixes, the user can ask
for `sol-loop`.

## 4. Comment on the PR

Use the PR the user named, or `gh pr view --json number,url,headRefOid` for the
current branch. If there is no PR, skip this step.

Post one GitHub review with one inline comment per finding:

```bash
gh api repos/<owner>/<repo>/pulls/<n>/reviews --method POST --input <file>
```

```json
{
  "commit_id": "<headRefOid>",
  "event": "COMMENT",
  "body": "<verdict and finding count>",
  "comments": [
    { "path": "src/ui/Input.tsx", "line": 42, "side": "RIGHT", "body": "<finding>" }
  ]
}
```

- Always use `COMMENT`. Never approve or request changes.
- Follow `leaving-pr-comments` for every comment and the review body. Name Sol's
  model as the author.
- Do not post findings you showed to be wrong, or issues that already have a
  comment on the PR. List them in your report.
- Findings in uncommitted or unpushed code cannot be anchored. Report them, and
  say they were not posted.
- If a finding is on a line outside the PR diff, put it in the review body with
  its file and line.
- If there are no findings to post, post nothing.

Verify that GitHub returned the review URL.

## 5. Report

Use this format, with the most severe finding first:

```markdown
### **#1 Empty input crashes the page**

An empty field on first load reaches `parse()`, which throws.

File: src/ui/Input.tsx:42
```

End with the verdict, the model, the base you reviewed, and the review URL or
why nothing was posted. If there are no findings, say that in one line.
