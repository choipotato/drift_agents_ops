---
name: repo-doc-audit
description: Audit an entire repository's documentation estate for semantic drift, contradictions, stale current-state claims, missing operational coverage, broken historical framing, and gaps in drift bindings. Use for repository-wide health checks, documentation cleanup campaigns, handover validation, or before treating the repo as a trustworthy source of truth. Unlike repo-doc-review, this audits the whole repository rather than a diff or changed-file scope.
---

# Repository documentation audit

Audit the repository as a documentation system, not as a patch.

This skill answers:

> Can a maintainer or agent trust this repository's documentation as a coherent, current source of truth?

This is a repository-wide audit. Do not reduce scope to the current diff unless
the user explicitly asks for a partial audit.

Do not evaluate whether technical conclusions are sufficiently proven. Use a
separate evidence-review skill for claim strength, causality, and proof quality.

## 1. Establish repository scope

Run from the repository root.

Record:

- repository root
- current branch and HEAD
- default branch when available
- total tracked Markdown files
- whether `drift.lock` exists
- Drift version

Inventory tracked documentation with Git rather than `find` so generated,
ignored, and untracked debris does not silently redefine scope.

Typical commands:

```bash
git rev-parse --show-toplevel
git branch --show-current
git rev-parse HEAD
git ls-files '*.md' '*.mdx'
drift --version
```

If the repository uses other normative prose formats, include them only when the
repo itself treats them as documentation.

Do not start from changed files. The entire tracked documentation estate is in
scope.

## 2. Run deterministic Drift coverage first

Run the full repository check:

```bash
drift check --format json
```

If JSON is unavailable or unsuitable, fall back to:

```bash
drift check
```

Record:

- total docs checked
- fresh docs
- stale docs
- broken links
- skipped bindings
- partial or unavailable verification
- missing files or symbols
- baseline/fingerprint failures

Also inspect:

```bash
drift status --format json
```

when available, to understand binding coverage.

Important: a clean `drift check` proves only that configured fingerprints,
targets, and checked Markdown links are fresh. It does **not** prove that prose
is semantically correct, complete, mutually consistent, or appropriately
framed.

Never report "documentation is correct" merely because Drift returns zero.

## 3. Classify the documentation estate

Classify every tracked documentation file into one of these roles, using path,
title, frontmatter, wording, and repository conventions:

- **CURRENT / NORMATIVE** — intended to describe what is true now or what must
  be done now
- **RUNBOOK / PROCEDURE** — operational instructions that must match current
  behavior and dependencies
- **ACTIVE PLAN / TODO** — future or incomplete work that must not masquerade as
  current state
- **REFERENCE** — supporting material whose currentness depends on explicit
  framing
- **HISTORICAL / ARCHIVE / INCIDENT** — past observations that should be
  preserved, not rewritten into current truth
- **GENERATED / DERIVED** — output whose authority belongs elsewhere
- **UNKNOWN** — purpose is ambiguous and itself may be a finding

Do not treat `archive/`, `reference/`, or dated filenames mechanically. A
reference file can still be normative; a root-level file can still be
historical. Read enough to determine intent.

Produce counts for each class.

## 4. Build the authority map

Identify where the repository expects current truth to come from.

At minimum map these classes when present:

- executable configuration, inventory, manifests, role defaults, and policy
- runtime/current-state records
- tests, validation gates, and conformance definitions
- primary README and onboarding docs
- operational runbooks
- active TODO/status indexes
- reference and historical material

Use this default current-state precedence unless the repository explicitly
defines another:

1. executable configuration, inventory, manifests, role defaults, and code that
   directly define behavior
2. current runtime/status records from recent verification
3. tests and validation/conformance gates
4. current README, runbook, and current-state prose
5. active plans and TODOs
6. historical/reference records

This precedence resolves documentation consistency only. It does not prove root
cause or evidence quality.

Record exceptions when a repo explicitly names a canonical SSOT.

## 5. Extract high-value repository facts

Unlike `repo-doc-review`, do not build facts only from a diff.

Survey the repository for facts maintainers and agents rely on, including:

- canonical hostnames and aliases
- service names, ports, endpoints, paths, mount points
- storage source/destination relationships
- defaults and policy values
- dependencies and sequencing
- boot, shutdown, migration, restore, and preflight order
- ownership and role assignment
- supported and unsupported platforms or versions
- status words: active, paused, retired, deprecated, blocked, complete
- required validation gates
- canonical commands and entry points
- renamed, retired, or compatibility-only identifiers

Prefer facts that appear in more than one document, because duplication is where
semantic drift accumulates.

Do not attempt to enumerate every implementation detail. Focus on operational or
maintenance contracts.

## 6. Perform repository-wide semantic checks

Check all CURRENT, NORMATIVE, RUNBOOK, and ACTIVE PLAN documents against the
authority map and high-value facts.

Report these failure classes.

### STALE

A current/normative document states an old value, old dependency, retired name,
obsolete procedure, or superseded status.

### CONTRADICTION

Two current/normative sources disagree about the same fact.

Do not report an explicit historical-vs-current difference as a contradiction.

### MISSING COVERAGE

