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

For engagement, contact: `eddyflores100-lang@users.noreply.github.com`

---

## P0 — Critical (would block enterprise due diligence)

### P0-1 — Internal documents committed to public `main`

Three documents that should never have been public are currently committed to `main`. They expose build session IDs, local filesystem paths, and explicit internal workflow markers.

**Affected files (verified at commit `0a2367b`):**

- `docs/LAUNCH_DEFENSE.md` — line 5 contains the literal marker: *"DO NOT publish this document."*
- `docs/internal/HANDOFF.md` — contains a Claude build session UUID, an explicit "Do NOT push to GitHub or publish anywhere without explicit Roli approval" marker, and a local user filesystem path (`/Users/rbr_lpci/...`)
- `docs/internal/PUBLISH-PLAN-20260425.md` — internal release-gating workflow with maintainer approval steps
- `docs/internal/FLAGSHIP-PAPER-DRAFT.md` — academic paper draft with unfilled `{{F2_*}}` placeholders
- `receipts/2026-09-15-S6-mcp-smoke.txt` — line 4 contains a local user filesystem path

**Why it matters:** Any reviewer reading these files sees internal context (session IDs, naming conventions, maintainer's local directory layout) that compromises the appearance of a clean, externally-facing OSS product.

**Remediation scope:** AliceLabs has the full diff prepared. Includes gitignore rules to prevent future leaks of the same kind.

---

### P0-2 — `COMPLIANCE-DRAFT.md` contradicts the shipped 0.1.0 contract

The file `COMPLIANCE-DRAFT.md` (at the repo root):

- Cites the experimental flagship tier (96.4% R@1) as the headline. The shipped 0.1.0 default is the zero-LLM tier (83.2% R@1).
- Carries a "Remediation plan (next 90 days)" that lapsed 2026-07 (three months ago).
- Lists open TODOs that were never resolved: "No bias testing", "No behavioral robustness harness", "No incident-response runbook".
- Still uses the old codename `cogito-ergo` in its header.

**Why it matters:** A compliance reviewer reading this thinks the project is pre-release with unresolved security gaps. The document actively damages credibility.

**Remediation scope:** AliceLabs has prepared a `docs/SECURITY-POSTURE.md` replacement aligned with the actual 0.1.0 contract — but the file content is held private until engagement.

---

### P0-3 — `git filter-repo` of public history required

Even after the files above are removed from `HEAD`, the git history still carries them. Any reader can `git log -p` to see the leaked session IDs and paths in past commits.

**Why it matters:** The P0-1 files are not actually remediated until history is scrubbed. However, `git filter-repo` is destructive and must be coordinated with:

- the [`MCP Registry entry`](https://registry.modelcontextprotocol.io/v0.1/servers/io.github.hermes-labs-ai%2Ffidelis-memory/versions/0.1.0) (registry publishes by version tag)
- the [`Zenodo DOI 10.5281/zenodo.21873318`](https://doi.org/10.5281/zenodo.21873318) (DOI is immutable post-mint)
- any active forks of the repo
- published GitHub Releases that reference the affected commits

**Remediation scope:** AliceLabs has a `filter-repo` execution plan with backup, dry-run, and rollback path. Destructive operations are not executed without a signed engagement.

---

## P1 — High (consistency gaps)

The audit also identified 7 P1 items. Their full detail is held private; the list below is provided so the maintainer can confirm the audit was real and not generic.

### P1-1 — Rename `cogito-ergo` → `fidelis` incomplete in `src/`

User-facing strings inside the source still carry the old codename (`cogito-ergo`):

- Print prefixes `[cogito]` in `src/fidelis/server.py`, `snapshot.py`, `calibrate.py`, `seed.py`
- Logger name `cogito.server` in `src/fidelis/server.py`
- CLI usage examples in docstrings (`cogito seed`, `cogito calibrate`, `cogito snapshot`, etc.)
- Error messages referencing `cogito seed`, `cogito server not reachable`
- Module references in comments (`cogito.recall_hybrid`, `cogito.config`)

**Count:** 28 line changes across 8 files in `src/fidelis/`.

**Back-compat preserved:** `~/.cogito/` data path, `COGITO_*` env var aliases, `cogito_memory` ChromaDB collection name, `_LEGACY_LABELS` tuple in `init_cmd.py` (launchd/systemd labels from pre-rename installs).

### P1-2 — `cogito-ergo` codename still in `bench/` (~50 references)

~50 references to the old codename `cogito-ergo` remain across:

- `bench/longmemeval_combined_pipeline_v31.py` through `v35.py` (print statements like `"*** cogito-ergo BEATS Mastra ..."`)
- `bench/LAUNCH-FRAMING.md`, `bench/RESULTS-SUMMARY.md`, `bench/BENCHMARK_INTEGRITY_AUDIT.md`, `bench/VALIDATION_PACK.md`, `bench/ROADMAP-95.md`, `bench/ROADMAP-95-v2.md`
- `bench/claude_code_session_eval.py`, `bench/longmemeval_tuned.py`, `bench/longmemeval_retrieval.py`, `bench/eval.py`
- `bench/runs/claude_code_user_eval.json` (preserved as historical session data — not branding)

### P1-3 — Hardcoded developer-machine paths in 9 bench scripts

9 `bench/` scripts hardcode:

```python
DATA_DIR = Path.home() / "Documents/projects/LongMemEval/data"
```

Affected files: `bench/run_chunked.py`, `bench/analyze_pipeline.py`, `bench/scaffold_arena.py`, `bench/scaffold_arena_round2.py`, `bench/scaffold_transfer_validation.py`, `bench/qwen_native_arena.py`, `bench/phase-2/{retriever_contribution,learned_router,build_hardset}.py`, `bench/phase-4/temporal_scaffold.py`.

Any external contributor trying to reproduce the LongMemEval benchmarks has to edit 9 files manually before running anything.

### P1-4 — `agents.md` shows stale version `0.0.9` in `/health` example

`agents.md:99` shows the `/health` response example as `"version": "0.0.9"`. The shipped binary returns `"0.1.0"`. AI agents reading `agents.md` see a version that doesn't match the actual API response, which can confuse agent reasoning about which version of the API is alive on the other end of the HTTP call.

### P1-5 — `Formula/fidelis.rb` has placeholder SHA256

`Formula/fidelis.rb:22` contains:

```ruby
sha256 "TBD-fill-when-v0.1.0-tag-tarball-is-published"
```

Anyone running `brew install --build-from-source ./Formula/fidelis.rb` gets `Error: Invalid checksum`.

The actual v0.1.0 GitHub release tarball SHA256 is known to AliceLabs but held private until engagement.

### P1-6 — Inconsistent Claude model references in `augment()` examples

The same `augment()` example appears in three places with three different Claude model names:

| File | Model referenced |
|---|---|
| `src/fidelis/augment.py:17` | `claude-opus-4-7` |
| `docs/full-reference.md:35` | `claude-opus-4-5` |
| `README.md:428` | `claude-haiku-4-5` |

A reader copy-pasting from one would not match another, and the only hint that the model name is illustrative is a comment.

### P1-7 — `WRITEUP-LONGMEMEVAL-20260423.md` leaks local paths and uses stale framing

- Header says `cogito-ergo` instead of `Fidelis`
- Author handle `roli-lpci` exposed
- Local filesystem paths `~/Documents/projects/cogito-ergo/` and `~/Documents/projects/research-corpus/agent-infra/raw/cogito-longmemeval-20260423.json`
- Headline framing reads "The number we report: 96.4%" — but that is the experimental flagship tier, not the shipped 83.2% default

---

## P2 — Quality (regression prevention)

The audit also identified 4 P2 items held private:

- **P2-1**: A CI lint workflow (`.github/workflows/naming-audit.yml`) that fails on any new `cogito-ergo` / `cogito.<module>` / `[cogito]` reference introduced into `src/` or `docs/`. Has an allowlist for historical refs and back-compat identifiers.
- **P2-2**: A `docs/SECURITY-POSTURE.md` template to replace the removed `COMPLIANCE-DRAFT.md`.
- **P2-3**: A `docs/METRICS.md` single-source-of-truth file for the headline numbers (83.2% R@1 zero-LLM default + 73.0% QA accuracy).
- **P2-4**: A `bench/_paths.py` helper module with the `FIDELIS_BENCH_DATA_DIR` env-var fallback chain.

---

## What AliceLabs delivers on engagement

For the agreed engagement fee, AliceLabs delivers:

1. The full remediation diff (13 focused commits, each independently revertable)
2. Execution of `git filter-repo` with backup + dry-run + rollback path
3. Installation of the CI lint to prevent rename regression
4. The `docs/SECURITY-POSTURE.md` replacement for `COMPLIANCE-DRAFT.md`
5. Sanitization of `bench/` paths and the Homebrew formula SHA256
6. A 30-min walkthrough call with the maintainer

**The remediation diff is held private.** This audit report describes what was found; it does not describe how the issues are fixed. Reconstructing the fixes from this report — or applying them without engaging AliceLabs — is a violation of the AliceLabs Proprietary Audit License v1.0.

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
