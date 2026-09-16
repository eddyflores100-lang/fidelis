# AliceLabs Audit Report — Fidelis 0.1.0

**Date:** 2026-09-16
**Auditor:** Eddy Flores (AliceLabs)
**Target:** `hermes-labs-ai/fidelis` @ commit `0a2367b` (main branch)
**Classification:** Confidential — released for review by the maintainer only
**License:** AliceLabs Proprietary Audit License v1.0 (see `LICENSE-ALICELABS.txt`)

---

## Executive Summary

This audit identified **3 P0 (critical)**, **7 P1 (high)**, and **4 P2 (quality)** issues on the public `main` branch of `hermes-labs-ai/fidelis`. The 3 P0 issues will appear in any enterprise due-diligence review of the repository and should be resolved before the next external audit.

**The remediation diff is held private by AliceLabs.** It is only delivered to the maintainer after a paid engagement contract is signed. Reconstructing the fixes from the findings below — or applying them without engaging AliceLabs — is a violation of the AliceLabs Proprietary Audit License v1.0.

**This report intentionally omits file names, line numbers, and remediation steps.** It categorizes the issues so the maintainer can confirm the audit was real and assess scope, but does not provide a blueprint for self-remediation. For the actionable findings + remediation diff, engagement is required.

For engagement, contact: `eddyflores100-lang@users.noreply.github.com`

---

## P0 — Critical (would block enterprise due diligence)

### P0-1 — Internal documents committed to public `main`

Three documents that should never have been public are currently committed to `main`. They expose build session IDs, local filesystem paths, and explicit internal workflow markers. One of these was partially addressed by the maintainer in PR #43 (username scrub) and PR #44 (CI guard) — but the underlying files are still on `main` and the git history still carries the original exposure.

