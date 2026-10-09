# VIS 本地实施总计划（M0–M6）

> 目前为架构和评测方案基线；模型、服务、自录数据集与评测系统均尚未实现。8–9周为资源充足情况下的初步估计，不是交付承诺。Windows本地RTX3060优先，不要求首期Docker/K8s。

## 1. 项目边界

本仓库只响应第4/5/6项中的语音部分。规划语种zh-CN、en-US、ja-JP、ko-KR、es-ES，实际合同语言待冻结；中文使用CER、英文WER，其余按冻结统计约定。Rust承担接口、会话、鉴权、编排，Python承担推理、录音数据处理和自动化测评。关键成果为可运行的语音流程、自主数据集、可重复评测和标书真实证据，不从零训练大型基础模型。

## 2. 里程碑

| 阶段 | 预计窗口 | 具体产物 | 阶段验收Gate |
|---|---|---|---|
| M0：需求冻结 | 第1周前半 | 语种/方言、数据许可、业务知识、硬件与测试协议 | 有书面冻结条款 |
| M1：测评底座 | 第1–2周 | Workspace、manifest/schema、哈希、speaker split、CER/WER、自测和报告 | CPU测试全通过、评分可复算 |
| M2：数据V0 | 第2–3周 | 按许可导入公开音频、小批次自录/双人标注 | 至少100条有效语音的真实评价 |
| M3：单路完整链路 | 第3–5周 | ASR、RAG/LLM、三重核查门禁、TTS、Rust接口 | 一路录音→识别→讲解→核查→合成成功 |
| M4：自主数据V1 | 第3–7周并行 | 5语种约10k条规划数据、交叉复核、speaker Holdout | 数据授权/质量/封存门禁通过 |
| M5：模型对照及压测 | 第6–8周 | 准确率、MOS/SIM、安全事实指标、GPU并发和成本 | 完整LOCAL报告及失败记录 |
| M6：接口与投标材料 | 第8–9周 | OpenAPI/WSS、第三方联调、实测截图、审计和证据索引 | 每个招标点具实证或明确未达标 |

M0必须确认：合同正式语种、噪声/口音/录音终端、95%是否每语种每场景、归一化规则、目标并发、独立审核策略、第三方API允许性、录音/声音授权与可用人工语言复核资源。不可将暂定五语种或10并发默认为采购方要求。

## 3. M1首轮实施细项与证明

1. 初始化Cargo Workspace、Python uv或venv；让评分引擎在CPU上运行，先不批量装重模型。
2. 设计eval/datasets中ASR manifest JSON Schema，含source/许可、音频sha256、speaker_id、split、语言、参考文本、标注审核状态。验证解码、去重、数据泄漏。
3. 实现eval/metrics评分规则，保留S/D/I/N和micro aggregation、Unicode/文本规范化，静音与空参考分开处理。
4. 编写至少12项评测系统自测，详见[测试平台](../evaluation/test-platform.md)。
5. 导入少量许可允许的FLEURS或自主录音，完成≥100条有人工确认参考文本的有效样本。
6. 尝试在RTX3060加载Qwen3-ASR 0.6B或等效可用模型，记录量化/版本/显存/推理时间；运行100条并形成run_manifest、predictions、metrics、report。
7. M1审阅样本泄漏、指标正确、模型错误/超时、全部失败条目是否入库。无法运行则状态INCOMPLETE，**不可填上游公开成绩冒充本地成绩**。

完成标志：从真实音频到人工Gold、模型输出与WER/CER报告可以再次执行获得一致结果。100条只是工程验证，不足以证明正式95%指标。

## 4. 数据采集与工程并行

M2按语种组织合法授权录音，先完成小批次标注和人工双审，冻结Regression；M3并行接ASR/TTS/知识检索、三层核查；M4持续扩大到自录1400条/语言+公开600条/语言的规划规模，speaker划分Development/Regression/Holdout约60/20/20。公开数据使用源方规定的split且不得随意重新分配。

M5统一使用相同冻结数据与推理配置做候选对照，指标未达标则分析失败原因（具体语言、口音、噪声、关键术语），测试改善应依靠新Regression/独立Holdout复验。M6进行第三方接口负面测试和联调，生成标书证据，不在缺少测试时虚构截图。

## 5. 明确不做和安全门禁

- 不先建设大型GPU调度/K8s，不训练通用ASR/TTS基础模型。
- 未经合法授权音色不克隆；原始音频、consent材料、私有Holdout Gold、密钥、模型权重不得进入公开Git历史。
- 三重核查模型异常或证据不足时fail-closed；不允许先播报未经核查的动态事实。
- 不擅自对并发量、语种数量、识别率等作超出已验证条件的投标承诺。
- 每阶段按合同关联、版本记录、实际日志、失败样本与指标计算公式验收；不满足门禁标BLOCKED/INCOMPLETE。

## 6. 本地目录草案

```text
vis/
  apps/gateway/          # Rust service, future
  apps/test-console/     # lightweight UI, future
  services/asr/          # Python model
  services/tts/
  services/trust/
  services/explanation/
  contracts/
  eval/datasets/
  eval/runners/
  eval/metrics/
  eval/reports/
  eval/tests/
  docs/
```

当前仓库只有方案文档，所列路径是后续实现计划；不得视为已交付代码。
