# GPU部署资源配置与费用估算（规划版）

> 日期2026-10-09。所有硬件容量为规划条件，非当前已完成性能测试或供应商正式报价；币种人民币，价格可能随地区、税费、库存和汇率变化。

## 1. 部署决策

首期本地Windows和RTX3060 12GB用于ASR/TTS单路和小模型测试；评测评分与数据整理应能在CPU运行，不要求全部模型同时常驻3060显存。Rust运行网关与会话管理，Python提供模型推理/评测。后续可以将GPU推理迁移到Linux服务器、云GPU或混合部署，不改变客户端API。

| 级别 | GPU | CPU/RAM | 存储 | 规划用途 |
|---|---|---|---|---|
| P0开发 | RTX3060 12GB | 8核/32GB | 1TB NVMe | 小模型、单路、功能验证 |
| P1试点 | L4 24GB或同等 | ≥16核/64–128GB | 2TB NVMe | 部分模型独立服务、低并发 |
| P2标准 | 2×24–48GB GPU | ≥24核/128GB | 2–4TB NVMe | ASR/TTS分卡、负载扩展 |
| P3私有LLM增强 | 多张48GB级GPU起 | 按模型配置 | 独立存储/备份 | 部分/全部私有化、多租户 |

实际显存要计入模型权重、KV Cache、音频缓冲区、模型复用、量化格式、并发与碎片开销。24GB显存**不自动保证**同时运行ASR+TTS+Guard+LLM，也不代表10路在线语音推理。生产环境还需评估ECC、冗余电源、故障恢复和备件。

## 2. 可配置参数草案

```yaml
service_mode: hybrid
hardware:
  cpu_cores: 16
  ram_gb: 64
  gpu: {model: "NVIDIA L4", count: 1, vram_gb_each: 24}
  nvme_gb: 2000
workload:
  target_concurrent_sessions: 10    # 待压测
  runtime_hours_per_day: 24
  runtime_days_per_month: 30
  monthly_asr_hours: 1000
  monthly_tts_characters: 2000000
models:
  asr: {id: "Qwen3-ASR-0.6B", execution: local}
  tts: {id: "Qwen3-TTS-0.6B", execution: local}
  safety: {id: "Qwen3Guard-0.6B", execution: local}
  fact_check: {execution: external_llm, fail_closed: true}
acceptance:
  per_language_error_rate_max: 0.05
  fixed_content_first_audio_p95_ms: 2500
```

以上参数不是现有运行配置，不保证模型均可同时加载；必须输出模型版本、驱动、量化、峰值显存和P95后才能给正式容量。

## 3. 云GPU与月度预算

每月GPU费用=租用单价USD/h×GPU数量×每日运行小时×运行天数×实际汇率。历史粗估参考Runpod公开租赁价格：https://www.runpod.io/pricing 。RTX A5000、L4、A40、L40S是候选GPU，正式采购前需获取实时报价。参考1USD≈¥6.70、720h/月时：

| GPU候选 | 单价参考USD/h | 仅GPU月费估算 |
|---|---:|---:|
| RTX A5000 24GB | 0.27 | ¥1,303 |
| L4 24GB | 0.49–0.59 | ¥2,365–2,847 |
| A40 48GB | 0.49–0.59 | ¥2,365–2,847 |
| L40S 48GB | 1.09 | ¥5,259 |

**每月运营费还包括** CPU/RAM、网络、存储、外部模型调用、内容核查、税费、监控、维修和人员。

| 档位 | 一次性采购初估 | 月运行粗估 | 说明 |
|---|---:|---:|---|
| P0复用设备 | ¥0–8,000增购 | 自用电费/短时租赁 | 已有3060优先 |
| P1混合部署 | ¥15,000–35,000 | ¥3,400–7,000 | 外部LLM另计实际用量 |
| P2标准 | ¥45,000–120,000 | ¥9,600–15,700 | 双卡与服务分离 |
| P3全私有增强 | ¥150,000–400,000+ | ¥23,000–35,000+ | 高可用需冗余独立核算 |

采购价区间并非同等规格报价，也不代表承载能力。3年TCO要单独核算折旧/采购、设备电力与PUE、维修、云GPU和API成本；避免双计采购价和折旧。

## 4. 容量验证门禁

P0真实测单路ASR/TTS/Guard显存、RTF、首包和质量，不能将量化后的结果套用到FP16。P1对相同数据和参数测1/2/4/8路；P2按瓶颈拆卡而非直接上多GPU。≥24h长稳态覆盖网络故障、GPU OOM、服务重启及会话取消。经过30天真实负载再评估自购或租赁。

记录：GPU型号、驱动、框架、推理精度、模型权重哈希、音频长度、QPS、流式chunk长度、平均/P95/P99、峰值显存、成本、失败率。指标不达标必须如实说明并调整规格。
