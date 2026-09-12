# orangechat2mikeko

把橘瓣 OrangeChat 的本地备份 zip 转换成米可 Mikeko 可直接导入的备份 zip。

## 下载

`orangechat2mikeko.html` —— 单文件网页工具，双击即用，无需联网。

## 用法

1. 打开 orangechat2mikeko.html
2. 选 OrangeChat 导出的备份 zip（设置 → 数据备份 → 本地备份 → 导出备份）
3. 勾选要转换的会话
4. 建议勾上「合并我已有的 mikan 备份」——否则 mikan 导入时会整目录替换 databases/，你现有的记忆、置顶、项目、工作流会被一起删掉
5. 下载生成的 mikeko-import-*.zip，在 mikan 里「设置 → 备份与恢复 → 导入备份」

## 说明

转换器把 OrangeChat 的 `rikka_hub.db`（Room 数据库）重新构造成 mikan 的 `mikeko.db`（SQLiteOpenHelper v17），并生成 mikan 能直接导入的 zip。全部计算在浏览器本地完成，不上传任何文件。