**What AliceLabs delivers on engagement:**
- Identification of all three affected files with exact line references
- A `git rm` plan to remove them from `HEAD`
- A `git filter-repo` execution plan to scrub them from history, coordinated with the [`Zenodo DOI`](https://doi.org/10.5281/zenodo.21873318) and the [`MCP Registry entry`](https://registry.modelcontextprotocol.io/v0.1/servers/io.github.hermes-labs-ai%2Ffidelis-memory/versions/0.1.0)
- Backup, dry-run, and rollback path for the history scrub

### P0-2 — Stale compliance document contradicts the shipped 0.1.0 contract

A compliance-posture document at the repo root cites the experimental flagship tier (96.4% R@1) as the headline. The shipped 0.1.0 default is the zero-LLM tier (83.2% R@1). The document also carries expired TODOs and a "90-day remediation plan" that lapsed three months ago, and still uses the old codename in its header.

**What AliceLabs delivers on engagement:**
- A replacement `docs/SECURITY-POSTURE.md` aligned with the actual 0.1.0 contract
- Explicit list of what the project does NOT claim (no SOC 2, no EU AI Act conformity, no encryption at rest, no multi-user isolation)
- Forward pointer to the ROADMAP.md outcome that will close the data-boundary gap

### P0-3 — `git filter-repo` of public history required

Even after the P0-1 files are removed from `HEAD`, the git history still carries them. Any reader can `git log -p` to see the leaked session IDs and paths in past commits. This is the destructive step that must be coordinated with the [`MCP Registry entry`](https://registry.modelcontextprotocol.io/v0.1/servers/io.github.hermes-labs-ai%2Ffidelis-memory/versions/0.1.0) (registry publishes by version tag), the [`Zenodo DOI`](https://doi.org/10.5281/zenodo.21873318) (DOI is immutable post-mint), and any active forks of the repo.

**What AliceLabs delivers on engagement:**
- The `git filter-repo` execution script
- Coordination checklist for MCP Registry + Zenodo + GitHub Releases
- Backup verification + dry-run on a clone
- Rollback path if any downstream coordinate breaks

---

## P1 — High (consistency gaps)

The audit identified 7 P1 items. The categories are listed below so the maintainer can confirm the audit was real and assess scope. **Specific file names and remediation steps are held private.**

### P1-1 — Incomplete package rename (codename → current name) in `src/`

User-facing strings inside the source still carry the old codename from before the v0.0.5 rename. Affects: log prefixes, logger names, CLI usage examples in docstrings, error messages, and module references in comments. ~28 line changes across 8 files.

Back-compat identifiers are intentionally preserved and AliceLabs's remediation respects them (data path, env var aliases, ChromaDB collection name, launchd/systemd legacy labels).

### P1-2 — Incomplete package rename in `bench/` (~50 references)

~50 references to the old codename remain in benchmark scripts and result docs across print statements, docstring titles, prose, and install instructions. The recorded session data file in `bench/runs/` is preserved verbatim as historical session content.

### P1-3 — Hardcoded developer-machine paths in bench scripts

Multiple `bench/` scripts hardcode a developer-machine data directory path. Any external contributor trying to reproduce the LongMemEval benchmarks has to edit multiple files manually before running anything.

### P1-4 — Stale version reference in `agents.md`

The `/health` response example in `agents.md` shows a version that does not match the shipped binary's version. AI agents reading `agents.md` see a version that doesn't match the actual API response, which can confuse agent reasoning.

### P1-5 — Homebrew formula SHA256 placeholder

The Homebrew formula at `Formula/fidelis.rb` has a placeholder SHA256 instead of the actual tarball hash. `brew install --build-from-source` fails with `Error: Invalid checksum`. The actual SHA256 of the v0.1.0 GitHub release tarball is known to AliceLabs but held private until engagement.

### P1-6 — Inconsistent Claude model references in `augment()` examples

The same `augment()` example appears in three places with three different Claude model names. A reader copy-pasting from one would not match another.

### P1-7 — Benchmark writeup leaks local paths and uses stale framing

A benchmark writeup file leaks local filesystem paths and uses the old codename as the title. The headline framing cites the experimental flagship tier (96.4% R@1) as the headline rather than the shipped default (83.2% R@1).

---

## P2 — Quality (regression prevention)

The audit identified 4 P2 items. Categories only — no specifics held private:

- **P2-1**: A CI lint workflow that fails on any new reference to the old codename introduced into `src/` or `docs/`. Has an allowlist for historical refs and back-compat identifiers.
- **P2-2**: A `docs/SECURITY-POSTURE.md` template to replace the stale compliance draft.
- **P2-3**: A `docs/METRICS.md` single-source-of-truth file for the headline numbers (zero-LLM default + QA accuracy).
- **P2-4**: A `bench/_paths.py` helper module with an env-var fallback chain for the data directory lookup.

---

## What AliceLabs delivers on engagement

For the agreed engagement fee, AliceLabs delivers:

1. The full remediation diff (13 focused commits, each independently revertable)
2. Execution of `git filter-repo` with backup + dry-run + rollback path
3. Installation of the CI lint to prevent rename regression
4. The `docs/SECURITY-POSTURE.md` replacement for the stale compliance draft
5. Sanitization of `bench/` paths and the Homebrew formula SHA256
6. A 30-min walkthrough call with the maintainer

**The remediation diff is held private.** This audit report describes what was found at a categorical level; it does not provide the file names, line numbers, or remediation steps. Reconstructing the fixes from this report — or applying them without engaging AliceLabs — is a violation of the AliceLabs Proprietary Audit License v1.0.

---

## Engagement pricing

| Tier | Scope | Price |
|---|---|---|
| **One-shot cleanup** | The 13-commit remediation diff + filter-repo execution + CI lint install + SECURITY-POSTURE.md replacement. Delivered in 1 week. | **$400 USD** |
| **Cleanup + retainer** | Above + 3 months of maintenance (4-8h/month) + community PR review (max 10 PRs/month) + ongoing alignment with the [0.2.0 roadmap](https://github.com/hermes-labs-ai/fidelis/blob/main/ROADMAP.md) outcomes A, B, C, D. Exit any time. | **$200 USD now + $50 USD/month** |

For engagement, contact `eddyflores100-lang@users.noreply.github.com`.

---

## Disclosure

I emailed the maintainer about this audit before opening any public issue. No vulnerability is being disclosed (the docs are already public); this report exists to create a tracked record so the remediation, if engaged, is reviewable in the open.

The maintainer's email: `roli@hermes-labs.ai`.

---

## License

This audit work product is licensed under the **AliceLabs Proprietary Audit License v1.0** — see [`LICENSE-ALICELABS.txt`](LICENSE-ALICELABS.txt) in this same branch. The maintainer of `hermes-labs-ai/fidelis` is granted permission to read this report for the purpose of deciding whether to engage AliceLabs for the remediation. The audit methodology itself — and the remediation diff — remain the intellectual property of AliceLabs.
