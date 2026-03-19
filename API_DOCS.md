# Grok2API 流式输出接口文档

> Base URL: `https://grok.infjpersonality.top`
> 认证方式: Bearer Token (`Authorization: Bearer <API_KEY>`)

---

## 目录

- [模型列表](#模型列表)
- [Chat Completions (流式)](#chat-completions-流式)
- [Chat Completions (非流式)](#chat-completions-非流式)
- [图片生成](#图片生成)
- [视频生成](#视频生成)
- [错误处理](#错误处理)

---

## 模型列表

### `GET /v1/models`

返回所有可用模型。

```bash
curl https://grok.infjpersonality.top/v1/models \
  -H "Authorization: Bearer YOUR_API_KEY"
```

| 模型 ID | 类型 | 说明 |
|---------|------|------|
| `grok-3` | 对话 | Grok 3 标准模型 |
| `grok-3-mini` | 对话 | Grok 3 轻量版 |
| `grok-3-thinking` | 推理 | Grok 3 深度思考 |
| `grok-4` | 对话 | Grok 4 标准模型 |
| `grok-4-thinking` | 推理 | Grok 4 深度思考 |
| `grok-4-heavy` | 推理 | Grok 4 重型推理 |
| `grok-4.1-mini` | 对话 | Grok 4.1 轻量版 |
| `grok-4.1-fast` | 对话 | Grok 4.1 快速版 |
| `grok-4.1-expert` | 推理 | Grok 4.1 专家版 |
| `grok-4.1-thinking` | 推理 | Grok 4.1 深度思考 |
| `grok-4.20-beta` | 推理 | Grok 4.20 Beta |
| `grok-imagine-1.0-fast` | 图片 | 快速图片生成 |
| `grok-imagine-1.0` | 图片 | 标准图片生成 |
| `grok-imagine-1.0-edit` | 图片编辑 | 图片编辑 |
| `grok-imagine-1.0-video` | 视频 | 视频生成 |

---

## Chat Completions (流式)

### `POST /v1/chat/completions`

兼容 OpenAI Chat Completions API，支持 SSE 流式输出。

### 请求参数

```json
{
  "model": "grok-3",
  "messages": [
    {"role": "system", "content": "你是一个有用的助手"},
    {"role": "user", "content": "你好"}
  ],
  "stream": true,
  "temperature": 0.8,
  "top_p": 0.95,
  "reasoning_effort": null,
  "tools": null,
  "tool_choice": null
}
```

| 参数 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| `model` | string | ✅ | - | 模型 ID |
| `messages` | array | ✅ | - | 消息数组 |
| `stream` | boolean | ❌ | `false` | 是否流式输出 |
| `temperature` | float | ❌ | `0.8` | 采样温度 (0-2) |
| `top_p` | float | ❌ | `0.95` | Nucleus 采样 (0-1) |
| `reasoning_effort` | string | ❌ | `null` | 推理强度 (thinking 模型专用) |
| `tools` | array | ❌ | `null` | Tool Calling 工具定义 |
| `tool_choice` | string/object | ❌ | `null` | 工具选择策略 |
| `parallel_tool_calls` | boolean | ❌ | `true` | 是否允许并行工具调用 |

### messages 消息格式

```json
[
  {"role": "system", "content": "系统提示"},
  {"role": "user", "content": "用户消息"},
  {"role": "assistant", "content": "助手回复"},
  {"role": "user", "content": [
    {"type": "text", "text": "描述这张图片"},
    {"type": "image_url", "image_url": {"url": "https://example.com/img.jpg"}}
  ]}
]
```

| role | 说明 | content 类型 |
|------|------|-------------|
| `system` / `developer` | 系统提示 | string |
| `user` | 用户消息 | string / array (支持 text, image_url, input_audio, file) |
| `assistant` | 助手回复 | string |
| `tool` | 工具返回 (需 tool_call_id) | string |

### reasoning_effort 推理强度

仅对 thinking 模型生效：

| 值 | 说明 |
|----|------|
| `none` | 不推理 |
| `minimal` | 最小推理 |
| `low` | 低 |
| `medium` | 中等 |
| `high` | 高 |
| `xhigh` | 极高 |

---

### 流式 curl 示例

```bash
curl -N https://grok.infjpersonality.top/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "model": "grok-3",
    "messages": [{"role": "user", "content": "你好"}],
    "stream": true
  }'
```

### 流式响应格式 (SSE)

每一行以 `data: ` 开头，JSON 格式：

**首个 chunk（角色声明）：**
```
data: {"id":"chatcmpl-xxx","object":"chat.completion.chunk","created":1773737387,"model":"grok-3","system_fingerprint":"","choices":[{"index":0,"delta":{"role":"assistant","content":""},"logprobs":null,"finish_reason":null}]}
```

**内容 chunk：**
```
data: {"id":"xxx","object":"chat.completion.chunk","created":1773737387,"model":"grok-3","system_fingerprint":"xxx","choices":[{"index":0,"delta":{"content":"你好"},"logprobs":null,"finish_reason":null}]}
```

**结束 chunk：**
```
data: {"id":"xxx","object":"chat.completion.chunk","created":1773737387,"model":"grok-3","choices":[{"index":0,"delta":{},"logprobs":null,"finish_reason":"stop"}]}
```

**终止信号：**
```
data: [DONE]
```

### 流式响应字段说明

| 字段 | 类型 | 说明 |
|------|------|------|
| `id` | string | 响应唯一 ID |
| `object` | string | 固定 `"chat.completion.chunk"` |
| `created` | integer | Unix 时间戳 |
| `model` | string | 使用的模型 |
| `system_fingerprint` | string | 系统指纹 |
| `choices[].index` | integer | 选项序号 |
| `choices[].delta.role` | string | 角色 (仅首个 chunk) |
| `choices[].delta.content` | string | 增量内容 |
| `choices[].finish_reason` | string/null | `null`=进行中, `"stop"`=完成 |

---

## Chat Completions (非流式)

```bash
curl https://grok.infjpersonality.top/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "model": "grok-3",
    "messages": [{"role": "user", "content": "你好"}],
    "stream": false
  }'
```

### 非流式响应

```json
{
  "id": "xxx",
  "object": "chat.completion",
  "created": 1773737255,
  "model": "grok-3",
  "system_fingerprint": "xxx",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "你好！有什么可以帮你的吗？",
        "refusal": null,
        "annotations": []
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 0,
    "completion_tokens": 0,
    "total_tokens": 0
  }
}
```

---

## 图片生成

使用图片模型 (`grok-imagine-1.0`, `grok-imagine-1.0-fast`)：

```bash
curl https://grok.infjpersonality.top/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "model": "grok-imagine-1.0",
    "messages": [{"role": "user", "content": "画一只可爱的猫咪"}],
    "stream": true,
    "image_config": {
      "n": 1,
      "size": "1024x1024",
      "response_format": "url"
    }
  }'
```

### image_config 参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `n` | int | `1` | 生成数量 (1-10, 流式最多 2) |
| `size` | string | `"1024x1024"` | 图片尺寸 |
| `response_format` | string | `"url"` | 返回格式: `url` / `b64_json` |

### 允许的图片尺寸

| size | 比例 |
|------|------|
| `1024x1024` | 1:1 |
| `1280x720` | 16:9 |
| `720x1280` | 9:16 |
| `1792x1024` | 3:2 |
| `1024x1792` | 2:3 |

---

## 视频生成

使用 `grok-imagine-1.0-video` 模型：

```bash
curl https://grok.infjpersonality.top/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "model": "grok-imagine-1.0-video",
    "messages": [{"role": "user", "content": "一只猫在跳舞"}],
    "stream": true,
    "video_config": {
      "aspect_ratio": "16:9",
      "video_length": 6,
      "resolution_name": "480p",
      "preset": "custom"
    }
  }'
```

### video_config 参数

| 参数 | 类型 | 默认值 | 说明 |
|------|------|--------|------|
| `aspect_ratio` | string | `"3:2"` | 视频比例 |
| `video_length` | int | `6` | 时长 6-30 秒 |
| `resolution_name` | string | `"480p"` | 分辨率: `480p` / `720p` |
| `preset` | string | `"custom"` | 风格: `fun` / `normal` / `spicy` / `custom` |

### 允许的视频比例

`16:9`, `9:16`, `3:2`, `2:3`, `1:1`（也支持像素写法如 `1280x720`）

---

## Tool Calling

```bash
curl https://grok.infjpersonality.top/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "model": "grok-3",
    "messages": [{"role": "user", "content": "北京今天天气怎么样？"}],
    "stream": true,
    "tools": [
      {
        "type": "function",
        "function": {
          "name": "get_weather",
          "description": "获取指定城市的天气",
          "parameters": {
            "type": "object",
            "properties": {
              "city": {"type": "string", "description": "城市名"}
            },
            "required": ["city"]
          }
        }
      }
    ],
    "tool_choice": "auto"
  }'
```

### tool_choice 选项

| 值 | 说明 |
|----|------|
| `"auto"` | 模型自动决定是否调用工具 |
| `"required"` | 强制调用工具 |
| `"none"` | 禁止调用工具 |
| `{"type": "function", "function": {"name": "xxx"}}` | 指定调用某个工具 |

---

## 图片理解 (Vision)

在 user 消息中嵌入图片：

```bash
curl https://grok.infjpersonality.top/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "model": "grok-3",
    "messages": [
      {
        "role": "user",
        "content": [
          {"type": "text", "text": "描述这张图片"},
          {"type": "image_url", "image_url": {"url": "https://example.com/photo.jpg"}}
        ]
      }
    ],
    "stream": true
  }'
```

支持的 content block 类型（仅 user 角色）：
- `text` — 文本
- `image_url` — 图片 URL 或 data URI (`data:image/jpeg;base64,...`)
- `input_audio` — 音频 data URI
- `file` — 文件 data URI

---

## 错误处理

### HTTP 状态码

| 状态码 | 说明 |
|--------|------|
| `200` | 成功 |
| `400` | 请求参数错误 |
| `401` | 认证失败 (API Key 无效) |
| `429` | 速率限制 (无可用 Token) |
| `500` | 服务器内部错误 |

### 错误响应格式

```json
{
  "error": {
    "message": "The model `xxx` does not exist or you do not have access to it.",
    "type": "invalid_request_error",
    "param": "model",
    "code": "model_not_found"
  }
}
```

### 流式错误 (SSE)

流式传输中发生错误时，会通过事件推送：

```
event: error
data: {"error":{"message":"No available tokens","type":"rate_limit_error","code":"rate_limit_exceeded"}}

data: [DONE]
```

---

## 代码示例

### Python (OpenAI SDK)

```python
from openai import OpenAI

client = OpenAI(
    api_key="YOUR_API_KEY",
    base_url="https://grok.infjpersonality.top/v1"
)

# 流式输出
stream = client.chat.completions.create(
    model="grok-3",
    messages=[{"role": "user", "content": "你好"}],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="", flush=True)
```

### JavaScript (fetch)

```javascript
const response = await fetch('https://grok.infjpersonality.top/v1/chat/completions', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Authorization': 'Bearer YOUR_API_KEY'
  },
  body: JSON.stringify({
    model: 'grok-3',
    messages: [{ role: 'user', content: '你好' }],
    stream: true
  })
});

const reader = response.body.getReader();
const decoder = new TextDecoder();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  
  const text = decoder.decode(value);
  const lines = text.split('\n').filter(l => l.startsWith('data: '));
  
  for (const line of lines) {
    const data = line.slice(6);
    if (data === '[DONE]') break;
    
    const chunk = JSON.parse(data);
    const content = chunk.choices?.[0]?.delta?.content;
    if (content) process.stdout.write(content);
  }
}
```

### curl (流式)

```bash
curl -N https://grok.infjpersonality.top/v1/chat/completions \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "model": "grok-4-thinking",
    "messages": [{"role": "user", "content": "解释量子纠缠"}],
    "stream": true,
    "temperature": 0.7,
    "reasoning_effort": "high"
  }'
```
