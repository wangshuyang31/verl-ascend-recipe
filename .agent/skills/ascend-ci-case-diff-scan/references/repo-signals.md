# Repo Signals

Use these target-repo facts when running `ascend-ci-case-diff-scan` against an external `verl` checkout.

## Primary inputs

- `.github/workflows/*.yml`
- `tests/README.md`
- `docs/ascend_tutorial/contribution_guide/ascend_ci_guide_zh.rst`

## Workflow families

- CPU workflows include `cpu_unit_tests.yml`.
- GPU workflows are test workflows that are not Ascend workflows.
- NPU workflows usually end with `_ascend.yml`, or clearly run on Ascend runners / Ascend images.

Examples typically present in the target `verl` repository:

- `reward_model_sglang.yml` and `reward_model_sglang_ascend.yml`
- `model.yml` and `model_ascend.yml`
- `e2e_sft_llm.yml` and `e2e_sft_llm_ascend.yml`
- `e2e_ascend.yml`
- `nightly_ascend.yml`
- `npu_unit_tests.yml`

## Ignore rules

- Ignore `examples/**` completely for this skill.
- Ignore commented workflow commands.
- Ignore non-test workflows such as `doc.yml`, `sanity.yml`, `check-pr-title.yml`, `pre-commit.yml`, `precommit-autofix.yml`, `secrets_scan.yml`, `scorecard.yml`, and similar policy/docs workflows.

## Extractable command forms

Recognize only workflow commands that visibly execute tests:

- `pytest ... tests/...`
- `bash tests/.../*.sh`
- `torchrun ... tests/...`
- `torchrun ... -m pytest tests/...`

This skill is static. It does not expand matrices, infer hidden includes, or execute shell scripts.

## Matching expectations

- Exact target matches are the strongest signal.
- Same target with materially different env/argument prefixes should usually become `manual_review_needed`.
