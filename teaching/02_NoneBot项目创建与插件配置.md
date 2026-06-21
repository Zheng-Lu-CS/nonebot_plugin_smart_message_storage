# NoneBot 项目创建与插件配置

这一章在 `D:\WorldModel\bot\smart_message_bot` 里创建一个最小可运行的 NoneBot2 项目。它不依赖 `nb` 命令。

## 1. 进入项目并激活环境

```powershell
Set-Location D:\WorldModel\bot\smart_message_bot
.\.venv\Scripts\Activate.ps1
```

如果还没有安装插件，先执行：

```powershell
python -m pip install -e D:\WorldModel\project\nonebot_plugin_smart_message_storage
```

## 2. 创建 bot.py

在 `D:\WorldModel\bot\smart_message_bot\bot.py` 写入：

```python
import nonebot
from nonebot.adapters.onebot.v11 import Adapter as OneBotV11Adapter

nonebot.init()

driver = nonebot.get_driver()
driver.register_adapter(OneBotV11Adapter)

nonebot.load_plugin("nonebot_plugin_smart_message_storage")

if __name__ == "__main__":
    nonebot.run()
```

这个文件做了三件事：

- 初始化 NoneBot。
- 注册 OneBot v11 适配器。
- 加载本插件。

## 3. 创建 .env.prod

在同一目录创建 `.env.prod`：

```env
DRIVER=~fastapi
HOST=127.0.0.1
PORT=8080

# 改成你的 QQ 号。多个超级用户可写成 ["123","456"]。
SUPERUSERS=["你的QQ号"]

# 数据库放在机器人项目目录下。
DB_URL=sqlite:///qq_messages.db

# OpenAI 官方接口。先留空也能启动，只是不识别图片。
AI_BASE_URL=https://api.openai.com/v1
AI_API_KEY=

# 填一个支持视觉输入、Chat Completions、JSON 输出的 OpenAI 模型。
# 不确定模型时，先看 OpenAI 模型和视觉文档，再用 04 章的验证步骤测试。
AI_MODEL=

IMAGE_BATCH_SIZE=5
IMAGE_FLUSH_SECONDS=1800
IMAGE_CONTEXT_BEFORE_CHARS=100
IMAGE_CONTEXT_AFTER_CHARS=100
```

说明：

- `SUPERUSERS` 决定谁能执行 `/立即识别 全部`。
- `DB_URL=sqlite:///qq_messages.db` 会在机器人运行目录生成 `qq_messages.db`。
- `AI_API_KEY` 留空时，插件只存消息，不做图片识别。
- `AI_MODEL` 必须是支持图片输入和 JSON 输出的模型，否则识别会失败。

## 4. 启动机器人

```powershell
python bot.py
```

正常启动时应看到类似信息：

```text
Smart message storage data dir: ...
Smart message storage AI image recognition is disabled: ai_api_key is not configured.
```

如果你已经配置了 `AI_API_KEY`，第二行禁用提示不会出现。

启动后不要关闭这个 PowerShell 窗口。下一步需要让 NapCat 连接到这个 NoneBot 服务。

## 5. 数据与缓存在哪里

插件有两类存储位置。

SQLite 数据库：

```text
D:\WorldModel\bot\smart_message_bot\qq_messages.db
```

这个库里有 `group_messages` 表，是主要消息归档。

localstore 数据目录：

```text
nonebot_plugin_smart_message_storage 数据目录
```

实际路径会在启动日志里打印，里面主要有：

```text
pending_images.json
image_cache/
```

含义：

- `pending_images.json`：待识别图片任务账本。
- `image_cache/`：等待提交给 AI 的图片缓存。

图片识别成功或失败后，对应 pending 任务和缓存图片会被清理。

## 6. 启动成功检查

先只检查 NoneBot 本身：

```powershell
python -m pip show nonebot2 nonebot-adapter-onebot nonebot-plugin-localstore nonebot-plugin-smart-message-storage
Test-Path .\bot.py
Test-Path .\.env.prod
```

再检查数据库是否会自动创建：

```powershell
Test-Path .\qq_messages.db
```

如果机器人刚启动但还没有收到任何消息，数据库文件也可能已经创建，但表内为空，这是正常的。

