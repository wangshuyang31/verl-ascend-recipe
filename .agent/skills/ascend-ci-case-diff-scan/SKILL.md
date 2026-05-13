---
name: ascend-ci-case-diff-scan
description: Scan an external verl repository for Ascend/NPU CI coverage gaps by comparing CPU/GPU workflow-executed test cases against NPU/Ascend workflows. Use when Codex needs to audit CI parity, report missing Ascend coverage, or generate a Markdown report about workflow-level test execution differences. Ignore examples/ NPU shell scripts.
---

# Ascend CI Case Diff Scan

## Overview

Use this skill from the current repository to audit workflow-level Ascend CI coverage in a target `verl` repository.

The skill focuses on two concrete checks:

1. Compare CPU/GPU workflow-executed test cases against NPU/Ascend workflows and report which CPU/GPU cases are missing from NPU workflows.

This skill is intentionally narrow:

- Read workflow `run:` commands as the source of truth for executed cases.
- Ignore `examples/**`, even if it contains NPU shell scripts.
- Ignore commented workflow lines.
- Do not execute tests.

## Workflow

1. Read [references/repo-signals.md](./references/repo-signals.md) to refresh the repo-specific rules and boundaries.
2. Identify the target `verl` repository root you want to analyze, for example `{PATH}/verl`.
3. Run the scanner from the current repository:

```shell
python .agent/skills/ascend-ci-case-diff-scan/scripts/scan_ascend_ci_case_diff.py \
  --repo-root {PATH}/verl \
  --output-dir ./report/ascend-ci-case-diff-scan
```

You can also use the explicit alias if it reads better in the prompt or command line:

```shell
python .agent/skills/ascend-ci-case-diff-scan/scripts/scan_ascend_ci_case_diff.py \
  --target-repo-root {PATH}/verl \
  --output-dir ./report/ascend-ci-case-diff-scan
```

4. Summarize the highest-signal gaps first:
   - CPU/GPU cases missing in NPU workflows
   - anything routed to `manual_review_needed`
   - relevant NPU-only coverage worth noting

## Case Extraction Rules

Treat workflow `run:` blocks as the only execution source. Extract only:

- `pytest ... tests/...`
- `bash tests/.../*.sh`
- `torchrun ... tests/...`
- `torchrun ... -m pytest tests/...`

For each extracted case, preserve:

- `workflow_name`
- `job_name`
- `step_name`
- `command_type`
- `target`
- `raw_command`
- `signature`

Use `signature` to keep meaningful env/parameter prefixes such as `ROLLOUT_NAME=sglang` or `ENGINE=sglang`. If two workflows hit the same target with different signatures, prefer `manual_review_needed` over assuming parity.

## Classification Rules

Use these output categories:

- `aligned`
- `missing_in_npu_workflows`
- `manual_review_needed`
- `npu_only`

Interpret them conservatively:

- `aligned`: CPU/GPU workflow case also appears in an NPU workflow with the same target and compatible signature.
- `missing_in_npu_workflows`: CPU/GPU workflow case has no NPU workflow match.
- `manual_review_needed`: same target exists but signatures differ, or the command cannot be matched safely.
- `npu_only`: case exists only in NPU workflows.

## Reporting

The scanner writes output into the current repository's `--output-dir` and reads workflows from the external target repository passed by `--repo-root` / `--target-repo-root`.

The scanner writes:

- `report.md`

Keep the human summary short and useful. Lead with missing NPU workflow coverage, then mention aligned coverage, manual-review cases, and relevant NPU-only coverage.
