# EmojiReactionLike

AstrBot 插件, 允许 bot 在接入 NapCat 平台时对 QQ 消息添加表情反应 (Reaction)。

支持手动指令触发、正则自动触发, 以及 LLM 通过文本标记自主添加表情反应。

## 功能

### 指令

| 指令 | 说明 |
|------|------|
| /react [emoji_id] | 回复一条消息并发送, 对被回复的消息添加表情反应。不指定 emoji_id 时使用第一条反应规则中的表情 |
| /reactlist | 列出互动反应表态 ID (443-448) 供参考 |
| /reactconfig | 查看当前插件配置 (仅管理员) |

### 自动反应

根据配置的正则规则, 自动对匹配的消息添加表情反应。支持多规则、多表情、多正则, 可配置仅群聊或仅私聊生效。

### LLM 表情反应

开启 `llm_react_enabled` 后, LLM 可以在回复文本中插入特殊标记来添加表情反应。插件会在 LLM 回复后一次性解析所有标记并调用 API, 然后从最终回复中清除这些标记。

支持两种标记格式:

- `[react:表情ID]` - 对当前用户消息添加反应
- `[react:表情ID,id:消息ID]` - 对指定 msg_id 的历史消息添加反应

### 消息 ID 注入

开启 `enable_msg_id_prefix` 后, 插件会在 LLM 请求前自动为聊天记录中的每条历史消息前插入 `msg_id:xxx`, 使 LLM 能够识别并指定历史消息进行反应。

注入效果示例:
```
msg_id:495891555 [pppopipupu/15:55:09]:  打火球术
---
msg_id:495891556 [pppopipupu/15:55:38]:  ？
---
msg_id:495891557 [芸诺_ miss/16:00:07]:  不是怎么都这么快。
```

## 配置项

所有配置可在 AstrBot WebUI 的插件配置页面中修改。

| 配置项 | 类型 | 默认值 | 说明 |
|--------|------|--------|------|
| auto_react | bool | false | 是否启用自动表情反应 |
| react_rules | template_list | [] | 反应规则列表, 每条规则包含 emoji_ids 和 regex_list |
| react_scope | string | all | 自动反应范围: all / group / private |
| llm_react_enabled | bool | false | 是否启用 LLM 输出表情反应 |
| enable_msg_id_prefix | bool | true | 是否注入消息 ID 参考表 |
| llm_react_regex | string | `\[react:([^\]]+)\]` | 匹配简单反应标记的正则 (反应当前消息) |
| llm_react_targeted_regex | string | `\[react:([^\],]+),id:([^\]]+)\]` | 匹配指定目标反应标记的正则 (反应指定消息) |

### react_rules 规则说明

每条规则包含两个字段:

- **emoji_ids**: 表情 ID 列表。QQ 原生表情填数字 (0-500), Unicode emoji 直接填字符, 插件自动用 `ord()` 转换。
- **regex_list**: 触发正则表达式列表。消息匹配列表中任意正则时触发反应。留空表示对所有消息反应。

### 人设提示词参考

在人格提示词中告知 LLM 可以使用反应标记, 例如:

```
当你想对消息进行表情反应时，请直接在你的正常回复文本中插入 [react:表情ID] 标记来对当前消息反应。如果要对历史消息反应，使用聊天记录中每条消息前的 msg_id，格式为 [react:表情ID,id:消息ID]。你可以在一条回复中插入多个标记。
警告：你必须将这些标记与你的回复文字写在一起，绝对不要使用任何发消息的工具（如 send_message）来单独发送这些标记！标记会被系统自动拦截并执行，用户只会看到你的正常回复文字。
可用的QQ数字表情ID: 448代表"火球术" 447代表"点赞" 446代表"摧心术" 445代表"魅惑怪物" 444代表"666" 443代表"死亡一指" 442代表"鸽子跳舞", 也可以使用Unicode emoji字符如😭。
```

## 平台支持

仅支持 aiocqhttp 平台 (NapCat / Lagrange 等 OneBot v11 实现)。
