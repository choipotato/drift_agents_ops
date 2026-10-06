---
name: repo-doc-review
description: Review repository documentation for stale bindings, broken links, cross-document contradictions, missing current-state updates, and incorrect historical framing. Use after code/config changes, when drift check reports stale anchors, or when README/runbook/current/reference/TODO docs may disagree.
---

# Repository documentation review

Review documentation as a representation of the repository's current state.

This skill answers one question:

> Are current repository documents consistent with the code, configuration, and other current documents they describe?

Do not evaluate whether a technical claim is sufficiently proven. That belongs in
a separate evidence-review skill.

## 1. Start with deterministic drift checks

Run:

```bash
drift check
```

If the task is scoped to changed files, use `drift refs <target>` and
`drift check --changed <path>` where appropriate.

Treat these as hard signals:

- stale code/file/symbol anchors
- deleted or renamed targets
- broken markdown links
- unavailable fingerprints or baselines
- partial verification due to skipped origin bindings

Do not blindly relink stale anchors. Follow the existing `drift` skill's relink
gate and review the prose before using `--doc-is-still-accurate`.

## 2. Build the changed-facts set

From the diff and affected targets, extract only facts that can make current
documentation stale:

- hostnames, service names, ports, paths, mount points
- defaults, dependencies, ordering, boot/shutdown sequences
- ownership, source/destination relationships, role assignments
- status transitions: planned, active, paused, fixed, deprecated, removed
- validation gates and required preflight checks
- renamed or deleted files, commands, symbols, jobs, or environments

Read enough context to know what changed. Do not infer semantics from an
identifier rename alone.

## 3. Find current-state documents that mention those facts

Search both the new value and the old value.

Prioritize:

1. current/runtime/status docs
2. runbooks and operational procedures
3. README and onboarding docs
4. configuration/inventory documentation
5. active TODO/plan/status indexes
6. generated current inventories
7. historical references and incident records

Historical material is not stale merely because reality later changed.

## 4. Check four failure classes

### STALE

A current or normative document still states an old value or relationship.

### CONTRADICTION

Two current-state documents disagree about the same fact.

Do not report historical-vs-current differences as contradictions when the
historical scope is explicit.

### MISSING

An operationally relevant repository contract changed but no current document
reflects it.

Do not demand docs for every implementation detail. Report only facts a
maintainer or operator would reasonably rely on.

### HISTORICAL FRAMING

A historical record is presented or linked as if it were current, or current
docs cite an old observation without its date/scope.

Prefer fixing framing or links over rewriting historical evidence.

## 5. Use current-state precedence

When current sources disagree, prefer the source that most directly defines the
repository state:

1. executable configuration, inventory, manifests, role defaults, and code
2. current runtime/status records from recent verification
3. current tests and validation gates
4. current README/runbook/status prose
5. active plans and TODOs
6. historical references and archived outputs

This order is only for documentation consistency. It does not establish evidence
strength or root cause.

## 6. Report actionable findings only

Use:

```text
STALE
path/to/doc.md:42
Claim: skaistor-03 has no /users dependency
Repository fact: current storage config mounts skaistor-01:/users
Action: update the dependency and affected verification step

CONTRADICTION
current/runtime.md:18
README.md:73
These current-state docs disagree on whether the service is paused.

MISSING
roles/storage/defaults/main.yml
The client mount contract changed, but no current runbook records the new
preflight requirement.

HISTORICAL — LEAVE AS IS
reference/2026-10-03/measurements.md
This correctly records the observation as of 2026-10-03.
```

For deterministic drift findings, include the relevant `drift check` reason
and binding target.

End with exactly one verdict:

- `No documentation drift found.`
- `Documentation drift found; current-state docs need updates.`
- `Only historical differences found; no current-state update required.`

Do not edit files unless explicitly asked.
