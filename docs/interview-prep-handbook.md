# 历史对话整理：面试准备手册

整理日期：2026-06-12

本文把本项目中历史对话和生成材料沉淀为可长期维护的面试准备手册。原始材料主要来自本地 `generated/`、`materials/interview/` 和 `projects/` 下的面试转录、复盘和项目讲稿；这些目录按仓库规则不提交到 GitHub。本文只保留结构化结论、公开表达口径和后续行动项。

## 1. 总定位

当前经历可以统一成一条主线：

> 大模型在检索、推荐、RAG 和 Agent 场景中的后训练、表征学习与业务闭环。

面试中不要把自己讲成“纯推荐”或“纯论文”。更稳的定位是：

- 有业务落地经验：做过推荐/搜索链路上游的多模态表征、数据构造、训练、离线评估和线上配合。
- 有研究扩展能力：围绕自监督 retriever、source memory 压缩、跨卡候选池扩展等问题做过方法探索。
- 有 RAG/Agent 后训练经验：关注 evidence faithfulness、工具调用、SFT/RL 训练和 reward 设计。
- 有明确待补短板：RL 基础、VLM 对齐细节、attention 工程、推理系统、现场代码题。

## 2. 项目资产矩阵

| 项目 | 面试定位 | 最适合场景 | 讲述重点 | 风险点 |
|---|---|---|---|---|
| CREM / 多模态推荐粗排表征 | 默认主项目 | 推荐、搜索、电商、多模态、后训练 | 业务相关性、LLM judge 样本构造、compression token、OpenQA 辅助监督、上线闭环 | 数字口径、贡献边界、FlashAttention/SDPA 追问 |
| CSGR / 自监督 retriever 训练 | 技术深挖项目 | RAG、检索、Agentic search、研究岗 | NTP-supervised retriever、full source exposure、compressed source memory、bottleneck replay、跨卡候选池 | 不要把问题讲成“没用 FlashAttention”；实验数字需统一 |
| MedAgent-R1 / 医疗 RAG Agent RL | RL/RAG 项目 | 后训练、RL、Agent、客服/知识库、医疗教育 | outcome-only RL 的 faithfulness 风险、observation-masked SFT、faithfulness-gated GRPO | RL 基础、reward 细节、个人贡献边界 |
| UBioRec / 用户小传压缩 | 辅助项目 | 个性化、推理成本、用户理解 | task-oriented compression、长上下文 prefill 成本、离线压缩表征 | 不要讲成完整项目 owner；不要宣称无损压缩 |
| 图学习论文 | 补充科研经历 | 图学习相关、论文产出追问 | 科研训练、论文状态、方法概括 | 不宜占主叙事时间 |

## 3. 面试项目选择顺序

默认顺序：

1. CREM：所有方向的默认主项目。
2. CSGR：对方偏检索、RAG、Agent、研究方法时展开。
3. MedAgent-R1：对方偏 RL、RAG、证据忠实、客服/知识库时展开。
4. UBioRec：被问个性化、用户理解、推理降本时补充。
5. 图学习：只作为论文和科研训练补充。

按岗位切法：

- 搜索/推荐/电商：CREM 主讲，把 item-item 相关性迁移到 query-item 相关性。
- 后训练/RL：CREM + MedAgent-R1，强调数据构造、SFT/RL、reward 和评估。
- RAG/Agent：CSGR + MedAgent-R1，强调 retriever、工具调用和 evidence faithfulness。
- 多模态/VLM：CREM 主讲，准备 ViT、projector、learnable queries、token 插入方式追问。
- 推理系统/效率：UBioRec + CSGR，强调压缩、prefill、source memory、replay。

## 4. CREM 讲法

### 4.1 30 秒版本

我在快手做的主项目是推荐链路上游的多模态表征训练，可以理解为粗排/召回阶段的 embedding 模型。原来的正样本主要来自用户行为共现和图文表面相似，但共现会有长尾稀疏和热门商品噪声，图文相似又容易把外观相似但用途不同的商品拉近。我们用业务行为采样候选 pair，再用 LLM judge 做语义过滤，构造更干净的正样本；训练上结合对比学习和 OpenQA 辅助任务，让 compression tokens 既能做检索表征，也保留可回答问题的语义信息。我主要负责数据 pipeline、模型和 tokenizer 适配、special mask、pooling/loss 训练、case 分析、离线评估和上线配合。

### 4.2 深挖结构

