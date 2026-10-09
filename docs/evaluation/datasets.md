# 自主测试数据集构建方案（DATASET-PLAN V1）

> 本文所有数量是 **建设规划目标**；尚未采集。正式语种按 M0 需求冻结，当前假定 `zh-CN/en-US/ja-JP/ko-KR/es-ES`。采用合法来源、人工金标准、说话人隔离和独立留出集；不能用合成语音声称真实场景 95% ASR 准确率。

## 1. 测试集目录和版本

| 数据集 | 数据内容 | V1规模 | 金标准与用途 |
|---|---|---:|---|
| ASR-BENCH | 五语种流式/非流式，噪声/口音/专业词 | 5×2,000=10,000条 | 人工文本；CER/WER/LID |
| TTS-BENCH | 每语种250句文本与合成任务 | 1,250段文本 | 目标文本；TTS错误率/MOS |
| CLONE-BENCH | 授权说话人（初期20人） | 每人参考音频+目标语句 | 音色SIM、人评、跨语言 |
| SAFETY-BENCH | 正常/违规/边界/注入 | 2,000条 | 人工标签；P/R/F1 |
| FACT-BENCH | 支持/矛盾/证据不足/复杂 | 1,000问 | 原子事实、证据与状态 |
| VOICE-E2E | 100段×≥5轮会话 | ≥500轮 | 行为/时延/安全门禁 |

最初先建设 V0（每语种公开100+自录20条），打通录入、标注、评分再扩容。公共和自录数据独立报告，不混为“自有全部采集”。

## 2. ASR 数据来源与采样

每语种 2,000 条的规划为：**公共测试集 600 条 + 自主录音 1,400 条**。自录部分建议≥35名说话人×40条；讲解/数字/地名/专业术语≥300条应计入 2,000 条中的标签子集，不应叠加重复计算。平均20秒估算全五语种约55.6小时，仅为容量规划。

- 公共候选：FLEURS（102语种，CC BY 4.0，保留署名），Common Voice（数据许可须按选用版本/MDC使用条款核查），LibriSpeech、AISHELL、WenetSpeech（按各自许可）；必须记录 `dataset_release`, `source_url`, `license_text`, `download_at`, `original_split`, `checksum`。
- 自录：真实、获得明确授权的人类录音；覆盖室内安静/室外真实噪声/远场/不同麦克风/自然口语/不同语速/口音。尽量平衡说话人贡献，不以设备增强替代真实采样。
- 发音提示：日/韩/西等必须有对应语言能力的标注审核人员；复杂音频经两人复核。口音标签应采集自愿声明或语音专家按明确标准标注，不从声音推断敏感身份。
- 数据增强：SNR、混响、压缩和噪声混合**另作鲁棒性集**；同一原始录音的增强版本固定在同一 split，不作为新独立说话人/样本计数。

### 分组和金标准

自录 1,400 条/语种按 **speaker_id** 分组推荐 Development 60%（约840）、Regression 20%（约280）、Sealed Holdout 20%（约280）；保证同一人的数据不跨禁止分组。分组在采集后按说话人粒度实现，因此实际条数可有偏差。公开数据保持发布者原始 split；严禁公开测试集混入自录 holdout 后再次随机分组。

正式验收集需要封存清单与 sha256；普通开发者不得查看 Gold；任何基于 Holdout 反馈作出的调优都需要新建独立 Holdout 复验。逐语言至少报告总字符/词数量、录音小时数、参与说话人数和场景覆盖。280 条/语种 Holdout 未必足以稳定证明5%上限，验收样本量最终应按方差、误差结构和置信区间预估，必要时追加样本。

### 录音协议

1. 出示用途及撤回政策并取得书面或可审计的数字授权；声音克隆授权**独立于ASR收集授权**。
2. 使用至少16kHz单声道PCM/WAV的受控录音或保存原始高质量录音及转换记录，避免静音裁剪破坏字首；采集源、设备、时间、语言、环境、音频时长和哈希。
3. 预筛：损坏、重复、版权/授权异常、无语音、极端失真以及语言不符样本标记为 `REJECTED`，不得静默丢弃困难但有效的语音。
4. ASR仅提供候选转写；人工听音改正数字、术语、外来词、语气词、遗漏。
5. 双人交叉审核，冲突仲裁，记日志，最终确认为 `verified`；对听不清内容使用标准未知标记且定义排除/计分规则。
6. 将数据冻结为带版本号的 manifest，并审计来源许可。

