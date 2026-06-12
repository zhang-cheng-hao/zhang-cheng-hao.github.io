# 历史材料索引

整理日期：2026-06-12

本文记录本地历史材料的来源和用途。原始转录、完整面经、PPT、论文草稿和生成产物包含大量私人准备信息，已通过 `.gitignore` 排除，不直接提交到 GitHub。

## 1. 本地材料分区

| 路径 | 内容类型 | 当前用途 | 是否提交 |
|---|---|---|---|
| `generated/interview-intel/` | 面试情报库、问题地图、薄弱点、项目讲稿 | 面试准备核心源材料 | 否 |
| `generated/interview-cleanup/` | 多场面试 ASR 纠错和通顺整理 | 回查完整上下文 | 否 |
| `generated/xiaohongshu-interview/` | 单场面试转录、分析、渲染产物 | 方向匹配和表达复盘 | 否 |
| `materials/interview/` | 原始面试资料、PPT、论文和转录 | 原始证据和备份 | 否 |
| `projects/zhangchenghao-cv/` | CV、面试速记、相关材料 | 简历和面试素材 | 否 |
| `papers/` | 论文模板和草稿 | 研究写作材料 | 否 |

## 2. 已沉淀为文档的内容

已整理到 `docs/interview-prep-handbook.md`：

- 总体面试定位。
- 项目资产矩阵。
- CREM、CSGR、MedAgent-R1、UBioRec 的公开讲法。
- 高频追问和风险口径。
- 技术薄弱点和表达问题。
- 7 天准备计划和面试前 checklist。

未直接公开的内容：

- 原始 ASR 文本。
- 面试官原话和逐字转录。
- 公司内部资源、流程和团队细节。
- 未核验的业务指标、内部奖项和实验数字。
- 私人反思、求职状态和面试流程细节。

## 3. 维护规则

1. 原始材料继续放在 `generated/`、`materials/` 或 `projects/`，不要直接解除 `.gitignore`。
2. 能公开复用的结论才进入 `docs/`。
3. 任何数字进入公开文档前，必须标注来源和口径；不确定数字只保留“需核验”。
4. 面试复盘先写入本地材料，再定期抽象成稳定模板。
5. 项目讲法优先更新 `docs/interview-prep-handbook.md`，避免散落在多个临时文件里。

## 4. 后续建议

- 新增 `docs/metric-fact-table.md`：集中维护所有可讲数字和不可讲数字。
- 新增 `docs/coding-interview-templates.md`：沉淀 C++ 高频代码题模板。
- 新增 `docs/rl-post-training-card.md`：整理 PPO、DPO、GRPO、DAPO 和 MedAgent-R1 速答卡。
