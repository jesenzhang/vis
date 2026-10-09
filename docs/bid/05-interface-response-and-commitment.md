# 语音专项接口技术指标、数据互通与联调承诺

**依据招标第6项 · 技术响应 V2.0（设计规范，待接口实现/联调验证）**

> 仅涵盖VIS语音专项，不宣称对整个数据中台或第三方知识产权提供无限开放。

## 1. 互联互通总体原则

对语音识别、实时转写、语音合成、授权音色管理、智能讲解及三重核查建立统一开放API。支持第三方Web/移动端/导览设备接入，客户端无需绑定特定模型供应商或使用不可公开的专有SDK。对不同模型及部署环境封装稳定接口协议，按API版本和能力清单管理兼容性。

## 2. 通信协议、数据格式和连接约束

| 类别 | 采用协议 | 数据格式 | 功能场景 |
|---|---|---|---|
| 业务调用/能力查询 | HTTPS REST | JSON UTF-8，OpenAPI 3.1 | 识别、TTS、讲解、管理 |
| 音频文件提交 | HTTPS multipart | WAV/PCM，按能力扩展MP3 | 非流式识别/克隆参考 |
| 实时流式ASR | WSS | JSON控制事件 + 二进制音频帧 | 实时识别及字幕 |
| 双向实时讲解 | WSS；支持约定的WebRTC适配 | 音频流/字幕/状态/控制事件 | 互动对话与打断 |
| 异步状态回调 | HTTPS Webhook | JSON，带签名及重放保护 | 完成/失败通知 |
| 服务契约 | HTTPS静态文档或接口 | OpenAPI JSON/YAML、事件Schema | 第三方对接 |

- TLS版本至少1.2，推荐1.3；按部署需求使用mTLS。
- 语言使用BCP47编码（例如zh-CN、en-US），统一UTF-8文本、毫秒时长、RFC3339事件时间。
- 输入音频基线可采用16kHz/16bit/单声道PCM，实际支持的采样率/编码/帧长通过capabilities接口发现并协商。
- 流媒体传输支持序列号、会话ID、轮次ID、结束标记、心跳和取消事件。不得将旧轮次音频在新轮次继续播放。

## 3. 接口清单与字段

| 方法与路径（草案） | 功能 | 关键请求参数 | 关键响应参数 |
|---|---|---|---|
| POST /v1/asr/transcriptions | 语音文件识别 | audio/audio_uri、language、timestamps | text、language、segments |
| WSS /v1/asr/stream | 实时转写 | session_id、sequence、audio_frame | asr.partial、asr.final |
| POST /v1/tts/synthesize | 语音合成 | text、language、voice_id、format | audio_uri/audio_stream、duration_ms |
| POST /v1/voices/clone | 建立授权音色 | reference_audio、consent_id | voice_id、status |
| GET /v1/voices | 授权音色列表 | paging、filter | voices |
| WSS /v1/dialogue/stream | 双向智能讲解 | session_id、音频与上下文 | transcript、review_state、audio |
| POST /v1/explanations | 讲解候选/核查 | query、language、knowledge_scope | answer、citations、review_status |
| POST /v1/content/review | 安全/事实核查 | content、evidence_scope | safety_result、fact_results、decision |
| GET /v1/jobs/{job_id} | 异步任务 | job_id | status、result/error |
| GET /v1/capabilities | 支持能力查询 | — | languages、formats、features |
| GET /v1/health | 服务状态 | — | service_status、version |

字段规范：request_id(string，必填)作为请求唯一标识；session_id(string，会话接口必填)；language(string，可指定auto)；audio_format(enum，音频接口)；sample_rate(integer，原始音频接口)；text(string，TTS/讲解接口按需必填)；voice_id(string，指定音色必填)；stream(bool，可选)；trace_id(string，关联日志)。通用响应包含status、result、error{code,message}、model_revision、processing_ms、request_id、trace_id。tenant_id由已验证的调用身份派生，不允许任意指定越权。

正式接口规范还须对每个字段提供类型、必填、长度/大小上限、枚举取值、默认、示例、错误码、兼容版本及安全要求。

## 4. JSON请求响应示例

**语音合成请求**（演示结构，非真实在线调用）：

```http
POST /v1/tts/synthesize HTTP/1.1
Authorization: Bearer <access_token>
Content-Type: application/json
```
```json
{
  "request_id": "demo-req-0001",
  "text": "欢迎使用智能语音讲解。",
  "language": "zh-CN",
  "voice_id": "authorized-voice-001",
  "audio_format": "wav",
  "sample_rate": 24000,
  "stream": false
}
```

**示例返回：**

