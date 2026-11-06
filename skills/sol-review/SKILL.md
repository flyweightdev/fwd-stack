---
name: sol-review
description: Use when the user asks for a review with Sol, or says "sol review". One read-only review pass. For a review and fix loop, use sol-loop.
---

# sol-review

Get one independent review of the current change from GPT Sol, then report the
findings. This does not change code. If the user wants the findings fixed, they
will say so, or they can ask for `sol-loop`.

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
Read surrounding code and repo guidance (CLAUDE.md, AGENTS.md). Repo guidance
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
#1 <title>
<paragraph>
File: <path>:<line>
```

Do not edit scoped files while the review runs.

## 3. Report

Check each finding against the code yourself. Keep every finding in the list.
If you think one is wrong, add a line that says why, with evidence. Do not
drop it silently, and do not fix anything.

Use this format, with the most severe finding first:

```markdown
### **#1 Empty input crashes the page**

An empty field on first load reaches `parse()`, which throws.

File: src/ui/Input.tsx:42
```

End with the verdict, the model, and the base you reviewed. If there are no
findings, say that in one line. Keep the findings in your reply. Post to GitHub
only if the user asks, and then use `leaving-pr-comments`.
