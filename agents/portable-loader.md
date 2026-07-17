# Portable Loader Prompt

在不原生识别 SKILL.md 文件夹的 Agent 平台中，使用下面的提示词加载本 Skill。

~~~text
你可以访问一个名为 factor-ranking-sage 的本地 Skill，路径是：

<FACTOR_RANKING_SAGE_SKILL_ROOT>

当用户请求匹配该 Skill 的 SKILL.md 描述时：

1. 先读取 <FACTOR_RANKING_SAGE_SKILL_ROOT>/SKILL.md。
2. 严格按照 SKILL.md 中的工作流和边界说明执行。
3. 仅在需要时读取 <FACTOR_RANKING_SAGE_SKILL_ROOT>/references/ 下的引用文件。
4. 在读取相关说明后，从 Skill 根目录运行内置脚本。
5. 保持文档中定义的配置字段、算法边界、文件路径、输出约定、验证边界和数据来源边界。
6. 不要编造 Skill 未支持的 SHAP、Permutation Importance、drop-one、retrain coalition、HPO、回测或组合优化行为。
7. 将输出视为因子排名候选，不要解释为样本外提升证明、交易信号、收益承诺或生产交易验证。
~~~

## 用途

本仓库提供一个可移植的 Skill 入口，用于对本地因子和标签 CSV 运行确定性回归 mRMR 或固定模型 Marginal-SAGE 因子排名。

## 运行入口

~~~bash
python scripts/run_factor_selection.py --input <input-json>
~~~

最小示例：

~~~bash
python scripts/run_factor_selection.py --input examples/factor_selection_mrmr_smoke.json
~~~

## 真实运行流程

1. 按照 references/input_schema.md 准备因子、标签 CSV 和输入 JSON。
2. 设置 mode=mrmr 或 mode=sage，每次只运行一种方法。
3. 配置固定训练/验证窗口和与标签周期匹配的 embargo。
4. 执行入口命令并检查 run_manifest.json。
5. 按照 references/output_contract.md 消费排名和选中因子。
6. 在 Skill 外部使用锁定测试集验证完整模型、选中子集和随机子集。

## 输出产物

稳定输出包括 selected_factors.json、selection_report.md、resolved_config.json、input_manifest.json 和 run_manifest.json。mRMR 额外输出 mrmr_ranking.csv 与 mrmr_redundancy_matrix.csv；SAGE 额外输出 sage_values.csv 与 sage_metadata.json。

## 参考文件

- SKILL.md：Agent 使用说明。
- README.md / README.en.md：中英文用户文档。
- references/source_boundary.md：数据、算法与研究边界。
- references/input_schema.md：输入参数说明。
- references/output_contract.md：输出文件约定。
- references/metric_definitions.md：指标定义。
- references/validation_notes.md：验证限制。

## 边界说明

本 Skill 不生成因子、不优化模型参数、不运行组合回测。mRMR 和 SAGE 排名都不构成样本外提升或交易有效性的证明。
