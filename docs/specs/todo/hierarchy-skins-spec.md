---
id: hierarchy-skins
status: draft
level: L2
repos: [Kiriketsuki/kTemplate]
issue: 12
parent: null
brief: docs/brief.md
updated: 2026-11-08
---

# Feature: Hierarchy skins

## Overview

**User Story**: As the owner of the personal repos, I want one canonical work hierarchy with swappable display names. Then tools and agents store stable level ids, and a repo changes its names by editing one file.

**Problem**: kTemplate hard-codes saga, epic, feature and task in its labels, issue templates and branch handler. The words have no agreed order, saga means two things, no level names a release, and no level holds an uncommitted idea. A rename today touches every workflow and template by hand.

**Out of Scope**:
- Aurrigo repos. They move later (brief D16).
- A themed skin such as the manga skin. The user dropped it (brief D14).
- The tutki skin loader (tutki T5.6) and the skiss import of `hierarchy.json`. Each product reads the file in its own spec.
- The edits to `merge-next`, the global git-workflow rule and the `gh-check-before-create.sh` hook. This spec files them as follow-up issues (R15).
- A GitHub App. The handler keeps using the workflow token.

---

## Success Condition

> This feature is complete when a scratch repo made from kTemplate handles all five issue kinds right under the legacy skin. A switch to the industry skin then renames the labels and templates with no handler change.

---

## Open Questions

| # | Question | Raised By | Blocks | Resolved |
|:--|:---------|:----------|:-------|:---------|
| 1 | kTemplate uses calendar versions (`YY.Major.Minor.Patch`) with bumps tied to epic, feature and task merges. The hierarchy uses semver, and a milestone names a release (brief D7). Does kTemplate move to semver with no bump on merge and a tag-push release, as tutki did in its PR #4? | brief, question 1 | T8 | [ ] |
| 2 | Does the GitHub API accept a cross-repo sub-issue parent? | brief, question 2 | none | [x] Not needed. Products, initiatives and sagas live on a GitHub Project (brief D3) |
| 3 | Do existing personal repos migrate at once, or when next touched? | brief, question 4 | T7 | [x] When next touched, with `hierarchy.mjs init` (this spec, R14) |

---

## Scope

### Must-Have

- **R1** (D1, D2, D15): THE repo SHALL hold a `hierarchy.json` file, version 1, that defines the nine canonical level ids `product`, `initiative`, `saga`, `repo`, `milestone`, `epic`, `feature`, `task` and `checklist`, and the two kinds `bug` and `hotfix`. A JSON schema in `schemas/hierarchy.schema.json` SHALL describe it.
- **R2** (D13): kTemplate SHALL ship two skins, `skins/legacy.json` and `skins/industry.json`. A new repo SHALL start with the legacy skin. The industry skin SHALL name `feature` as Story and keep the other work names standard.
- **R3** (D1): WHEN a user runs `node scripts/hierarchy.mjs check`, THE script SHALL exit 0 for a valid file. IF the file breaks a rule, THEN the script SHALL exit 1 and print the reason. The rules: every id exists, no display name repeats, each prefix equals its id, and the parent rules have no cycle.
- **R4** (D3, D5, D6, D10): THE skin SHALL mark `epic`, `feature`, `task`, `bug` and `hotfix` as issue levels. `product`, `initiative` and `saga` SHALL map to a GitHub Project. `milestone` SHALL map to a GitHub Milestone. `checklist` SHALL map to a task-list line in an issue body. `repo` SHALL map to the repo itself.
- **R5**: WHEN a user runs `node scripts/hierarchy.mjs labels`, THE script SHALL create or update one type label per issue level. Each label takes its colour and description from the skin. A second run SHALL change nothing. `--dry-run` SHALL print the plan and write nothing.
- **R6**: WHEN a user runs `node scripts/hierarchy.mjs templates`, THE script SHALL render `.github/ISSUE_TEMPLATE/*.yml` from the skin names. WHEN a user runs it with `--check`, THE script SHALL exit 1 if a checked-in template differs from the rendered output.
- **R7** (D9, D11): WHEN an issue gets a type label, THE handler SHALL apply the branch rule of its level. A feature SHALL always get a branch and a draft PR. An epic SHALL get a branch only when its `Integration branch` field says `yes`. A task SHALL get a branch only when its `Parallel worktree` field says `yes`. A bug or a hotfix SHALL always get a branch from main. Every branch prefix SHALL be the canonical id, whatever the skin.
- **R8** (D9): WHEN the handler picks a base branch, THE handler SHALL try a valid `Branch off:` value first. Next it SHALL try the nearest ancestor issue with a branch, then main. WHEN a task gets no branch, THE handler SHALL comment the parent feature branch on the issue.
- **R9** (D1): WHEN the handler maps a label to a level, THE handler SHALL read the map from `hierarchy.json`. IF the file is missing, THEN the handler SHALL use the built-in legacy map, so a repo that has not migrated keeps working.
- **R10** (D8): WHEN a feature or task issue has no milestone and its parent epic has one, THE handler SHALL set the parent's milestone on the child. WHEN the children of one epic sit in two or more milestones, THE handler SHALL comment a warning on the epic.
- **R12** (D12): THE skin SHALL list the allowed commit and PR types. The legacy and industry skins SHALL list the Conventional Commits types without `saga` and `epic`.
- **R13**: `.github/CI-CD-Guide.md` SHALL describe the canonical ids, both skins, the branch rules, the base order and the three script commands.
- **R14**: WHEN a user runs `node scripts/hierarchy.mjs init --skin legacy` in a repo made from an older kTemplate, THE script SHALL add `hierarchy.json`, then run `labels` and `templates`, and print what it changed.

