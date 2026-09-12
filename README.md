# orangechat2mikeko

把橘瓣 OrangeChat 的本地备份 zip 转换成米可 Mikeko 可以直接导入的备份 zip。

## 下载

`orangechat2mikeko.html` —— 单文件网页工具，双击即用，不需要联网。

## 用法

1. 打开 orangechat2mikeko.html
2. 选 OrangeChat 导出的备份 zip（设置 → 数据备份 → 本地备份 → 导出备份）
3. 勾选要转换的会话
4. 建议勾上「合并我已有的 mikan 备份」——否则 mikan 导入时会整目录替换 databases/，你现有的记忆、置顶、项目、工作流会被一起删掉
5. 下载生成的 mikeko-import-*.zip，在 mikan 里「设置 → 备份与恢复 → 导入备份」

## 已处理的两个关键坑

### 1. 导出时数据还在 WAL 日志里

橘瓣用 SQLite 的 WAL 模式，写入先落在 `rikka_hub-wal`，主库可能只有表结构。
导出备份时如果应用没做 checkpoint，只看主库就会读到 0 个会话 ——
典型现象是「导出包几百 MB、几百个条目，却显示 0 个会话」。

转换器会自动解析 WAL 并把已提交的页合并到主库上，聊天记录完整读出。

### 2. 附件目录名是 upload/ 不是 uploads/

橘瓣的 `FileFolders.UPLOAD = "upload"`，导出到 zip 里是 `upload/<文件名>`。
转换器兼容 `upload/`、`uploads/`、`files/upload/`、`files/uploads/` 四种前缀。

### 3. 大包不会撑爆浏览器

导出包可能有几百 MB（实测 263 MB / 433 个条目），其中绝大多数是附件。
转换器只解析 zip 中央目录，附件按需读取、用完即释放：
实测 252 MB 的包，峰值内存只有几 MB。

## 转换映射

| OrangeChat | Mikeko |
|---|---|
| `conversationentity.title` | `conversations.title` |
| `custom_system_prompt` + 会话内 system 消息 | `conversations.system_prompt` |
| `message_node.messages[selectIndex]` | `messages` 一行 |
| reasoning part | 正文里的思考块（mikan 渲染成思考面板） |
| tool part | `segments` 里的工具卡片（保留调用位置与结果） |
| image / document / video / audio / voice_message | `image_path` + `attachment_mime`（附件一并打包） |
| usage 统计 | `token_count` / `cached_tokens` |
| createdAt / finishedAt | `created_at` / `duration_ms` |

写入的 `mikeko.db` 带正确的 `user_version = 17`、`messages_conversation` 索引、
`sqlite_sequence`，并通过 `PRAGMA integrity_check` 与 `foreign_key_check`。

## 测试

578 项断言，包括与 Node zlib、Python zipfile、sqlite3 CLI 的双向互操作验证。

```sh
sh tests/run_all.sh
```
