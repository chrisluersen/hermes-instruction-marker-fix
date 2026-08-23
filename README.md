# Hermes Instruction-Marker Fix

A current-baseline patch for a Hermes Agent coding-context detection bug: an instruction-only workspace containing `AGENTS.md`, `CLAUDE.md`, or `.cursorrules` is incorrectly classified as a code project even when it contains only notes or prose.

## Behavior corrected

Hermes currently combines true code manifests and agent instruction files in one project-marker list. This patch separates them:

- manifests such as `pyproject.toml`, `package.json`, and `go.mod` remain sufficient code-project evidence;
- instruction files require a corroborating source-code file;
- conventional `src/`, `app/`, `lib/`, and test package layouts are detected without scanning arbitrary nested vault projects;
- the complete marker union remains available to workspace snapshots;
- git repositories containing source files continue to be recognized through the existing code-file detector.

This prevents instruction-rooted knowledge vaults from receiving a coding posture and irrelevant verify-on-stop expectations solely because they have an agent context file.

## Current upstream baseline

- Upstream: <https://github.com/NousResearch/hermes-agent>
- Baseline commit: `f293e7206b4ddd66042329442c6afebc19a8808d`
- Files changed:
  - `agent/coding_context.py`
  - `tests/agent/test_coding_context.py`

## Apply

```bash
git clone https://github.com/NousResearch/hermes-agent.git
cd hermes-agent
git checkout f293e7206b4ddd66042329442c6afebc19a8808d
git apply --check /path/to/hermes-instruction-marker-fix.patch
git apply /path/to/hermes-instruction-marker-fix.patch
```

Do not force-apply the patch to a different baseline. Rebase the small two-file change and rerun verification if upstream moved.

## Verify

Using the Hermes development environment:

```bash
python -m pytest \
  tests/agent/test_coding_context.py \
  tests/agent/test_verification_stop.py \
  tests/run_agent/test_file_mutation_verifier.py \
  -q
```

Candidate result: **76 passed, 1 skipped, 1 warning**. The warning is pytest's already-imported `anyio` assertion-rewrite warning from reusing the existing Hermes environment.

## Scope

- Does not disable coding context or verify-on-stop.
- Does not change how true manifests or source-bearing repositories are detected.
- Does not modify a live Hermes installation.
- Does not contain credentials, organization-specific data, runtime transcripts, or machine-specific paths.
- This repository is a reviewable patch kit, not an upstream fork.

## License

The patch modifies MIT-licensed Hermes Agent source. The upstream MIT license is included as `LICENSE`.