### Should-Have

- **R11** (D6, D7): WHEN a semver tag `vX.Y.Z` matches the title of an open GitHub Milestone, THE release workflow SHALL name the release `vX.Y.Z <milestone title>`. Then it SHALL close that milestone. This depends on Open Question 1.
- **R15**: THE work SHALL file follow-up issues for `merge-next` in agenKic-hub and for the vault git-workflow rule and naming hook. Each then reads the canonical ids and the skin commit types.

### Nice-to-Have

- A `hierarchy.mjs explain <level>` command that prints the level's names, branch rule and parent rule.

---

## Alternatives Considered

| Alternative | Why not |
|:------------|:--------|
| Branch prefixes from the skin names, for example `story/` | The global hook, `merge-next` and tutki then need every skin. Canonical prefixes keep one rule set. |
| Hard-code the new names in the handler and templates | Every later rename repeats the work. The brief chose a skin file (D1). |
| A branch at every level | Rebase chains grow, and one writer per branch gets harder. The brief chose feature always and the rest optional (D9). |
| Sagas and initiatives as issues in a hub repo | Nobody has confirmed API support for cross-repo parents. A GitHub Project spans repos today (D3). |
| A milestone integration branch made by the handler | A GitHub Milestone has no issue event. An optional script can make it later. |
| A manga skin | The user dropped it on 2026-11-08 (D14). |

---

## Technical Plan

**Affected Components**:

| Path | Change |
|:-----|:-------|
| `hierarchy.json` | New. The repo skin file, legacy by default |
| `skins/legacy.json`, `skins/industry.json` | New. The two shipped skins |
| `schemas/hierarchy.schema.json` | New. JSON schema for version 1 |
| `scripts/hierarchy.mjs` | New. `check`, `labels`, `templates` and `init`, Node standard library only, `gh` for labels |
| `test/hierarchy.test.mjs`, `test/fixtures/hierarchy/` | New. `node --test` cases for every check rule and both skins |
| `.github/ISSUE_TEMPLATE/*.yml` | Rendered from the skin. Epic gains `Integration branch`. Task gains `Parallel worktree` |
| `.github/workflows/issue-branch-handler.yml` | Reads the skin, applies the branch rules, the base order and the milestone copy |
| `.github/workflows/release.yml` | R11, after Open Question 1 |
| `.github/CI-CD-Guide.md` | R13 |

**Data Model Changes**:

- `hierarchy.json`, version 1:
  ```json
  {
    "v": 1,
    "skin": "legacy",
    "levels": {
      "product":    { "name": "Product",        "store": "project" },
      "initiative": { "name": "Initiative",     "store": "project", "stages": ["idea", "shaping", "bet", "active", "done"] },
      "saga":       { "name": "Saga",           "store": "project" },
      "repo":       { "name": "Repository",     "store": "repo" },
      "milestone":  { "name": "Milestone",      "store": "milestone" },
      "epic":       { "name": "Epic",           "store": "issue", "label": "epic",    "branch": "optional", "parents": ["milestone"] },
      "feature":    { "name": "Feature",        "store": "issue", "label": "feature", "branch": "always",   "parents": ["epic", "milestone"] },
      "task":       { "name": "Task",           "store": "issue", "label": "task",    "branch": "worktree", "parents": ["feature"] },
      "checklist":  { "name": "Checklist item", "store": "tasklist" }
    },
    "kinds": {
      "bug":    { "name": "Bug",    "label": "bug",    "branch": "always", "base": "main" },
      "hotfix": { "name": "Hotfix", "label": "hotfix", "branch": "always", "base": "main" }
    },
    "labels": { "epic": { "color": "3e4b9e" }, "feature": { "color": "a2eeef" }, "task": { "color": "0075ca" }, "bug": { "color": "d73a4a" }, "hotfix": { "color": "b60205" } },
    "commitTypes": ["feat", "fix", "refactor", "docs", "test", "chore", "perf", "ci", "hotfix", "release"]
  }
  ```
