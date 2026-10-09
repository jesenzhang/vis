# VIS 文档索引

> 设计草案 V1.0（2026-10-09）。服务、数据采集、识别准确率、并发和评测系统尚未本地验收。

| 文档 | 范围 | 状态 |
|---|---|---|
| [**投标语音专项正文 V2.0**](bid/README.md) | **标书第4/5/6项语音部分：技术方案、四图、数据证据、评测系统、指标、接口与承诺** | **标书审阅稿** |
| [招标响应](requirements/bid-response.md) | 招标第4、5、6项语音范围和证据矩阵 | 设计 |
| [模块化语音架构](architecture/pipeline.md) | 核心流水线、三重核查、时序与边界 | 设计 |
| [自主测试集](evaluation/datasets.md) | 来源、录制、标注、版本、封存测试 | 规划 |
| [测试系统](evaluation/test-platform.md) | 执行器、评测数据流、报告与门禁 | 设计 |
| [指标与验收](evaluation/metrics-and-acceptance.md) | CER/WER、MOS、SIM、Guard、P95与统计方法 | 设计 |
| [公开模型基准](evaluation/public-baselines.md) | 公开测试数值、出处和限制 | PUBLIC |
| [开放接口](interfaces/contracts.md) | 协议、数据、鉴权、配额、字段、联调 | 设计 |
| [硬件与费用](deployment/hardware-and-cost.md) | 本地3060、云GPU、采购和容量估算 | 预算 |
| [实施路线](implementation/roadmap.md) | M0–M6目标、门禁、阶段交付 | 规划 |
| [证据计划](evidence/evidence-plan.md) | 截图、报告、可核查性 | 待采集 |

## 证据分级

- **DESIGN**：本仓库设计与计划，不代表已实现。
- **PUBLIC**：公开论文、模型官方仓库或第三方结果，不代表本地实测。
- **LOCAL**：记录我方运行的完整原始预测、环境和数据版本的实测。
- **ACCEPTANCE**：合同约定条件下、冻结的私有留出集进行的正式验收。

所有公开图表标注链接、模型版本、数据集及获取日期；性能目标、预算数值与未运行测试不得标注PASS。

## M0必须冻结

正式支持语言及方言、讲解领域、WER/CER计量及文本归一化、噪声/设备/口音条件、并发/延迟、数据和模型许可、音色授权、联调对象、现场证据和硬件预算。未冻结前五语种只是实施假设。
