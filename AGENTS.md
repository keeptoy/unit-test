# AGENTS.md

> **Canonical instructions:** This English file is the single authoritative
> instruction source loaded by compatible agents. Do not create translated
> `AGENTS.*.md` variants that could drift or be mistaken for a second authority.

## Repository mission

This repository is an open-source vertical prototype for adapting a local Codex
Skill Hook runtime to Codex Cloud system-managed Hooks. The first and currently
only supported integration is `OthmanAdi/planning-with-files` (PWF).

Do not describe the project as a universal Skill converter yet. The long-term
direction is to prove the PWF Cloud path, extract Skill-independent deployment
and Host-adaptation capabilities, and validate those abstractions with a second
read-only Plugin before promising general compatibility.

## Start every maintenance session here

Read these files in order before changing behavior or architecture:

1. `PROJECT_UNDERSTANDING.md` — durable project model and Cloud evidence.
2. `.planning/.active_plan` — active plan selector.
3. The active plan's `task_plan.md`, `progress.md`, and `findings.md`.
4. `work_plan.md` — staged Phase 1–9 release roadmap.
5. `README.md` and `黑盒验证.md` — public claims and Cloud acceptance runbook.
6. `git status --short --branch` — existing worktree state.

Use the planning files for multi-step work. Put external research in
`findings.md`, chronological implementation/test results in `progress.md`, and
update `task_plan.md` only when the execution contract or phase state changes.

## Current product truth

- `v0.2.2` is the published, Cloud-validated rollback baseline.
- `v0.3.0` is an unpublished Managed Runtime Modernization iteration.
- Phase 1 is complete locally; the remaining acceptance step is publication of
  the separate `v0.3.0-alpha.1` ZIP and bootstrap assets followed by a fresh
  Codex Cloud install/doctor/inventory/compatibility smoke.
- Phase 1 installs the verified upstream runtime as inactive inventory. Managed
  Hook commands still execute `hook_adapter.py` with v0.2.2-compatible behavior.
- Do not switch execution to the owned upstream runtime before the Phase 1 Cloud
  smoke is reviewed.
- Current production behavior enables only read-only `SessionStart` and
  `UserPromptSubmit` handlers.

## Non-negotiable architecture and safety rules

1. Managed Hook commands use absolute paths beneath the configured
   `hooks.managed_dir`.
2. Never execute mutable, unverified runtime scripts directly from a user's
   Skill directory as the long-term design.
3. Install and doctor fail closed on missing, changed, unknown, symlinked, or
   inventory-drifted trusted-runtime content.
4. Preserve non-owned requirements and Hook definitions wherever the ownership
   contract promises preservation.
5. Runtime advisory failures remain non-fatal to the Codex loop; containment or
   attestation failures must prevent unsafe plan text from being injected.
6. Hook stdout must remain valid, bounded Codex JSON. Send detailed diagnostics
   to the separate non-injecting diagnostic surface.
7. Treat Hook stdin `session_id` and validated `transcript_path` as primary Host
   inputs. Environment variables and session-store enumeration are compatibility
   inputs, not sole identity sources.
8. Treat Codex transcript JSONL record shapes as changeable Host data. Parse
   defensively and fail safely on unknown records.
9. Preserve upstream files byte-for-byte when possible. Put Codex Cloud Host
   translation in the owned adapter or a documented compatibility overlay.
10. Add tests before enabling each lifecycle event. Enable events in separate,
    reviewable Cloud rollouts.
11. Keep rollout canaries until fresh-session verification passes.
12. Add hard Stop gating last, behind explicit opt-in, caps, stall detection, a
    kill switch, and an independently tested rollback path.
13. Do not create a self-referential Release archive: the bootstrap that pins
    the ZIP checksum remains a separate asset outside that ZIP.

## Local development checks

Run the checks relevant to the files changed. Before committing a release-facing
change, run the complete available set:

```bash
npm test
python3 -m py_compile hooks/hook_adapter.py
node --check install.js
python3 tools/import_upstream_runtime.py check
python3 tools/build_release.py check --archive dist/pwf-codex-cloud-hooks-v0.3.0-alpha.1.zip
bash -n init-cloud-sandbox-v0.3.0.bash
git diff --check
```

If a required interpreter is unavailable, report the exact environmental
limitation; do not claim that check passed. Tests must use temporary Codex homes
and requirements files and must not mutate live `/opt/codex` or
`/etc/codex/requirements.toml` state.

## Contribution opportunities

High-value contributions include:

- running and recording the documented fresh Codex Cloud pre-release smoke;
- improving Host-contract fixtures without treating observed transcript JSONL
  as a permanent public schema;
- retiring compatibility overlays when a pinned upstream PWF release provides
  equivalent behavior;
- extracting a thin, Skill-independent Hook Host protocol boundary after the
  PWF runtime switch is proven;
- proposing a second, low-risk, read-only Plugin adapter to test reuse of the
  installer, manifest, supervisor, diagnostics, and Cloud testkit; and
- documenting Codex Cloud image or Hook-contract changes with dates, versions,
  sanitized evidence, and explicit inference labels.

Do not add automatic discovery and execution of arbitrary Skill Hooks. A future
general framework must require an explicit source pin, runtime allowlist,
license/provenance record, event allowlist, dependency contract, Cloud fixtures,
and administrator review.

## Public documentation and discoverability

Keep public wording accurate while making the project discoverable. Use natural,
specific terms such as `Codex Cloud Hooks`, `Managed Hooks`, `Codex Skill cloud
adapter`, `Codex Plugin runtime`, `requirements.toml`, `managed_dir`, `remote
agent hooks`, `session resume`, and `planning-with-files` where relevant.

Do not keyword-stuff, imply OpenAI endorsement, claim official Enterprise
support, or say that every Skill can be adapted automatically. Distinguish:

- local Skill installation from Cloud runtime installation;
- Skill discovery from Hook registration;
- Plugin Hook trust from Managed Hook policy;
- current Cloud observations from stable platform contracts; and
- the PWF vertical implementation from the possible future general framework.

When behavior, package inventory, test count, release hashes, or rollout status
changes, update `README.md`, `PROJECT_UNDERSTANDING.md`, the active planning
files, and relevant runbooks together so search results do not surface stale
claims.