- The industry skin differs only in names and labels: `feature` gets name `Story` and label `story`.
- `branch` takes `always`, `optional`, `worktree` or `none`. The prefix is always the level id.

**CLI Contracts**:

- `node scripts/hierarchy.mjs check [FILE]`: exit 0 when valid, exit 1 with one reason per line.
- `node scripts/hierarchy.mjs labels [--dry-run] [-R OWNER/REPO]`: create or update the type labels.
- `node scripts/hierarchy.mjs templates [--check]`: render or verify the issue templates.
- `node scripts/hierarchy.mjs init --skin legacy|industry`: add the file, then run `labels` and `templates`.

**Dependencies**: Node 20 or later and `gh` on the machine that runs the script. The handler uses `actions/github-script`, as it does today.

**Risks**:

| Risk | Likelihood | Mitigation |
|:-----|:-----------|:-----------|
| A handler change breaks branch creation in repos that copied kTemplate | Medium | R9 keeps the built-in legacy map. T4 tests the handler in a scratch repo before merge |
| A relabel triggers the handler and makes duplicate branches | Medium | The handler already skips an issue that has a branch comment. T2 tests a second `labels` run on a repo with open issues |
| The industry label `story` misses the old `feature` trigger | Low | The handler reads labels from the skin (R9), so `story` maps to `feature` |
| Calendar version bumps fire on a task branch that no longer exists | Medium | Open Question 1 decides the version scheme before T8 |

**Rollout**:

1. T1 to T3 add the file, the skins and the script. Nothing in CI changes yet.
2. T4 and T5 change the handler. A scratch repo tests every level before merge.
3. T6 updates the guide. T7 adds `init`.
4. Open Question 1 decides the version scheme. T8 follows it.
5. T9 files the follow-up issues.
6. Personal repos migrate with `init` when next touched. tutki and skiss read the file in their own specs.

---

## Acceptance Scenarios

```gherkin
Feature: Hierarchy skins
  As the owner of the personal repos
  I want one canonical hierarchy with swappable names
  So that tools store stable ids and a rename edits one file

  Background:
    Given a scratch repo made from this kTemplate branch
    And "hierarchy.json" holds the legacy skin

  Rule: The skin file is valid

    @R1 @R3
    Scenario: A valid skin passes the check
      When the user runs "node scripts/hierarchy.mjs check"
      Then the command exits 0

    @R3
    Scenario: A skin with a missing level fails the check
      Given "hierarchy.json" has no "feature" level
      When the user runs "node scripts/hierarchy.mjs check"
      Then the command exits 1
      And the output names the missing level "feature"

  Rule: Each level follows its branch rule

    @R7
    Scenario Outline: An issue gets the branch rule of its level
      Given an issue labeled "<label>" with "<field>" set to "<value>"
      When the handler runs
      Then the result is "<result>"

      Examples:
        | label   | field              | value | result                              |
        | feature | none               | none  | branch feature/<n>-* and a draft PR |
        | epic    | Integration branch | no    | no branch                           |
        | epic    | Integration branch | yes   | branch epic/<n>-* from main         |
        | task    | Parallel worktree  | no    | a comment naming the parent branch  |
        | task    | Parallel worktree  | yes   | branch task/<n>-* from the parent   |
        | bug     | none               | none  | branch bug/<n>-* from main          |
        | hotfix  | none               | none  | branch hotfix/<n>-* from main       |

    @R8
    Scenario: A feature under an epic with no branch bases on main
      Given an epic issue with no branch
      And a feature issue that is a sub-issue of that epic
      When the handler runs for the feature
      Then the feature branch bases on "main"

    @R9
    Scenario: A repo with no skin file keeps working
      Given the repo has no "hierarchy.json"
      When an issue gets the label "task"
      Then the handler uses the built-in legacy map

  Rule: A skin switch renames without a handler change

    @R2 @R5 @R6
    Scenario: The industry skin renames labels and templates
      Given "hierarchy.json" holds the industry skin
      When the user runs "node scripts/hierarchy.mjs labels" and "node scripts/hierarchy.mjs templates"
      Then the repo has a "story" label
      And the feature template title says "Story"
      And "node scripts/hierarchy.mjs templates --check" exits 0

    @R5
    Scenario: A second labels run changes nothing
      Given the labels already match the skin
      When the user runs "node scripts/hierarchy.mjs labels --dry-run"
      Then the output says "no changes"

  Rule: An epic stays in one milestone

    @R10
    Scenario: A child takes the milestone of its epic
      Given an epic in milestone "v0.3.0"
      When a feature sub-issue with no milestone gets the label "feature"
      Then the feature is in milestone "v0.3.0"

    @R10
    Scenario: An epic with children in two milestones gets a warning
      Given an epic with one child in "v0.3.0" and one child in "v0.4.0"
      When the handler runs for the second child
      Then the epic has a comment that names both milestones
```