A current operational contract exists in executable/configuration sources but no
maintainer-facing current document explains it where one is reasonably needed.

Examples:

- a required preflight gate exists only in code
- a new mount dependency is absent from the runbook
- an operator-visible default changed with no current documentation

Do not demand prose for private implementation details.

### DUPLICATED AUTHORITY

Multiple documents independently claim to be the authoritative current source
for the same fact and can drift separately.

Recommend one owner plus references where practical.

### ACTIVE/HISTORICAL MIXUP

A past incident, archived plan, or dated measurement is linked or worded as if
it describes current state.

### TODO/STATUS DRIFT

An item marked open is already implemented, or an item marked complete still has
an unresolved current requirement.

Do not close or reopen TODOs based only on naming. Confirm against current repo
facts.

### ORPHAN CURRENT DOC

A document presents current operational truth but has no clear authority,
inbound context, ownership, or relationship to the repo's current-state map.

### BINDING COVERAGE GAP

A high-value current document describes code/config behavior but has no useful
Drift binding where a stable file or symbol target exists.

Do not require bindings for historical prose, policy rationale, or facts that
cannot be represented by a stable target.

## 7. Search for known-old values and competing names

For high-value facts, search both canonical and legacy forms across the entire
tracked documentation set.

Typical examples:

- old and new hostnames
- old and new service names
- retired paths
- aliases versus canonical names
- "paused" versus "active"
- "planned" versus "complete"
- old dependency relationships

Use repository-aware search such as `git grep` or `rg`.

A legacy term in historical material is usually valid. The finding is when its
scope or framing allows a reader or agent to mistake it for current truth.

## 8. Audit navigation and source-of-truth discoverability

A repository can be internally correct and still unsafe for agents if the entry
points lead to obsolete or ambiguous material.

Check:

- root README and START-HERE documents point to the actual current sources
- current-state indexes do not route readers into superseded material without
  clear framing
- active TODO indexes match the actual active files
- historical/archive directories are clearly marked as non-current
- duplicated top-level current-state documents do not compete silently
- canonical docs link outward rather than forcing readers to discover authority
  by filename guessing

Report discoverability problems only when they can cause a reasonable maintainer
or agent to select the wrong source.

## 9. Distinguish deterministic and semantic results

Keep these separate in the report.

### Deterministic Drift result

Report the exact outcome of `drift check` and binding coverage.

### Semantic audit result

Report repository-wide stale claims, contradictions, missing coverage,
authority duplication, framing problems, TODO drift, and binding gaps.

A clean deterministic result plus semantic findings is a normal and useful
outcome.

## 10. Report coverage before findings

Start with an audit summary:

```text
AUDIT COVERAGE
Repository: /path/to/repo
HEAD: <sha>
Tracked docs: 1324
Current/normative: 42
Runbooks: 17
Active plans/TODO: 68
Reference: 95
Historical/archive: 1102

Drift:
  checked: 1324
  stale: 0
  broken: 0
  verification: full

Semantic coverage:
  current/normative reviewed: 42/42
  runbooks reviewed: 17/17
  active TODO/status reviewed: 68/68
  historical/reference framing sampled/reviewed: <state exactly what was done>
```

Do not claim complete coverage for a class you did not actually inspect.

For large historical estates, full content review may be wasteful. It is
acceptable to inspect historical material primarily for framing, links from
current docs, and legacy-term collisions, but say so explicitly.

## 11. Report findings by severity and class

Use:

```text
BLOCKER — CONTRADICTION
current/storage.md:42
skai-storage-rebuild-runbook.md:88
Current sources disagree on the /users source for skaistor-03.
Authority: ansible/roles/skai_storage/... defines skaistor-01:/users.
Action: reconcile the current docs.

HIGH — STALE
README.md:73
Uses retired hostname fullmoon-003 in a current topology section.
Canonical current name: skaistor-03.

MEDIUM — MISSING COVERAGE
ansible/roles/skai_backup/defaults/main.yml
A preflight dependency is operationally required but absent from the backup
runbook.

LOW — BINDING COVERAGE GAP
current/runtime.md
Describes a stable service/config relationship but has no Drift binding to the
defining target.

HISTORICAL — LEAVE AS IS
reference/2026-10-03/measurements.md
Legacy name is correctly scoped to a dated observation.
```

Severity guidance:

- **BLOCKER** — following the docs can cause an incorrect or unsafe operation, or
  two canonical current sources directly conflict
- **HIGH** — likely to mislead maintenance or automation
- **MEDIUM** — important incompleteness or discoverability problem
- **LOW** — hygiene, ownership, or binding improvement without current wrong
  instructions

Do not inflate severity to make the report look substantial.

## 12. End with an audit verdict

End with one of:

- `Repository documentation is coherent at the audited scope.`
- `Repository documentation has non-blocking drift or coverage gaps.`
- `Repository documentation has material current-state contradictions.`
- `Audit incomplete: <specific unverified scope>.`

Then list the smallest remediation order:

1. current-state contradictions
2. stale operational instructions
3. missing current coverage
4. TODO/status drift
5. authority/discoverability cleanup
6. binding coverage improvements

Do not edit files unless explicitly asked.