1. 业务位置：推荐链路上游粗排/召回，用 embedding 先召回 TopK，再交给下游精排。
2. 核心问题：共现信号不干净，图文相似容易误判，业务相关性和表面相似不一致。
3. 数据方案：业务候选提供线上分布，LLM judge 提供语义过滤，不完全依赖 LLM 凭空造样本。
4. 训练方案：retrieval 分支用 in-batch negatives；OpenQA 分支用 answer CE 给 compression tokens 更密集监督。
5. 工程方案：扩 tokenizer/processor，记录 `sep_indices`，改 4D causal mask，从 compression token hidden states 做 pooling。
6. 验证方式：消融、离线召回 case、训练稳定性、线上 A/B 分层讲，不混数字。
7. 复盘：收益主要来自更干净的语义 pair 构造和压缩监督，而不是单纯堆模型结构。

### 4.3 高频追问准备

- 项目在推荐链路哪一层？输入、输出、下游是什么？
- 旧方法的核心问题是什么？为什么共现和图文相似不够？
- LLM judge 如何构造正样本？如何控制偏差？
- OpenQA 数据怎么拼接？`labels=-100` 和 attention mask 分别做什么？
- compression token 是 4、8 还是 16 个？分别对应哪个实验？
- 为什么不直接让 QA 看完整上下文？如果不改 mask 会怎样？
- FlashAttention、SDPA、FlexAttention 与自定义 4D mask 是什么关系？
- 线上收益属于离线指标、小流量 A/B 还是大盘影响？归因边界是什么？
- 个人贡献是什么？哪些是团队已有工作或他人负责？

## 5. CSGR 讲法

### 5.1 30 秒版本

CSGR 是我后续主导的自监督 retriever 训练工作。它利用语言模型 next token prediction loss 给 retriever 提供监督：如果某个 source 能降低 target 的 NTP loss，就说明它对当前文档有帮助。但 baseline 需要 full source exposure，reader 要看其他文档的大量 token hidden states，同时 retriever、similarity、source memory、reader loss 连成一个很重的计算图，候选池很难跨卡扩展。我的方法用 compressed source memory 替代 full-token source hidden states，并在 embedding/source memory 这些 bottleneck 上 detach、all-gather、流式算中间梯度，再 replay 前段恢复参数梯度。

### 5.2 必须讲清的两层问题

- Source interface 问题：reader 到底需要暴露多少 source memory，不是仅靠 attention kernel 可以解决。
- 跨卡梯度问题：扩大候选池时不能把全计算图直接 all-gather，需要在 bottleneck 上缓存中间变量和梯度。

### 5.3 FlashAttention 追问口径

不要说：

> 因为没用 FlashAttention，所以需要 CSGR。

应该说：

> FlashAttention/SDPA/FlexAttention 是 attention kernel 和 mask 表达层面的优化，可以降低显存常数；CSGR 解决的是 source interface 暴露多少 memory，以及跨卡候选池如何保持可回传梯度的问题。两者是互补关系。

## 6. MedAgent-R1 讲法

### 6.1 30 秒版本

MedAgent-R1 关注医疗 RAG Agent。这个场景不只要求最终答案正确，还要求答案里的 claim 能被检索证据支持。我们观察到 outcome-only RL 可能提升最终准确率，但会让模型更倾向于从参数记忆里直接回答，甚至编造 evidence，导致 faithfulness 下降。解决思路是先做 observation-masked SFT，让模型学会工具调用但不学习生成环境 observation；再用 GRPO，并在 reward 里加入 faithfulness gate：如果 answer claim 不被 evidence 支持，即使答案对，也不给主要 accuracy reward。

### 6.2 贡献边界

稳妥表达：

> 这个项目不是我单独主导 reward 设计。核心 reward 框架由团队共同讨论/一作主导，我主要负责训练实现、实验跑数、case 分析和问题定位。

### 6.3 RL 基础速答

- PPO：on-policy policy gradient，通常需要 critic/value model 估计 advantage，稳定但训练成本高。
- DPO：偏好学习，不显式 rollout，用 chosen/rejected pair 直接优化 policy 与 reference 的 log-ratio。
- GRPO：同一 prompt 采样多条 response，用组内相对 reward 做 advantage，省掉 critic，适合规则 reward/R1 类训练。
- DAPO：需要补细节后再讲，至少要能说明它针对 GRPO/RL 训练中的长度偏置、样本有效性和 advantage/clip 等问题做改进。

## 7. UBioRec 讲法

稳妥定位：

> 这是团队的多任务用户理解项目，我主要负责用户小传压缩训练。

讲述顺序：

