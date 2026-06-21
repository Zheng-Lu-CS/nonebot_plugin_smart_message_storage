# OpenAI API 与图片识别

插件通过 OpenAI 兼容的 `chat/completions` 接口做图片识别。源码会把图片压缩为 JPEG，再用 base64 形式放进 `image_url` 内容里发给模型。

OpenAI 视觉文档入口：

```text
https://platform.openai.com/docs/guides/images-vision
```

## 1. 三个关键配置

`.env.prod` 里和 AI 有关的配置是：

```env
AI_BASE_URL=https://api.openai.com/v1
AI_API_KEY=你的 OpenAI API Key
AI_MODEL=一个支持视觉输入、Chat Completions、JSON 输出的 OpenAI 模型
```

含义：

- `AI_BASE_URL`：API 服务地址。使用 OpenAI 官方接口时填 `https://api.openai.com/v1`。
- `AI_API_KEY`：你的 OpenAI API Key。不要公开、不要提交到 Git。
- `AI_MODEL`：模型名。必须支持图片输入，并能在 Chat Completions 里返回 JSON。

如果 `AI_API_KEY` 为空，插件会跳过识图逻辑，只保存原始消息内容。

## 2. 如何准备 OpenAI API Key

基本流程：

1. 打开 OpenAI Platform。
2. 登录账号。
3. 创建 API Key。
4. 确认账户有可用额度或账单配置。
5. 把 API Key 写入 `.env.prod` 的 `AI_API_KEY`。

不要把 API Key 发到 QQ 群、公开仓库、截图或日志里。

## 3. 如何选择模型

不要随便填一个模型名。这个插件要求模型同时满足：

- 支持图片输入。
- 支持 Chat Completions 接口。
- 支持或至少能稳定遵循 JSON 输出。

如果 OpenAI 文档推荐的模型发生变化，以官方模型和视觉文档为准。填好模型后，用本章的最小验证流程测试。

示例配置形态：

```env
AI_BASE_URL=https://api.openai.com/v1
AI_API_KEY=sk-xxxxxxxxxxxxxxxx
AI_MODEL=替换为支持视觉输入的模型名
```

如果模型不支持图片，日志里通常会出现请求失败、模型不支持该内容类型、或返回非预期内容等错误。

## 4. 图片识别流程

插件收到图片消息后会这样处理：

1. 把原始消息写入 `group_messages`。
2. 下载图片。
3. 压缩成 JPEG，保存到 `image_cache/`。
4. 在 `pending_images.json` 里登记待识别任务。
5. 满足以下任一条件时提交 AI：
   - 待识别图片数量达到 `IMAGE_BATCH_SIZE`，并且已积累足够后文。
   - 最早待识别图片超过 `IMAGE_FLUSH_SECONDS`。
   - 用户发送 `/立即识别`。
   - 用户回复图片发送 `/识别`。
6. AI 返回 JSON。
7. 插件把数据库里的图片 CQ 码回写成 `[image:{summary:"...",tip:"..."}]`。
8. 清理 pending 任务和图片缓存。

## 5. 最小验证流程

先把 `.env.prod` 配好：

```env
AI_BASE_URL=https://api.openai.com/v1
AI_API_KEY=你的 API Key
AI_MODEL=你的视觉模型
IMAGE_BATCH_SIZE=5
IMAGE_FLUSH_SECONDS=1800
IMAGE_CONTEXT_BEFORE_CHARS=100
IMAGE_CONTEXT_AFTER_CHARS=100
```

重启机器人：

```powershell
Set-Location D:\WorldModel\bot\smart_message_bot
.\.venv\Scripts\Activate.ps1
python bot.py
```

在群聊或私聊里发送：

```text
这张图是测试图片，帮我之后能搜到
```

然后发送一张图片，再发送一句后文：

```text
上面那张图是识别测试
```

手动触发识别：

```text
/立即识别
```

机器人应回复类似：

```text
已提交当前会话待识别图片 1 张。
```

稍等后检查数据库：

```powershell
python -c "import sqlite3; con=sqlite3.connect('qq_messages.db'); rows=con.execute('select id,raw_message from group_messages order by id desc limit 10').fetchall(); [print(r) for r in rows]"
```

如果看到：

```text
[image:{summary:"...",tip:"..."}]
```

说明图片识别和回写成功。

## 6. 回复图片使用 /识别

在 QQ 里回复一条图片消息，发送：

```text
/识别
```

行为：

- 如果这张图已经识别过，机器人直接返回数据库里的总结结果。
- 如果还没有识别，机器人会立即为这条图片补建 pending 任务并提交识别。
- 如果回复的消息不含图片，机器人会提示回复消息不包含图片。
- 如果数据库里找不到被回复消息，说明那条消息可能发生在机器人上线前，或没有被插件记录。

## 7. 隐私和费用

配置 AI 后，图片和相关聊天上下文会发送给 API 服务商。请确认：

- 目标群和好友知道或同意你做归档和识图。
- 不要把敏感证件、账单、聊天隐私随意提交给模型。
- OpenAI API 调用会产生费用，具体价格以 OpenAI 官方平台显示为准。

## 8. 常见 AI 错误

401 或 403：

- API Key 错误、过期、没权限或账户不可用。
- 检查 `.env.prod` 里的 `AI_API_KEY`。

模型不支持图片：

- `AI_MODEL` 不是视觉模型。
- 换成官方文档中支持图片输入的模型。

模型返回非 JSON：

- 插件要求 JSON，源码会解析 `choices[0].message.content`。
- 如果模型输出 Markdown、解释文字或图片链接，识别会失败。

网络超时：

- 源码请求超时时间是 120 秒。
- 检查网络、代理、API 服务可用性和图片大小。

未配置 API Key：

- 日志会提示跳过识图。
- 这是正常状态，消息仍会入库。

