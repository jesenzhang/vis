# 语音专项接口技术指标与互通契约（API-DESIGN V1）

> 以下为拟实现的接口契约，尚无真实服务可供调用。开放范围限VIS语音系统的接口和合法授权数据，不能许诺第三方未授权模型权重、商业源码或密钥。

## 1. 通信协议与数据格式

| 业务 | 协议 | 数据格式 | 约束 |
|---|---|---|---|
| 常规服务 | HTTPS REST（TLS≥1.2，推荐1.3） | JSON UTF-8、multipart/form-data | OpenAPI 3.1 |
| 流式ASR | WSS | JSON控制帧 + 二进制音频帧 | 帧20/40ms可协商 |
| 双向语音 | WSS，可选WebRTC | JSON事件、PCM/Opus | 交互/打断/背压 |
| 语音文件上传 | HTTPS | WAV/PCM，扩展MP3、Opus | 限时长/文件大小 |
| 异步回调 | HTTPS Webhook | JSON、验证签名/重放保护 | 通知任务完成 |

语种采用BCP 47标识，时间戳毫秒，日期RFC3339并保留时区；音频输入初步支持16kHz s16le单声道，合成24kHz或48kHz按模型能力协商。必须通过GET /v1/capabilities暴露实时支持语种、音频格式及版本，不承诺所有后端支持任意编码。

## 2. 主要API与字段

| 方法和URI | 功能 | 关键请求字段 | 关键响应字段 |
|---|---|---|---|
| POST /v1/asr/transcriptions | 音频文件识别 | file/audio_uri、language、timestamps | text、language、segments |
| WSS /v1/asr/stream | 实时ASR | session_id、sequence、audio/encoding | asr.partial/asr.final |
| POST /v1/tts/synthesize | 合成语音 | text、language、voice_id、audio_format | audio_uri或音频流、duration_ms |
| POST /v1/voices/clone | 授权声音克隆 | reference_audio、consent_id | voice_id、status |
| GET /v1/voices | 查询授权音色 | paging | voices |
| WSS /v1/dialogue/stream | 实时讲解交互 | session_id、language、音频帧 | text、audio、review事件 |
| POST /v1/explanations | 智能讲解 | question、language、knowledge_scope | answer、citations、review_status |
| POST /v1/content/review | 三重核查 | content、evidence_scope | decision、risk、facts、evidence |
| GET /v1/jobs/{job_id} | 异步任务查询 | job_id | status、result/error |
| GET /v1/capabilities | 查询能力 | — | languages、formats、features |
| GET /v1/health | 健康查询 | 身份按需 | status、version |

所有API提供字段名称、类型、必填、默认值、枚举、示例、业务错误码。公共请求含request_id、language、session_id（会话时）、trace_id；公共响应含request_id、status、result、error{code,message}、model_revision和trace_id。tenant_id必须来自可信认证上下文，不允许未经授权的调用方自行伪造。

## 3. 示例请求及响应（设计示意，非在线实测）

```http
POST /v1/tts/synthesize
Authorization: Bearer <access_token>
Content-Type: application/json
```
```json
{
  "request_id": "req-0001",
  "text": "欢迎使用智能讲解服务。",
  "language": "zh-CN",
  "voice_id": "authorized-voice-001",
  "audio_format": "wav",
  "sample_rate": 24000,
  "stream": false
}
```
```json
{
  "request_id": "req-0001",
  "status": "success",
  "result": {
    "audio_uri": "https://service.example/audio/001.wav",
    "duration_ms": 3200,
    "audio_format": "wav"
  },
  "model_revision": "example-revision",
  "trace_id": "trace-0001",
  "error": null
}
```

以上字段、地址和耗时均为接口示例，未证实已经上线。

## 4. WSS流事件

定义会话事件对象：type、session_id、turn_id、seq、sent_at、payload；音频帧为有序二进制数据，使用sequence进行丢帧与重放判断。事件包括session.start、audio.start、audio.chunk、audio.end、asr.partial、asr.final、review.status、tts.audio、turn.complete、turn.interrupt及error。打断turn后不得继续发送旧turn的未播放音频和字幕。

## 5. 鉴权、限流与错误处理

采用OAuth 2.0 Client Credentials用于服务对服务访问；校验JWT或不透明token的签名、issuer、audience、expires和scope，可选mTLS。scope示例speech:recognize、speech:synthesize、voice:manage、explanation:generate、content:review。

对声音克隆，consent_id、授权主体、租户、允许用途、有效期和撤销状态均需核验；不以公共URI直接公开用户录音，使用短期签名URL和受控对象访问。

**规划配额而非实测容量**：普通REST每应用60次/分钟；实时连接每应用最多10路；上传20MB；普通录音15分钟，长任务异步处理。正式值按合同/压测修正。超限返回429+Retry-After；400无效参数、401未认证、403禁止、404资源不存在、409冲突、413过大、415格式不支持、422业务约束、5xx系统故障。

需要定义request_id幂等行为、重试/指数退避、连接重建、心跳、会话失效及Webhook签名验真。

## 6. 对外开放及联调承诺

提供OpenAPI 3.1 JSON/YAML、WSS事件schema、字段数据字典、音频协议、SDK可选示例、curl/HTTP客户端样例及按合同约定的测试环境。配合第三方完成鉴权、负向越权、限流、格式、音频序列、断连/重连、语言切换、授权撤销和版本兼容性联调，并形成记录。

开放标准通信协议和合法授权的数据导出，不强制特定模型SDK，不采用不公开的私有协议阻碍项目约定接入；接口遵循/v1版本化和兼容升级规则。敏感原始音频、客户数据、第三方未授权模型权重与训练语料不得以“开放接口”名义泄露。
