# QQ 小号与 NapCat / OneBot 接入

本教程默认使用 QQ 小号作为机器人账号。登录到 NapCat 的 QQ 号，就是插件能看到和记录的账号。

## 1. 为什么推荐小号

推荐小号的原因：

- 降低主号长期挂协议端的风险。
- 机器人行为和个人聊天分开，隐私边界更清楚。
- 出问题时可以停用小号，不影响主号日常使用。

小号的限制：

- 小号必须加入目标群，才能记录目标群的新消息。
- 私聊好友必须能和小号互发消息，插件才能记录这条小号私聊会话。
- 小号不能直接读取你个人号与某位好友之间已经存在的私聊历史。

如果你坚持使用个人号，插件能记录个人号上线后收到的新消息，但账号安全和风控风险更高，也仍然不能自动读取 QQ 客户端历史数据库。

## 2. 安装和启动 NapCat

请按 NapCat 官方文档安装和登录：

```text
https://napneko-napcatqq.mintlify.app/
```

建议操作：

- 用 QQ 小号登录 NapCat。
- 确认小号能正常收发消息。
- 把小号加入目标群。
- 让目标私聊好友和小号互发一句测试消息。

## 3. 配置 OneBot v11 反向 WebSocket

NoneBot 这边监听：

```text
127.0.0.1:8080
```

OneBot v11 反向 WebSocket 地址通常写成：

```text
ws://127.0.0.1:8080/onebot/v11/ws
```

在 NapCat 的 OneBot 配置里添加一个反向 WebSocket 连接，填入上面的地址。

如果你的 NapCat 文档或界面里路径名称不同，以实际文档为准。NoneBot OneBot 适配器连接配置见：

```text
https://onebot.adapters.nonebot.dev/docs/guide/setup
```

## 4. 启动顺序

推荐顺序：

1. 启动 NoneBot：

```powershell
Set-Location D:\WorldModel\bot\smart_message_bot
.\.venv\Scripts\Activate.ps1
python bot.py
```

2. 启动 NapCat，并确保小号在线。
3. 在 NapCat 里启用 OneBot v11 反向 WebSocket。
4. 观察 NoneBot 日志，确认 OneBot 客户端连接成功。

## 5. 连接验证

在目标群里让小号能看到一条消息，例如：

```text
测试智能消息存储
```

然后在机器人项目目录检查数据库：

```powershell
Set-Location D:\WorldModel\bot\smart_message_bot
.\.venv\Scripts\Activate.ps1
python -c "import sqlite3; con=sqlite3.connect('qq_messages.db'); print(con.execute('select id,time,group_id,user_id,raw_message from group_messages order by id desc limit 5').fetchall())"
```

如果能看到刚才的测试内容，说明入库成功。

## 6. 群聊和私聊的识别方式

群聊：

- 数据库里 `group_id` 是真实群号。
- `/查消息 关键词` 在当前群里查。
- `/查消息 群号 关键词` 可以从任意会话查指定群。

私聊：

- 数据库里 `group_id=-1`。
- `user_id` 是和机器人私聊的对方 QQ。
- 在这条私聊会话里发送 `/查消息 关键词`，只会查当前这个好友与小号的私聊。

## 7. 关于已有历史记录

小号不能读取个人号历史，因此要补录已有历史，推荐：

- 群聊历史：个人号把关键历史文本和图片逐条转发给小号所在群，或者发给小号私聊。
- 私聊历史：个人号把和好友的关键历史逐条复制/转发给小号，或者截图发给小号。
- 图片历史：尽量发送原图；如果只能截图，AI 会总结截图内容，但数据库里不会有原始消息结构。

不要把“QQ 合并转发聊天记录”当成自动导入方式。当前插件不会展开合并转发里的每条消息，也不会自动识别合并记录里的每张图片。

