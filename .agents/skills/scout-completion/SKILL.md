---
name: scout-completion
description: Load when a scout reports completion, presents a visual artifact for iteration, or is being considered for promotion to implementation.
user-invocable: false
metadata:
  internal: true
---

# Scout outcome and promotion

A completed scout must leave a self-contained report before its scratch worktree can be discarded; read and relay its findings, record the report as the Done artifact, and re-evaluate the queue.
A report may recommend implementation but does not authorize it.
Before treating the investigation or any visual review as complete, load `captain-hold-lifecycle`; teardown enforces that shared completion gate.
When a scout's deliverable is a visual artifact the captain will iterate on, keep it alive and follow the crew-hosted Lavish board contract in `docs/configuration.md` rather than arming or polling the board from firstmate.
When implementation is separately authorized, promote the existing scout through `bin/fm-promote.sh` rather than creating a duplicate task.
The promoted worker must inventory scratch state, return to a clean copy of the task's base (its recorded base branch, else the default branch), carry over only intended fix changes, create the ship branch, and follow the project's selected delivery path while leaving scratch commits and debug edits behind and turning a reproduced bug into the regression test.

## Trusting a scout's seam

A report's `## Seam` section (the patch, its base commit, how it was exercised, the measured effects, and what was not verified) is established evidence for every later step: the issue, the ship or promotion instructions, and the no-mistakes intent cite it and never turn it into acceptance criteria to reproduce.
Acceptance covers only what the seam does not: the shipped change matches the patch or explains each deviation, the patch still fits current code (existing tests and cheap deterministic checks such as query plans), and whatever the scout listed as unverified or did not exercise.
Re-verify a seam measurement only when the shipped change deviates from the patch in a way that could affect it, the measured code changed on the base since the seam's commit, or the scout flagged the result as uncertain, and then re-verify only the affected part.
Every done-when line in an issue or worker instruction either fits that acceptance or names the re-verification condition that requires it.
