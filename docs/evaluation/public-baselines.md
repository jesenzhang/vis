# 模型公开基准测试数据（仅PUBLIC，不代表VIS实测）

> 整理时间2026-10-09。下表是模型官方或第三方公开环境中的结果，仅供候选初选。不同数据集和分词规则不能直接比较或作为“我方95%识别准确率”验收证明。

## 1. Qwen3-ASR官方基准

来源：https://github.com/QwenLM/Qwen3-ASR （Evaluation表格；沿用官方WER及统计定义）。

| 数据集 | Whisper large-v3 | Qwen3-ASR 0.6B | Qwen3-ASR 1.7B |
|---|---:|---:|---:|
| LibriSpeech test-clean | 1.51 | 2.11 | 1.63 |
| LibriSpeech test-other | 3.97 | 4.55 | 3.38 |
| MLS 8种语言汇总 | 8.62 | 13.19 | 8.55 |
| Common Voice 13语种汇总 | 10.77 | 12.75 | 9.18 |
| FLEURS 12语种汇总 | 5.27 | 7.57 | 4.90 |
| FLEURS扩展30语种汇总 | 8.16 | 21.80 | 12.60 |

注意：LibriSpeech clean上Whisper 1.51优于Qwen 1.7B的1.63；FLEURS 30语种汇总12.60不能说明“30种语言都达到95%”。中文相关官方数据是否等同本项目CER必须复测、明确统计定义。

## 2. Qwen3-TTS官方内容一致性基准

来源：https://github.com/QwenLM/Qwen3-TTS ，Seed-TTS的12Hz Base评价。

| 模型 | Seed test-zh | Seed test-en |
|---|---:|---:|
| Qwen3-TTS-0.6B-Base | 0.92 | 1.32 |
| Qwen3-TTS-1.7B-Base | 0.77 | 1.24 |
| CosyVoice3官方对照 | 0.71 | 1.45 |

第三方参考：https://github.com/OpenBMB/UltraEval-Audio/blob/main/replication/qwen3_tts.md 。其复现中Qwen3-TTS 1.7B Base英文WER 1.58、中文CER 0.87，SIM英文71.24/中文76.89（使用其自身量表）。内容一致性、MOS、说话人SIM三个指标彼此不同。

## 3. Qwen3Guard官方安全基准

来源：https://github.com/QwenLM/Qwen3Guard/blob/main/eval/README.md （指定Qwen3GuardTest任务）。

| 模型 | F1（%） |
|---|---:|
| Qwen3Guard Gen 0.6B | 83.6 |
| Qwen3Guard Gen 4B | 84.0 |
| Qwen3Guard Stream 0.6B | 81.6 |
| Qwen3Guard Stream 4B | 85.4 |

F1不等于危险内容召回率，也不等于生产安全率。必须在自主安全数据上单独评价高风险漏检和正常内容误封。

## 4. 事实与多语种公开集

- LongFact/SAFE：https://github.com/google-deepmind/long-form-factuality ，参考原子事实拆解→外部证据→支持性分类的方法，不给VIS填无实测的百分数。
- FLEURS：https://huggingface.co/datasets/google/fleurs ，明确数据版本、语言配置与CC BY 4.0署名条件。
- Common Voice：https://commonvoice.mozilla.org/ ，使用前确认数据版本、Mozilla Data Collective使用与再托管限制，不将原始素材上传公开仓库。
- LibriSpeech、AISHELL、WenetSpeech按各自数据许可使用。

## 5. 本地复现和证据

固定公开数据版本、模型权重、采样率、文本归一化、推理参数与评分版本；条件不一致就标“不可直接比较”。保存逐语种原始预测、S/D/I/N、推理环境、失败/排除样本。标书截图必须明确标注PUBLIC/LOCAL/ACCEPTANCE、来源URL、获取时间和模型版本，不能使用自制图假冒官方原图或本地系统截图。