## 3. 元数据 Schema 与实例

存储建议为 `manifest.jsonl`，原始音频放在仓库外受控目录/对象存储；文件路径为**相对路径**，不放个人身份信息。

```json
{"sample_id":"asr-zh-000001","dataset_id":"asr-private","dataset_version":"1.0.0","split":"development","language":"zh-CN","speaker_id":"spk-001","audio_path":"audio/zh-CN/000001.wav","audio_sha256":"<actual sha256>","duration_ms":5220,"sample_rate":16000,"reference_text":"请介绍一下这件文物的历史。","environment":"indoor_quiet","device_type":"mobile","scenario_tags":["history","question"],"source":"self_recorded","license_id":"consent-001","annotation_status":"verified"}
```

上述是假数据 Schema 示例，不计入评测结果。另以`consent_id`映射独立受控授权库；开发报告只引用匿名编号。`audio_sha256`要求真实64位十六进制值；Schema验证必须拒绝占位符、缺少许可、缺失音频及重复 sample_id。

**禁止提交到Git**：原始/处理后的含人声录音、授权书、音色嵌入/权重、私有Holdout标准答案、邮箱/姓名、API密钥。仓库只提交 Schema/字典/导入工具/统计与经批准可公开的去标识化摘要。

## 4. TTS 与声音克隆数据

- TTS：每语种250条文本，覆盖问答、历史讲解、数字日期、混合语言、断句、缩写、术语与罕见字；固定文本版本，不同模型使用完全相同的文字。
- CLONE：首期20位**独立授权**说话人，为每人准备3s/5s/10s参考片段，至少20条目标语句。参考与生成句内容不得重合；按说话人/语种报告。
- TTS自动测：用**独立固定ASR**转写生成音频后计算 CER/WER、DNSMOS（辅助）；采用人工MOS及固定说话人嵌入的SIM评估克隆，不能以SIM代替人的听感。
- 记录voice_id、consent_id、目标语言、参考音频哈希、模型配置、生成音频哈希及授权撤销状态。

## 5. 安全与事实核查集

### SAFETY 2,000 条（数量规划）

分组需同时覆盖正常业务真实分布和相对均衡的风险挑战集，避免仅报告在人工平衡样本上的F1。字段：`case_id`, `input_or_output`, `risk_type`, `severity`, `expected_action(PASS/REVIEW/BLOCK)`, `rationale`, `annotator_ids`。高风险标签需双审核；跨语言同义翻译样本属于同一 case family，必须同 split。

### FACT 1,000 题（数量规划）

400 充分证据、250含矛盾事实、250证据不足、100多源/时效/多语言关联。每道题标注原子事实及证据，格式包含：`question_id`, `language`, `question`, `atomic_claims[]`, `gold_label`, `evidence_id`, `evidence_source`, `evidence_version`, `expected_decision`, `reviewer_ids`。对 `INSUFFICIENT` 不得凭模型自信度改判 `SUPPORTED`；证据文件本身需要版本控制和权限。

## 6. E2E 场景

100段多轮会话（≥500轮）：语种检测/切换、自然提问、专有名词、RAG命中与无证据、三重核查拒答、授权声音、用户打断、重连、模型故障、TTS取消；每段存输入文件/步骤/预期行为/禁止行为/耗时测点。测试者必须确认自动合成的语音不会用来冒充真人音频覆盖ASR验收。

## 7. 建设验收门禁

- 100%输入文件可解码；100%样本字段/哈希通过校验；100%正式 gold 经人工确认，Holdout 双人复核；来源与授权100%可追踪。
- 无违规跨 split 重复、同说话人泄漏、同一段音频不同变换进入不同 split。
- 数据版本冻结，采样统计按语种/说话人/设备/场景导出；有完整拒绝/剔除统计，禁止选择性剔除难例。
- 参与人员、采集预算和语种审核资源未确认时不得认定 V1 已完成。

## 8. 数据许可核验入口

- FLEURS: https://huggingface.co/datasets/google/fleurs （CC BY 4.0，署名要求）
- Common Voice / MDC: https://commonvoice.mozilla.org/ ; https://commonvoice.mozilla.org/en/terms （CC0与平台获取/再分发限制并存；按当期条款核查）
- Open datasets/model rights: 逐版本维护许可证清单，不得将公开可下载等同可任意再分发/商用。