# skill-factor-ranking-sage

**简体中文** | [English](README.en.md)

> **项目定位 / Project Positioning**
>
> 本项目是面向 Codex / QUANTSKILLS 的模型因子筛选 Skill，用于在给定本地因子和标签 CSV 后，运行确定性回归 mRMR 或固定模型 Marginal-SAGE，输出可复现、可检查、可归档的 Top-K 因子排名。
>
> 本仓库不是因子库、HPO 工具或回测引擎，不保证被选因子能够提升样本外模型表现。

本项目作为 QUANTSKILLS 社区贡献提交，不代表官方认证、投资建议或生产交易验证。

role: skill | platform: codex | category: tooling | status: active | validation: runnable | methods: mrmr / marginal-sage

---

skill-factor-ranking-sage 是一个自包含的量化模型因子排名 Skill。运行时不依赖第三方 mrmr 或 sage 包，只保留两个相互独立的方法：

- **mRMR**：使用回归 F 统计量衡量 relevance，使用绝对 Pearson 相关衡量 redundancy，通过贪心 quotient 规则选择 Top-K。
- **Marginal-SAGE**：训练一次固定 LGBM 或 MLP，在验证样本上通过经验边际填充和随机排列估计全局 MSE 贡献及标准误。

第三方 mrmr 和 sage 包只用于可选一致性测试，不是运行依赖。

## 这个 Skill 解决什么问题

本 Skill 适合处理：

- 从本地多因子矩阵生成确定性的 mRMR Top-K 排名
- 对固定 LGBM / MLP 估计全局 Marginal-SAGE 因子贡献
- 输出输入哈希、时间窗口、配置副本、排名 CSV 和运行清单
- 为后续独立的全量、子集和随机基线实验提供候选因子集

本 Skill 不负责：

- 生成 Alpha 因子或标签
- 搜索 LGBM / MLP 超参数
- SHAP、Permutation Importance、drop-one 或 retrain coalition
- 选择最优 K、组合构建、交易成本或回测
- 证明选中子集能够提高样本外收益

## 方法差异

| 维度 | mRMR | Marginal-SAGE |
|---|---|---|
| 输入数据 | 训练行 | 训练行和验证行 |
| 是否训练模型 | 否 | 是，固定 LGBM 或 MLP |
| 主要目标 | 高 relevance、低 Pearson redundancy | 固定模型全局 MSE 贡献 |
| 随机性 | 无 | 有，受样本和排列影响 |
| 主要诊断 | relevance、冗余矩阵 | sage_value、sage_std、收敛信息 |
| Top-K含义 | 贪心mRMR顺序 | SAGE值降序前K名 |
| 是否保证重训后提升 | 否 | 否 |

## 工作流

~~~text
1. 准备本地 factor CSV 和 label CSV
2. 配置固定训练/验证窗口与 embargo
3. 选择 mode=mrmr 或 mode=sage
4. 先运行对应 smoke 配置
5. 运行真实输入 JSON
6. 检查 run_manifest、排名、标准误和警告
7. 在 Skill 外部锁定测试集
8. 比较全量、Top-K 和多个随机 Top-K 基线
~~~

## 仓库内容

~~~text
skill-factor-ranking-sage/
├── SKILL.md
├── README.md / README.en.md
├── LICENSE
├── requirements.txt
├── agents/
│   ├── openai.yaml
│   ├── cursor-rule.mdc
│   └── portable-loader.md
├── examples/
│   ├── factor_selection_mrmr_smoke.json
│   ├── factor_selection_lgbm_smoke.json
│   ├── factor_selection_mlp_smoke.json
│   ├── toy_factors.csv
│   └── toy_labels.csv
├── references/
│   ├── source_boundary.md
│   ├── input_schema.md
│   ├── output_contract.md
│   ├── metric_definitions.md
│   ├── validation_notes.md
│   └── algorithm_sources.md
├── scripts/
│   ├── run_factor_selection.py
│   └── factor_selection_runtime/
└── tests/
~~~

## 数据要求

至少提供两个本地 CSV：

- feature_path：包含日期列、ticker列和数值型因子列
- label_path：包含相同日期列、ticker列和标签列

约束：

- 每行表示一个 (date, ticker) 样本
- 两个文件内部的 (date, ticker) 必须唯一
- 日期支持 YYYYMMDD 或可解析日期字符串
- 因子列必须为数值类型
- runtime按键执行inner join，并删除标签缺失行
- available_date可选；缺失时会报告point-in-time风险，不能证明无未来信息

完整字段见 [references/input_schema.md](references/input_schema.md)。

## 快速开始

