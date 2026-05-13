# Ascend Skills 使用说明

本文档介绍当前仓库内 `ascend-ci-case-diff-scan` skill 的用途、调用方式与输出结果。它的定位是：由当前仓库承载扫描能力，对外部 `verl` 开源仓进行静态分析，便于在昇腾 NPU 场景下快速审视 CI 用例覆盖差异。

## 1. Skill 目标

`ascend-ci-case-diff-scan` 用于做一类静态检查：

1. 以 `.github/workflows/*.yml` 中 **CPU/GPU workflow 实际执行的用例** 为基线，检查哪些用例没有在 NPU/Ascend workflow 中执行。

该 skill 是一个静态扫描器，只分析 workflow 中 `run:` 可见的测试执行命令，不会真正执行测试。

## 2. 检查边界

当前实现的核心分析输入来自被分析的目标 `verl` 仓库：

- `.github/workflows/*.yml`

其中：

- `examples/**` 下的 NPU shell 脚本 **不参与检查**
- workflow 中注释掉的命令 **不参与检查**
- 非测试类 workflow（如文档、sanity、check-pr）会被忽略
- GitHub Actions matrix **不会展开**
- 只有 `run:` 中可以直接看到的命令会被纳入判断

## 3. 调用示例

### 3.1 直接运行扫描脚本

在当前仓库根目录下执行，并把 `--repo-root` 指向外部 `verl` 仓库：

```shell
python .agent/skills/ascend-ci-case-diff-scan/scripts/scan_ascend_ci_case_diff.py \
  --repo-root {PATH}/verl \
  --output-dir ./report/ascend-ci-case-diff-scan
```

如果你希望参数语义更明确，也可以使用等价别名：

```shell
python .agent/skills/ascend-ci-case-diff-scan/scripts/scan_ascend_ci_case_diff.py \
  --target-repo-root {PATH}/verl \
  --output-dir {PATH}/verl-ascend-recipe/report/ascend-ci-case-diff-scan
```

### 3.2 通过 skill 描述触发

如果你是在支持 skill 的 Codex 环境中使用，可以这样描述：

```text
Use $ascend-ci-case-diff-scan to compare CPU/GPU workflow cases against Ascend workflows and report missing or unmatched NPU coverage.
```

也可以直接用中文提出目标，例如：

```text
请使用 ascend-ci-case-diff-scan，分析外部 `verl` 仓库里 CPU/GPU workflow 已执行、但 NPU workflow 没有执行的用例，以及需要人工复核的差异项。
```

## 4. 输出结果

扫描完成后会在当前仓库的输出目录下生成一份产物：

- `report.md`：面向人工阅读的报告

主要分类包括：

- `aligned`：CPU/GPU workflow 中的 case，在 NPU workflow 中也执行了
- `missing_in_npu_workflows`：CPU/GPU workflow 跑了，但 NPU workflow 没跑
- `manual_review_needed`：目标相同但参数矩阵不同，或无法可靠自动判定
- `npu_only`：只在 NPU workflow 中出现的 case

对于同一 `target` 下既有明显对齐项、也有剩余参数差异项的情况，当前实现会优先把已明确对齐的执行命令归入 `aligned`，只把无法继续自动配对的剩余命令归入 `manual_review_needed`，以减少误报。

`report.md` 中的每个条目会尽量按固定英文前缀展开，便于快速扫读：

- `Original file location`
- `Difference`
- `Signature summary`
- `CPU/GPU case locations`
- `NPU case locations`

其中 workflow 定位信息会进一步拆成更明确的字段，便于快速定位：

- `Workflow file`
- `Line`
- `Workflow context`
- `Command`

同时会附带 workflow 文件行号，便于直接跳转到对应配置位置。

对于 `manual_review_needed` 部分，报告会尽量把 CPU/GPU 与 NPU 的引用按相邻上下文成对展示，方便人工逐组比较，而不是先集中展示一侧、再集中展示另一侧。

如果你希望把报告写到其他目录，可以在运行时修改 `--output-dir`；脚本不会把输出目录写死。与此同时，脚本会校验 `--repo-root` / `--target-repo-root` 是否真的是一个包含 `.github/workflows` 的目标仓库，避免误把当前 skill 仓库本身当作被分析对象。

## 5. 结果解读建议

建议按以下顺序阅读结果：

1. 先看 `missing_in_npu_workflows`
2. 再看 `manual_review_needed`
3. 最后再看 `aligned` 与 `npu_only`

其中：

- `missing_in_npu_workflows` 更适合直接驱动补齐工作流
- `manual_review_needed` 往往意味着 GPU/NPU 复用了同一目标，但参数矩阵并不完全一致，需要人工确认是否真的等价

## 6. 当前实现特性

当前版本是静态扫描器，具备这些特征：

- 不执行任何 workflow
- 不展开 GitHub Actions matrix
- 以 workflow `run:` 中可见命令作为唯一执行依据
- 对复杂参数差异保持保守，优先归入 `manual_review_needed`