1. 用户小传很长，线上每个任务完整读取会增加 prefill 成本和延迟。
2. 压缩目标是 task-oriented compression，不是通用无损压缩。
3. 离线把长用户画像压成少量 special token 或固定表征，线上下游任务复用。
4. 换任务后需要重新验证，不保证一次压缩适配所有任务。
5. 个人贡献是模型适配、压缩训练和初步评估，不把完整多任务系统和业务指标都说成自己主导。

## 8. 主要薄弱点

### 8.1 技术基础

- RL 方法横向比较：PPO、DPO、GRPO、DAPO、RLHF/RLAIF、reward model、KL penalty、advantage、clip。
- VLM 对齐：ViT、projector、Q-Former/learnable queries、图文 token 插入、冻结/解冻策略、LoRA。
- Attention 工程：4D causal mask、FlashAttention、SDPA、FlexAttention、自定义 block mask、loss mask vs attention mask。
- 推理系统：KV cache、prefix cache、cache hit、offload、vLLM paged attention、prefill/decode 成本。
- 代码题：双指针、区间、二分、TopK、滑窗、栈队列、LRU、DFS/BFS、DP、字符串。

### 8.2 表达问题

- 不要用“效果不错”“涨了不少”这类虚词，必须拆成指标、范围、流量、周期、归因边界。
- 不要主动说“技术深度不高”。可以说“难点不在复杂结构，而在把业务数据变成可训练、可上线、可评估的信号”。
- 不要把团队成果说成个人完全提出。每个项目都要准备个人贡献边界。
- 不要在业务匹配面太早讲论文；先回答对方岗位业务，等面试官追问再展开研究工作。

### 8.3 数字口径

所有公开讲述前必须统一：

- CREM A/B 观察周期、流量比例、指标名称、业务范围。
- CREM 离线 AUC、benchmark、MMEB、in-batch ACC 等指标层级。
- compression token 数量：CREM 与 UBioRec 分别是多少，分别用于哪个实验。
- MedAgent-R1 accuracy、fabrication、EC-F1、safety 等数字。
- CSGR 候选池、batch、显存、模型规模、CoIR 分数。
- teacher model、内部奖项、训练资源等不确定信息不主动讲。

## 9. 7 天准备计划

### Day 1：统一事实和数字

- 做一页《数字口径表》：每个数字标注“可公开讲 / 只内部理解 / 不确定不要讲”。
- 特别核验 CREM、MedAgent-R1、CSGR 的所有关键指标。

### Day 2：RL 补课

- 补 PPO、DPO、GRPO、DAPO、KL、advantage、reward model。
- 把 MedAgent-R1 准备成 30 秒、2 分钟、5 分钟三个版本。

### Day 3：CREM 工程深挖

- 画出数据 pipeline 和 attention mask 分块图。
- 准备 FlashAttention/SDPA/FlexAttention 追问。
- 准备 hard negative、InfoNCE、temperature、batch size、GradCache 回答。

### Day 4：CSGR 和检索理论

- 统一 baseline、source exposure、compressed memory、replay、显存和效果口径。
- 准备“为什么不是 FlashAttention 能解决”的答法。

### Day 5：VLM 与推理系统

- 补 VLM 对齐和 high-density token 的位置编码/attention 问题。
- 补 KV cache、prefix cache、offload、vLLM 基础。

### Day 6：代码题恢复

- 手写并复述 12 道模板题：移动零、合并区间、二分查找、TopK、无重复最长子串、有效括号、LRU、岛屿数量、层序遍历、编辑距离、LIS、合并 K 个链表。

### Day 7：模拟面试

- 做一次项目深挖模拟，一次 RL/代码/业务混合模拟。
- 每个主项目录一版 2 分钟讲述，检查数字、贡献边界和业务匹配是否清楚。

## 10. 面试前 30 分钟 checklist

- 今天岗位属于后训练、RL、Agent、推荐、多模态、基模还是业务算法？
- 只看一页数字口径表，不临场编数字。
- CREM：业务问题 -> LLM judge pair -> compression token + OpenQA -> 工程贡献 -> 验证 -> 复盘。
- CSGR：NTP 自监督检索 -> full source exposure / cross-GPU bottleneck -> compressed memory + replay。
- MedAgent-R1：outcome-only RL 提高准确率但伤害 faithfulness -> masked SFT + gated GRPO。
- 贡献边界：CREM 主责；UBioRec 压缩模块；MedAgent-R1 训练/实验；CSGR 主导但有 mentor/合作者。
- 准备 3 个反问：业务指标、训练/评测闭环、实习生 ownership。