安装依赖：

~~~bash
python -m pip install -r requirements.txt
~~~

运行三个smoke配置：

~~~bash
python scripts/run_factor_selection.py --input examples/factor_selection_mrmr_smoke.json
python scripts/run_factor_selection.py --input examples/factor_selection_lgbm_smoke.json
python scripts/run_factor_selection.py --input examples/factor_selection_mlp_smoke.json
~~~

运行自定义配置：

~~~bash
python scripts/run_factor_selection.py --input <input-json>
~~~

## 核心输入变量

### 通用字段

| 字段 | 类型 | 说明 |
|---|---|---|
| run_name | string | run ID前缀 |
| output_root | string | 输出根目录；每次生成runs/<run_id> |
| random_seed | integer | SAGE模型、背景、验证抽样和排列随机种子 |
| mode | string | mrmr或sage |
| selection_count | integer | 输出Top-K数量 |
| input.feature_path | string | 因子CSV路径 |
| input.label_path | string | 标签CSV路径 |
| data.date_col | string | 原始日期列名 |
| data.ticker_col | string | 原始ticker列名 |
| data.label_col | string | 标签列名 |
| data.feature_include | list[string] | 可选因子白名单 |
| data.feature_exclude | list[string] | 可选因子排除列表 |

### 固定窗口

| 字段 | 说明 |
|---|---|
| validation.train_start / train_end | 训练闭区间 |
| validation.valid_start / valid_end | 验证闭区间 |
| validation.embargo_days | 两窗口间至少需要存在的交易日期数量 |

mRMR只使用训练行；SAGE在训练行拟合模型并在验证行估计贡献。

### mRMR字段

| 字段 | 可选值 |
|---|---|
| mrmr.relevance | 固定为f |
| mrmr.redundancy | 固定为c |
| mrmr.denominator | mean或max |

### SAGE字段

| 字段 | 说明 |
|---|---|
| model.type | lgbm或mlp |
| model.params | 直接传给对应模型适配器的参数 |
| sage.loss | 当前固定为mse |
| sage.background_size | 经验边际填充背景行数 |
| sage.evaluation_size | 验证候选池最大行数 |
| sage.batch_size | 每轮采样数量 |
| sage.n_permutations | 样本/排列预算；null表示依赖收敛检测 |
| sage.detect_convergence | 是否允许满足阈值后提前停止 |
| sage.convergence_threshold | 最大标准误与贡献范围之比的停止阈值 |

## 输出产物

每次运行写入：

~~~text
output_root/runs/<run_id>/
├── selected_factors.json
├── selection_report.md
├── resolved_config.json
├── input_manifest.json
└── run_manifest.json
~~~

mRMR额外输出 mrmr_ranking.csv 和 mrmr_redundancy_matrix.csv。SAGE额外输出 sage_values.csv 和 sage_metadata.json。

完整字段见 [references/output_contract.md](references/output_contract.md)。

## 如何解释结果

- mRMR的relevance是单变量回归F统计量，不是LGBM增量贡献。
- SAGE的sage_value是固定模型在不同联盟中的平均MSE贡献，不是删除因子后重训的损失变化。
- SAGE的sage_std较大或converged=false时，不应把相邻排名解释为稳定顺序。
- selection_count是用户提供的研究假设，runtime不会证明K最优。
- Top-K应在锁定样本外数据上与全量因子和多个随机Top-K基线比较。

## 验证范围

当前验证等级为 runnable：

- 三个toy smoke工作流可运行
- 本地mRMR与参考包在可选parity测试中对齐
- 本地Marginal-SAGE与参考包在可选parity测试中对齐
- 核心配置、输出契约和移除功能边界有自动化测试

该等级不表示mRMR或SAGE能够提高样本外RankIC、MSE、组合收益或交易收益。

## 项目状态与风险边界

- **项目状态**：Community Project，未经QUANTSKILLS官方审核、认证或背书。
- **数据来源**：仓库只包含为 smoke test 人工构造的合成 toy data，不对应真实证券、行情或账户；真实因子、标签和行情由用户提供，并由用户负责许可与合规。
- **时间边界**：runtime无法仅凭CSV证明因子可用时间和标签窗口没有未来信息。
- **算法边界**：只提供有限复现的回归mRMR和Marginal-SAGE，不提供通用特征选择平台。
- **研究边界**：输出仅为因子排名候选，不代表模型提升、交易有效性或生产可用性。
- **用途**：仅供量化研究、教育和方法论参考，不构成投资建议。

## License

This project is licensed under the GNU General Public License v3.0. See [LICENSE](LICENSE).