```json
{
  "request_id": "demo-req-0001",
  "status": "success",
  "result": {
    "audio_uri": "https://example.invalid/audio/001.wav",
    "duration_ms": 3200,
    "audio_format": "wav"
  },
  "model_revision": "example-revision",
  "processing_ms": 850,
  "trace_id": "demo-trace-001",
  "error": null
}
```

音频地址、处理时长、返回数据和模型版本**均为示例值**，不得截取并称为已完成的在线接口联调或实测成绩。

## 5. 鉴权和权限设计

- 支持OAuth 2.0的授权流程及经校验的Bearer访问令牌；服务端校验issuer、audience、签名与有效期。
- 服务间调用可采用Client Credentials模式，按需启用双向TLS。
- 采用按能力scope授权（speech:recognize、speech:synthesize、voice:manage、explanation:generate、content:review）；每个API均做租户隔离、能力权限及资源范围验证。
- 声音克隆必须核对consent_id、授权对象、允许用途、有效期和撤销状态；音频文件采用私有存储/短期签名访问。
- 记录访问审计、trace_id、请求错误及限流情况，避免打印明文密钥或未经脱敏的原始录音。

## 6. 请求频率限制及异常

支持按应用、租户、用户分层限制请求速率、文件大小、音频长度和实时会话配额。内部建议起始配置：REST 60次/分钟/应用、实时连接并发10路/应用、上传20MB、单次普通录音≤15分钟；**这只是接口规划配额，不是某款GPU已验证的并发处理能力**。验收时根据服务器实测结果与招标方协商最终数值。

超过配额返回HTTP 429，必要时携带Retry-After；实现可配置重试与退避策略。标准错误类别：400参数错误、401认证失败、403越权、404资源不存在、409状态冲突、413过大、415格式不支持、422语义约束、429限流、500内部错误、503服务不可用。所有错误返回request_id、trace_id及稳定业务error.code，不暴露内部密钥与堆栈。

## 7. 版本与数据互通

接口按/v1等方式版本化；兼容版本内新增可选字段不得破坏既有客户端；重大不兼容变更另开版本并按照合同约定提供迁移支持。对合法授权的识别文本、讲解内容、审计记录和音频元数据支持约定的数据导出及第三方系统读取；敏感录音与音色资产按授权和最小权限传输。

## 8. 开放接口与集成联调承诺（拟纳入投标文件）

1. **协议开放承诺**：本项目交付的语音服务使用公开、可文档化的HTTPS REST/WSS及双方约定的实时音频通信协议，不以未公开私有协议限制合法第三方集成。
2. **数据接口开放承诺**：提供语音识别、实时转写、语音合成、授权音色、智能讲解和三重核查的开放接口及参数文档、字段字典、请求响应示例、错误码和版本说明。
3. **鉴权与访问规则承诺**：明确访问令牌机制、访问权限、请求频率、并发连接、文件大小限制、重试及限流响应。
4. **联调支持承诺**：提供约定测试环境、测试凭据、接口实例、演示和问题定位服务，配合业主及指定单位完成通信、数据、权限、异常处理与性能联调，保留联调报告。
5. **不设不合理技术壁垒承诺**：不强制使用特定模型厂商专有SDK或隐藏协议作为访问条件，不通过不必要的技术限制阻碍本项目约定的正常互通。
6. **兼容维护承诺**：提供接口版本、兼容约束及迁移说明，保障约定范围内的后续集成与升级。

**承诺范围**：本项目实际交付且拥有合法开放权限的语音接口及授权数据；不包含第三方模型权重、未授权训练数据、供应商私有源代码、密钥或无权开放的知识产权。

## 9. 第6项联调验证用例

| 用例 | 输入操作 | 预期 |
|---|---|---|
| INT-01 | 标准REST识别 | 返回合法JSON及转写 |
| INT-02 | WSS流式音频按序传输 | partial、final、sequence可追踪 |
| INT-03 | 无Token/过期Token | 401拒绝 |
| INT-04 | 不足scope或跨租户访问 | 403拒绝 |
| INT-05 | 高频请求超额 | 429、Retry-After |
| INT-06 | 错误音频格式/超限 | 415/413等规范错误 |
| INT-07 | 未授权或已撤销音色 | 403/业务错误码 |
| INT-08 | 会话打断与旧音频 | 取消旧turn且不串音 |
| INT-09 | 第三方非专有SDK客户端 | 能完成标准接口调用 |
| INT-10 | 不兼容接口版本请求 | 按版本策略返回规范错误 |

完成上述验证后可提交API文档、实际请求响应截图、联调记录和问题关闭情况；设计示例不算实际联调成功。
