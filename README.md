# SoftwareOS

SoftwareOS is Moveo's team framework for Spec-Driven Development on Claude Code. It gives PMs and devs one shared planning hierarchy — customer → ecosystem → product → epic → spec — where **the spec is the single source of truth** for what was built and why. Every level is a slash command that writes plain markdown into the project's `softwareos/` folder, so agents and humans read the same docs. Forked from agent-os v3 (we kept its standards system intact).

```
PM ──> /plan-customer ──> [/plan-ecosystem] ──> /plan-product ──> /plan-epic ──> /shape-spec
          one-time            optional           per product       recurring      recurring
          customer/           ecosystem/         products/<p>/     products/<p>/epics/<e>/   …/specs/<s>/

Onboard an existing codebase instead:  /join-project  (discovers customer → ecosystem → products → epics)
Onboard a new techlead:                /explain-os    (presents all of this, grounded in the project's docs)

Standalone:  /hotfix   /spec-changes   /spec-of   /project-status   /verify   /code-review   /refresh-indexes   /curate   /go-production
Skills:      testing   qa   audit-log   pr   process-meeting   explain-os   skill-creator
```

Every project runs in one of two **modes** — `setup` (lightweight) or `production` (tests, security review, and QA demanded and enforced). See [Project modes](#project-modes).

| Flag | Effect |
|---|---|
| `--profile <name>` | Install standards from a specific profile (default: `moveo`, per base `config.yml`) |
| `--update` | Refresh commands/skills/agents/hooks from the base repo; leaves your `softwareos/` docs and standards untouched |
| `--commands-only` | Only update commands; preserve existing standards |
| `--no-hooks` | Skip installing hooks (also respected by `--update`) |
| `--no-agents` | Skip installing agents (also respected by `--update`) |
| `--no-skills` | Skip installing skills (also respected by `--update`) |
| `--verbose` | Detailed output |
| `-h, --help` | Usage |

## The planning hierarchy

| Command | Cadence | Who | Output |
|---|---|---|---|
| `/plan-customer` | one-time per customer | PM | `softwareos/customer/` — business goals, team, tech stack, repo index |
| `/plan-ecosystem` | optional (multi-product customers) | PM | `softwareos/ecosystem/ecosystem.md` — products + shared services map |
| `/plan-product` | one-time per product | PM + Tech | `softwareos/products/<product>/` — mission, roadmap, tech-stack, architecture, dependencies |
| `/plan-epic` | recurring | PM / Tech / Both | `softwareos/products/<product>/epics/<epic>/` — epic.md + product-brief.md (PM) + tech-plan.md (Tech) |
| `/shape-spec` | recurring | PM / Tech / Both | `products/<product>/epics/<epic>/specs/YYYY-MM-DD-<slug>/` — spec.md (+ tasks.md if Tech/Both) |

Each level reads the levels above it, so a spec is grounded in its epic, product, and customer context automatically.

**Onboarding an existing codebase:** instead of running `/plan-*` by hand, `/join-project` discovers the whole hierarchy — customer → ecosystem → products → epics — from the code in one pass (two gates: confirm the taxonomy, choose the **mode**), writes the docs into the nested `softwareos/products/<product>/epics/<epic>/…` tree, generates an `index.yml` at each level, and emits a `join-report.md`. Run `/refresh-indexes` any time to rebuild those index files from the filesystem (drift recovery).

## Project modes

`softwareos/config.yml` carries `mode: setup | production`. One toggle per project — it decides how hard the gates bite.

| | `setup` | `production` |
|---|---|---|
| `/shape-spec` test question | offers TDD / after / **none** | `none` unavailable; Verification group always has unit tests + security review + QA |
| `/verify` | open tasks are status, not failures | FAILs on open Verification tasks, missing `qa-report.md` on a done spec, or a skipped test stage |
| qa skill | PASS WITH NOTES absorbs unverifiable ACs; runs checks-only with no ACs | those become FAIL; no ACs means stop and shape them first |
| pr skill preflight | verification pipeline only | also needs a passing `qa-report.md` and a recorded `> Reviewed:` |
| `/code-review` | `security-reviewer` on sensitive paths; HIGH is advice | always dispatched; unfixed HIGH/CRITICAL blocks the PR |
| hooks | commit/force-push guards, unchanged | unchanged — `context-banner` shows the mode; git itself is never gated |

**Who sets it:** the mode is always **asked, never inferred** — it's a decision about how the team wants to work, and no combination of files in a repo can answer it. `/join-project` asks as its second and final gate, showing what it found (CI, real tests, deploy artifacts, release tags) as *context for your answer* rather than as a proposal. `/plan-customer` asks when creating a config fresh, and never touches an existing `mode:`. The installer scaffolds `setup` so the key always exists. `/go-production` transitions an already-onboarded project: it audits every spec, makes you fix or **explicitly waive** each blocking gap, and records the result in `softwareos/production-readiness.md`. `/go-production --revert` goes back, also recorded.

**Greenfield/brownfield is a different axis.** `/join-project` is always brownfield, but a brownfield repo can be a live system (`production`) or a legacy codebase mid-rewrite (`setup`). Mode tracks operational risk, not code age.

**How the value resolves** — identical in every command, hook, and agent:

| `mode:` in `config.yml` | Runs as | Says |
|---|---|---|
| `setup` / `production` | that mode | nothing |
| Case variant (`Production`) | that mode | notes the canonical form is lowercase |
| Anything else (`prod`, `strict`, empty `mode:`) | `setup` | **warns** — names the bad value and says production gates are OFF |
| Key absent | `setup` | nothing |

Case variants resolve *toward* the stricter regime on purpose: someone who typed `Production` meant production, and under-enforcing a project that meant to be gated is the failure worth avoiding. An unrecognized value can't be guessed at, so it falls back to `setup` — but never quietly, because a one-character typo would otherwise remove every gate while the team believed they were on.

An absent key is the only silent fallback: that's the compatibility floor for projects onboarded before modes existed, not the normal path. `--update` never adds the key (`config.yml` is yours), so existing projects keep behaving exactly as they did until someone runs `/go-production`.

**Enforcement lives in the tools, not in git.** `/verify`, the qa skill, `/code-review`, and the pr preflight are where production mode bites, because they can actually run the suite, read the report, and check the diff. Hooks only surface the mode — no push or merge gate. Blocking git would obstruct the very things that produce verification evidence (CI runs on a pushed branch; review happens on a PR), and a merge clicked in the GitHub UI is unreachable from a hook anyway. If you want a gate that genuinely holds at merge, use GitHub branch protection with required status checks.

**Onboarding a person:** `/explain-os` presents the built-in deck — what SoftwareOS is, the hierarchy, which command to use when, and where your first hour goes — one slide at a time, in about ten minutes. Before it starts, the `os-explainer` agent reads the project's `softwareos/` tree, so the hierarchy slide shows *your* products and epics, the dev-loop slide shows *your* branch and its open tasks, and the closer names *your* next command. It also reports **gaps** — which commands haven't been run here yet, and whether the indexes have drifted. Pick a track (`techlead` · `new-project` · `pm` · `quick`), or skip the deck and just ask: `/explain-os "when do I use /hotfix vs /spec-changes?"`. Read-only; the deck lives in `.claude/skills/explain-os/slides/`.

## Day-to-day

**Dev loop:**

1. `/shape-spec` (in plan mode) — pick the epic, shape `spec.md` + `tasks.md`
2. Branch `feat/<epic-slug>/<spec-slug>`, work tasks T1… and tick them off
3. **testing** skill — TDD / verification tasks
4. `/verify` — build, lint, typecheck, tests
5. `/code-review` — standards-aware review of the branch diff
6. **qa** skill — verify acceptance criteria, writes `qa-report.md`
7. **pr** skill — runs preflight, opens the PR, writes the URL back to `tasks.md`

**PM loop:**

- **process-meeting** skill — turn a recording/transcript into a module-understanding write-up under `meetings/`, ending in an ordered "what to build next" that feeds `/plan-epic` and `/shape-spec`
- `/plan-epic` — open the next epic (product brief + tech plan)
- `/project-status` — read-only progress report: tasks, branches, devs, PRs, stale/drift flags
- `/spec-changes` — change a spec in place with an audit trail
- `/spec-of <area>` — reverse lookup: which spec owns this route, page, or file?
- **audit-log** skill — full timeline of a spec (changes, hotfixes, commits)

## Conventions

**Per-project tree** (everything under `softwareos/`):

```
softwareos/
├── config.yml                 # customer, customer_root (optional), default_branch, mode
├── production-readiness.md    # only in production mode — audit, waivers, transition history
├── customer/                  # business goals, team, tech stack, repo index
├── ecosystem/                 # ecosystem.md + products.index.yml (product registry)
├── meetings/                  # cross-product meeting write-ups (process-meeting skill)
├── products/<product-slug>/
│   ├── mission.md roadmap.md tech-stack.md architecture.md dependencies.md repos.md
│   ├── epics.index.yml
│   ├── meetings/              # YYYY-MM-DD-<slug>.md — upstream of epics, no index
│   └── epics/<epic-slug>/     # no date prefix — slug is used in branch names
│       ├── epic.md  product-brief.md  tech-plan.md
│       ├── specs.index.yml
│       └── specs/YYYY-MM-DD-<spec-slug>/
│           ├── spec.md        # single source of truth
│           └── tasks.md  standards.md  references.md  qa-report.md  visuals/
└── standards/                 # index.yml + <area>/*.md
```

Multi-repo customers keep full customer/ecosystem docs in one hub repo; other repos point at it via `customer_root` in `config.yml`.

**Branches:** `feat/<epic-slug>/<spec-slug>` for spec work, `hotfix/<epic-slug>/<bug-slug>` for bugs, `chore/<slug>` otherwise. Full rules in `standards/global/git-workflow.md`.

**Bugs & changes — the spec stays true:**
- Bugs never get a bug doc or a bugs folder. `/hotfix` triages the report, fixes the defect with a regression test, and records the correction as a Changelog row (Type=hotfix) on the **owning spec** — linking the external Jira/Monday ticket in the Ref column — and strengthens a standard when the root cause is a missing/weak rule. Your tracker holds the ticket; the spec Changelog is the in-repo record. Never a new spec for a bug.
- Change requests edit the spec in place via `/spec-changes` (Changelog row, Type=change). Months later, `spec.md` is still what's actually built.
- **Changing a spec invalidates the evidence that it was satisfied.** `/spec-changes` marks `qa-report.md` (a `> Stale:` line) and the `> Reviewed:` record stale when it edits acceptance criteria or the technical approach — nothing is deleted, but the gates stop counting it until the qa skill and `/code-review` are rerun. Otherwise a spec could pass every check, move underneath them, and still read green.

**Role-gated docs:** epic and spec sections are owned by PM, Tech, or Both. Unfilled sections stay visible with a pending marker (`_Pending — rerun /shape-spec as <role> to fill._`) so gaps are explicit, not silent.

**tasks.md states:** `[ ]` open · `[x]` done · `[~]` in progress · `[-]` cancelled. IDs `T<n>` / `T<n>.<m>` are append-only — never renumbered. `/project-status` parses this format, so don't freestyle it. The `> Key: value` metadata block (`Branch`, `PR`, and the `Reviewed` / `SecurityReviewed` records `/code-review` writes) must stay within the file's first 15 lines — every parser reads only that window.

## Standards & profiles

The agent-os standards system, unchanged:

- `/discover-standards` — extract conventions from a codebase into standards files
- `/inject-standards` — inject relevant standards into the current context (uses `standards/index.yml`)
- `/index-standards` — rebuild the index after adding/removing standards
- `scripts/sync-to-profile.sh` — push good project standards back to a base profile for reuse

Profiles live in `profiles/`. `moveo` is the house profile (inherits `default`) and is the install default. Grow it by discovering standards in client repos and syncing them back: `sync-to-profile.sh --profile moveo`.

## Updating

- **A project:** rerun the installer with `--update`. Commands, skills, agents, and hooks are refreshed; your `softwareos/` docs and standards are never touched.
- **The base repo:** `git pull` in `~/software-os`, then `--update` your projects.
- **Upstream agent-os:** we forked buildermethods/agent-os v3.0 and have diverged — don't merge upstream. Diff their `commands/` and `scripts/` manually, port what's useful, and bump `upstream.agent_os_version` in the base `config.yml`.

## Attribution

SoftwareOS is a fork of [Agent OS](https://github.com/buildermethods/agent-os) by Brian Casel (Builder Methods), with agents and commands adapted from [affaan-m/ECC](https://github.com/affaan-m/ECC). See [NOTICE.md](NOTICE.md).
