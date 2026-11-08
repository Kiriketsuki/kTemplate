---
record: brief
id: hierarchy-skins
mode: interrogate
exit: feature-spec
status: approved
approved_by: user
date: 2026-11-08
repos: [Kiriketsuki/kTemplate, Kiriketsuki/skiss, Kiriketsuki/tutki]
level: L2
---

# Work Hierarchy Skins

## Problem

The personal repos use saga, epic, feature and task. Nobody agrees on the order of saga and epic. The word saga covers two things today: cross-repo work and a major roadmap milestone. No level names a release. No level holds an idea that is not yet committed work. The names live in kTemplate, the branch handler, `merge-next`, the global git rules and tutki D21, so a change touches many places.

## Goals and Non-Goals

Goals:
- One canonical set of level ids that every tool stores.
- A skin file that maps each id to display names, labels and branch prefixes.
- A clear place for cross-repo work, one-repo releases and uncommitted ideas.
- Fewer branch levels, with the feature as the PR unit.

Non-Goals:
- A change to Aurrigo repos now. Aurrigo moves later, after months of personal use.
- A manga or other themed skin. The user dropped it on 2026-11-08.
- A change to Conventional Commits types other than the removal of the level types.
- A change to tutki now. tutki reads the skin file in a later spec change.

## Decisions

| ID | Decision | Reason | Source |
|---|---|---|---|
| D1 | Separate structure from names. Tools store canonical level ids. A skin maps ids to names. | Agents, Aurrigo and public tutki users need stable ids, and a rename then costs one file. | user, round 1 and 3 |
| D2 | Canonical ids: `product`, `initiative`, `saga`, `repo`, `milestone`, `epic`, `feature`, `task`, `checklist`. | These cover the product, the idea funnel, cross-repo work, the repo, the release and three work levels. | user, rounds 2 to 7 |
| D3 | Two containers: `product` (a GitHub Project) and `repo` (the repo itself). Neither is an issue. | They are places, not work. GitHub confirms cross-repo sub-issue parents only in the UI picker. | user, round 5, research |
| D4 | `initiative` holds a big idea. Stages: idea, shaping, bet, active, done. Agents shape. Only a human bets. A bet creates one or more sagas. | An idea needs shaping and a go decision before branches exist. Agents do the grunt work, the human decides. | user, rounds 8 and skiss |
| D5 | `saga` is the cross-repo roadmap milestone. It carries a date and spans repos. A milestone in each repo links to one saga. | This keeps the meaning the user already gives saga. | user, round 6 |
| D6 | `milestone` is the one-repo release. It is bound by scope, has a target date, and its title names a semver release, for example "v0.3.0 Codex parity". | Solo agentic work ships in bursts, so a fixed time-box slips or idles. | user, round 3, research |
| D7 | Versions stay semver. A milestone names a release. The milestone count is not encoded in the version. | Semver tells users about compatibility, and a milestone count does not. | user, round 3 |
| D8 | An epic stays inside one milestone. If it slips, the whole epic moves. A split epic becomes two epics. | An epic that spans two releases has no clean done point. | user, round 4 |
| D9 | Branches: `feature` always gets a branch and is the PR unit. `epic`, `milestone` and `saga` may get an optional integration branch. `task` gets a branch only for a parallel worktree, and merges into its feature. | One writer per branch, one PR per unit, and fewer rebase chains. | user, round 5, research |
| D10 | `checklist` is a line in a task body. It has no issue, no branch and no id. | Spec tools keep steps as checkboxes, and three working levels is the common default. | user, round 2, research |
| D11 | Bug and hotfix are kinds of work, not levels. They branch from main. | They describe the work, not its size. | user, round 3 |
| D12 | Personal repos drop the saga and epic commit and PR types. Conventional Commits stay. | Commit types describe the change, not the level. | user, round 3 |
| D13 | Skins: `legacy` (Product, Initiative, Saga, Repository, Milestone, Epic, Feature, Task, Checklist item) and `industry` (Milestone, Epic, Story, Task for the work levels). | Legacy keeps today's words for the least change. Industry suits public users. | user, rounds 5 and 6 |
| D14 | The user dropped the manga skin. | User choice, 2026-11-08. | user |
| D15 | Each repo has a `hierarchy.json` skin file. | One file swaps the skin, and tools read it instead of hard-coded names. | user, round 3 |
| D16 | Scope: personal repos now, Aurrigo later. | Aurrigo has colleagues, Jira and shared conventions. | user, round 1 |
| D17 | Home: this brief and the research live in obKidian. The spec and build live in kTemplate. tutki and skiss read `hierarchy.json` later. | The research spans several repos, and kTemplate holds the labels and the branch handler. | user, round 4 |

## Alternatives Weighed

- Manga skin (Franchise, Series, Season, Arc, Chapter, Page, Panel): kept through several rounds, then dropped by the user.
- All seven themed levels as issues: rejected because containers are not work, and a panel is below issue size.
- Three work levels only (no feature level): rejected for epic, feature and task.
- A full rename with names hard-coded in every tool: rejected for a skin over canonical ids.
- Saga as a fourth work level above epic: rejected for saga as the cross-repo milestone.
- Calendar versions or a version tied to the milestone count: rejected for semver.
- A fixed two-week or six-week cycle: rejected for scope-bound milestones with a target date.
- Branches at every level: rejected for feature always and the others optional.

## Constraints

- STE writing for all prose.
- The `gh-check-before-create.sh` hook enforces branch and commit names. Its type list must change with D12.
- The kTemplate branch handler reads labels. A label change must keep old repos working until they migrate.
- kTemplate itself uses calendar versions with bumps tied to epic, feature and task. That conflicts with D7.

## Assumptions and Open Questions

| # | Question | Blocking |
|---|---|---|
| 1 | Does kTemplate move to semver, as tutki did in its PR #4? | yes, for the kTemplate version workflows |
| 2 | Does the GitHub API accept a cross-repo sub-issue parent? | no, products use a GitHub Project |
| 3 | When do `merge-next` and the global git-workflow rule switch to the canonical ids? | no, follow-up |
| 4 | Do existing personal repos migrate at once, or when next touched? | no, the spec proposes when touched |

## Loop Hints

Not a loop exit. This brief feeds feature-spec, which writes the hierarchy skin spec in kTemplate. Research: the vault notes `000-System/Research/2026-11-08-hierarchy-research.md` and `000-System/Research/2026-11-08-agentic-sdlc-research-MOC.md`. Vault brief: `100-Projects/120-Planning/129-Personal/Work Hierarchy Skins - Proposal.md`.