---

## Task Breakdown

| ID | Task | Priority | Dependencies | Status | Reqs | Boundary | Check |
|:---|:-----|:---------|:-------------|:-------|:-----|:---------|:------|
| T1 | Add the schema, both skins, `hierarchy.json` and `hierarchy.mjs check` | High | None | pending | R1, R2, R3, R4, R12 | `hierarchy.json`, `skins/`, `schemas/`, `scripts/hierarchy.mjs`, `test/` | `node --test test/` exits 0, and `node scripts/hierarchy.mjs check` exits 0 |
| T2 | Add `hierarchy.mjs labels` with `--dry-run` | High | T1 | pending | R5 | `scripts/hierarchy.mjs`, `test/` | In the scratch repo, a second `labels --dry-run` prints `no changes` |
| T3 | Add `hierarchy.mjs templates` and render the issue templates | High | T1 | pending | R4, R6 | `scripts/hierarchy.mjs`, `.github/ISSUE_TEMPLATE/` | `node scripts/hierarchy.mjs templates --check` exits 0 |
| T4 | Make the handler read the skin, apply the branch rules and the base order | High | T1, T3 | pending | R7, R8, R9 | `.github/workflows/issue-branch-handler.yml` | In the scratch repo, the seven Scenario Outline rows give the stated results |
| T5 | Add the milestone copy and the split-milestone warning | Med | T4 | pending | R10 | `.github/workflows/issue-branch-handler.yml` | In the scratch repo, both milestone scenarios give the stated results |
| T6 | Update the CI-CD guide | Med | T4 | pending | R13 | `.github/CI-CD-Guide.md` | The guide names all nine ids, both skins and the three commands, and kilint reports no errors |
| T7 | Add `hierarchy.mjs init` for older repos | Med | T2, T3 | pending | R14 | `scripts/hierarchy.mjs`, `test/` | `init --skin legacy` in a copy of an older repo exits 0 and `check` then exits 0 |
| T8 | Apply the version decision from Open Question 1, then name releases after milestones | Low | T4 | pending | R11 | `VERSION`, `.github/workflows/` | A test tag `v0.0.1` creates a release named after its milestone, or the guide records why R11 waits |
| T9 | File the follow-up issues for `merge-next`, the git-workflow rule and the naming hook | Low | T4 | pending | R15 | none | `gh issue list` in agenKic-hub shows the `merge-next` issue, and the vault holds the rule and hook notes |

---

## Exit Criteria

- [ ] All Must-Have scenarios pass: `node --test test/` exits 0, and the scratch-repo log shows each handler scenario.
- [ ] No regressions: a repo with no `hierarchy.json` still gets `task/<n>-*` branches.
- [ ] CLI contracts match the implementation: `node scripts/hierarchy.mjs --help` lists `check`, `labels`, `templates` and `init`.
- [ ] Templates are current: `node scripts/hierarchy.mjs templates --check` exits 0 on main.
- [ ] Docs follow STE: `python3 /home/kiriketsuki/dev/obKidian/000-System/Scripts/kilint/bin/kilint .github/CI-CD-Guide.md docs/` reports no errors.
- [ ] The Success Condition holds: the scratch-repo log shows all five issue kinds under the legacy skin. It also shows the industry switch with no handler diff.

---

## Implementation Notes

Workers append what they learned here, one dated line per note, with the task ID.

---

## References

- Decision record: `docs/brief.md`
- Vault brief: `100-Projects/120-Planning/129-Personal/Work Hierarchy Skins - Proposal.md`
- Research: vault `000-System/Research/2026-11-08-hierarchy-research.md` and `000-System/Research/2026-11-08-agentic-sdlc-research-MOC.md`
- Format: tutki contract C2, `Kiriketsuki/tutki` `docs/specs/todo/tutki-spec.md`
- Prior work: `docs/canonical-template-design.md` (untracked in the local clone)
