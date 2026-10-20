---
default_mode:
destination:
destination_ref:
---

`default_mode` 只填 `report` 或 `assistant`。留空则每次先问。
`destination` 填记录要推送到的工具，例如 `notion`、`jira`、`linear`、`slack`、`github-issues`，或 `local`（只留本地记录）。留空则首次写记录前问一次，用户同意后写入。
`destination_ref` 填目标位置，例如 Notion database 链接、Jira 项目 key、Slack 频道名。由技能在首次确定或创建目标后写入，之后都往这里追加。
